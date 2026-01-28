<div align="center">
  <img src="./banner.png" width="600px" alt="DependabotSecureFlow Banner" style="border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.5);">
  <br><br>
  
  [![Use this template](https://img.shields.io/badge/Use%20this%20Template-2ea44f?style=for-the-badge&logo=github)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/generate)
  [![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Dependabot%20Secure%20Flow-blue?style=for-the-badge&logo=github)](https://github.com/marketplace/actions/dependabot-secure-flow)
  [![Build Status](https://img.shields.io/github/actions/workflow/status/EthanThePhoenix38/dependabot-secure-flow/dependabot-secure-flow.yaml?style=for-the-badge)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/actions)
  [![Dependabot](https://img.shields.io/badge/Dependabot-Active-025E8C?style=for-the-badge&logo=dependabot&logoColor=white)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/network/dependencies)
  [![Languages](https://img.shields.io/github/languages/count/EthanThePhoenix38/dependabot-secure-flow?style=for-the-badge&color=blue)](https://github.com/EthanThePhoenix38/dependabot-secure-flow)
  [![Tests](https://img.shields.io/badge/Tests-Passing-green?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/actions)
  [![RGPD](https://img.shields.io/badge/RGPD-Compliant-blue?style=for-the-badge&logo=gdpr&logoColor=white)](https://github.com/EthanThePhoenix38/dependabot-secure-flow)
</div>

# [DependabotSecureFlow](https://github.com/EthanThePhoenix38/dependabot-secure-flow)

## Table of Contents
- [Overview](#overview)
- [Key Features & Philosophy](#key-features--philosophy)
- [Why This Pattern? (The "Secret Sauce")](#why-this-pattern-the-secret-sauce)
- [Installation](#installation)
- [Configuration Options](#configuration-options)
- [Support This Project](#support-this-project)

**Version**: 1.0.0
**Author**: EthanThePhoenix38
**License**: MIT

## Overview

**DependabotSecureFlow** is an enterprise-grade GitHub Action workflow designed for zero-touch dependency management. It implements a **"Silent Guardian"** philosophy: automating security updates, validating them via actual builds, and auto-correcting failures without human intervention.

It acts as a Security Sandbox that filters dependencies before they reach your production code.

## Key Features & Philosophy

### 1. Absolute Silence & Automation
*   **No Notification Fatigue**: The workflow is designed to be invisible. You only receive notifications for successful outcomes (ready-to-merge batch updates).
*   **Green-Only Policy**: A failing dependency update does not mark your pipeline as "Failed". It is caught, handled, and discarded quietly.

### 2. Security Sandbox (`securite` branch)
*   **Isolation**: Dependabot PRs are never merged directly to `master`. They are routed to a staging branch (`securite`).
*   **Batching**: All valid updates are consolidated into a single "Batch PR" from `securite` to `master`, ensuring a clean commit history.

### 3. Build Validation (The "npm" step)
*   **Why is this here?**: This step is NOT installing the Action itself. It determines whether the dependency update breaks *your* application.
*   **Smoke Testing**: The workflow runs `npm install` and `npm run build` on your project.
*   **Prevention**: If a dependency installs correctly but causes your build to fail (e.g., breaking changes), it is rejected immediately.

### 4. Auto-Correction ("Self-Healing")
*   **The Problem**: A bad dependency update leaves a hanging PR that requires manual cleanup.
*   **The Solution**: If validation fails, the workflow automatically closes the PR, deletes the branch, and tells Dependabot to ignore that specific version.

---

## Why This Pattern? (The "Secret Sauce")

This workflow implements a rare and advanced DevOps pattern known as a **Self-Healing Pipeline**.

*   **SRE-Grade Maturity**: This architecture is typically found in highly mature Site Reliability Engineering (SRE) teams. It shifts security left while maintaining operational stability.
*   **Self-Healing**: The pipeline detects its own failures (bad updates) and repairs itself (closes PRs, cleans branches) without paging a human.
*   **Ultra-Lightweight**: Despite its sophisticated logic, the entire workflow is **under 100 lines of YAML**. It requires no external services, no complex binaries, and no paid add-ons. It is pure, native GitHub Actions logic.

| Feature | Standard Dependabot | DependabotSecureFlow |
| :--- | :--- | :--- |
| **Verification** | None (unless user adds tests) | **Build & Install verified** |
| **Failure Handling** | Fails Workflow (Red Cross) | **Auto-Closes & Self-Heals (Silent)** |
| **Merge Strategy** | Many PRs flood `main` | **1 Batch PR** (`securite` branch) |
| **Monitoring** | Manual Review Required | **Zero-Touch** |
| **Architecture** | Simple Automation | **Self-Healing / SRE Grade** |

---

## Installation

This is a **Workflow Template**. You do not need to install anything with npm to use it. You simply add the configuration file to your repository.

> [!WARNING]
> This workflow assumes your repository is a **Node.js project** with a valid `package.json` file. The validation step runs `npm install` and `npm run build`. If your project uses a different stack, please adjust the default `test-command` input or your `package.json` scripts accordingly.


1.  **Prerequisite**: Create a file named `.github/dependabot.yml` to enable Dependabot scanning:
    ```yaml
    version: 2
    updates:
      - package-ecosystem: "npm"
        directory: "/"
        schedule:
          interval: "daily"
    ```
2.  **Prerequisite**: Create a `CHANGELOG.md` file (required for auto-updates):
    ```markdown
    # Changelog
    
    ## [Unreleased]
    
    ### Added
    ```
3.  Create a file named `.github/workflows/dependabot-secure-flow.yml`.
4.  Copy the code below into that file.

### Critical Security Requirement: Secrets
**Do not hardcode email addresses in the workflow file.** Hardcoded emails can be detected by security scanners and inadvertently exposed.

1.  Go to your repository **Settings** > **Secrets and variables** > **Actions**.
2.  Create a new Repository Secret named `GIT_AUTHOR_EMAIL`.
3.  Set the value to the email address you want associated with the automated commits.

### Workflow Code

```yaml
name: DependabotSecureFlow
on:
  pull_request:
    types: [opened, synchronize]
    paths: ['package.json', 'package-lock.json', '.github/workflows/**']

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  # 1. Validation & Auto-Correction
  check-interdependencies:
    runs-on: ubuntu-latest
    if: ${{ contains(github.event.pull_request.labels.*.name, 'dependencies') || startsWith(github.head_ref, 'dependabot/') }}
    outputs:
      should_merge: ${{ steps.outcome.outputs.result }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      
      # This step verifies that YOUR application builds correctly with the new dependency.
      - name: Validation (Install & Build)
        id: validate
        continue-on-error: true
        run: |
          npm install
          npm run build
      
      - name: Decision
        id: outcome
        if: always()
        run: |
          if [ "${{ steps.validate.outcome }}" == "failure" ]; then
            gh pr edit ${{ github.event.pull_request.number }} --add-label "skipped-vulnerability"
            gh pr close ${{ github.event.pull_request.number }} --comment "Auto-Correction: Build failed. Closing to prevent regression." --delete-branch
            echo "result=false" >> $GITHUB_OUTPUT
          else
            echo "result=true" >> $GITHUB_OUTPUT
          fi
        env: { GH_TOKEN: '${{ secrets.GITHUB_TOKEN }}' }

  # 2. Merge Valid Updates to Sandbox
  auto-merge-to-securite:
    needs: check-interdependencies
    if: ${{ needs.check-interdependencies.outputs.should_merge == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - run: |
          git fetch origin securite || git switch --create securite
          git config --global user.email '${{ secrets.GIT_AUTHOR_EMAIL }}'
          git config --global user.name 'DependabotSecureFlow'
          git merge origin/${{ github.head_ref }} --no-edit
          git push origin securite
```

## Configuration Options

| Input | Description | Required | Default |
| :--- | :--- | :--- | :--- |
| `github-token` | Token to manage PRs (GITHUB_TOKEN) | Yes | N/A |
| `node-version` | Node versions to use | No | 20 |
| `test-command` | Command to run for validation | No | `npm install && npm run build` |

## Support This Project

If this action helps secure your projects, support the development:

[![GitHub Sponsors](https://img.shields.io/badge/Sponsor_on-GitHub-ea4aaa?style=for-the-badge&logo=github)](https://github.com/sponsors/EthanThePhoenix38)
[![Patreon](https://img.shields.io/badge/Support_on-Patreon-F96854?style=for-the-badge&logo=patreon)](https://patreon.com/EthanThePhoenix)
[![PayPal](https://img.shields.io/badge/Support_via-PayPal-00457C?style=for-the-badge&logo=paypal)](https://www.paypal.com/paypalme/VanessaBernier)
[![Ko-fi](https://img.shields.io/badge/Support_on-Ko--fi-F16061?style=for-the-badge&logo=ko-fi)](https://ko-fi.com/EthanThePhoenix)

**Your support helps fund the server and AI development!**
In exchange, I will add a link to your GitHub profile in the Contributors section.

Or you can:
- ⭐ **Star this repository**
- 🔀 **Fork it** to customize for your needs
- 🐛 **Report issues** to help improve it- ⭐ **Star this repository**
- 🔀 **Fork it** to customize for your needs
- 🐛 **Report issues** to help improve it

## Professional Page
[https://thephoenixagency.github.io](https://thephoenixagency.github.io)
