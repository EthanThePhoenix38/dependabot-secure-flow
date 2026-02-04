<div align="center">
  <img src="./banner.png" width="600px" alt="DependabotSecureFlow Banner" style="border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.5);">
  <br><br>
  
  [![Use this template](https://img.shields.io/badge/Use%20this%20Template-2ea44f?style=for-the-badge&logo=github)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/generate)
  [![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Dependabot%20Secure%20Flow-blue?style=for-the-badge&logo=github)](https://github.com/marketplace/actions/dependabotsecureflow)
  [![Build Status](https://img.shields.io/github/actions/workflow/status/EthanThePhoenix38/dependabot-secure-flow/dependabot-secure-flow.yaml?style=for-the-badge)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/actions)
  [![Dependabot](https://img.shields.io/badge/Dependabot-Active-025E8C?style=for-the-badge&logo=dependabot&logoColor=white)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/network/dependencies)
  [![YAML](https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white)](https://github.com/EthanThePhoenix38/dependabot-secure-flow)  [![Tests](https://img.shields.io/badge/Tests-Passing-green?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/actions)  [![MIT License](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](https://choosealicense.com/licenses/mit/)  [![GitHub release](https://img.shields.io/github/v/release/EthanThePhoenix38/dependabot-secure-flow?style=for-the-badge)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/releases)  [![Security](https://img.shields.io/badge/Security-Hardened-yellow?style=for-the-badge&logo=security)](https://github.com/EthanThePhoenix38/dependabot-secure-flow/security)
  [![RGPD](https://img.shields.io/badge/RGPD-Compliant-blue?style=for-the-badge&logo=gdpr&logoColor=white)](https://github.com/EthanThePhoenix38/dependabot-secure-flow)

</div>

# Dependabot Secure Flow

> Automated dependency management with security-first batch processing

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Dependabot%20Secure%20Flow-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/dependabotsecureflow)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 🎯 Overview

**Dependabot Secure Flow** is a GitHub Action that automatically manages dependency updates through a secure, batched workflow. Instead of merging Dependabot PRs directly to `main`, this action:

1. ✅ Auto-merges Dependabot PRs into a `securite` branch
2. 🧪 Validates builds and tests
3. 📦 Batches multiple updates together
4. 📝 Auto-generates changelog entries
5. 🚀 Creates a single PR to `main` for review

## 🚀 Features

- **Security-First**: All updates are validated before reaching `main`
- **Batch Processing**: Multiple dependency updates are grouped together
- **Auto-Correction**: Failed builds automatically close problematic PRs
- **Changelog Automation**: Automatic timestamp and changelog updates
- **Zero Configuration**: Works out of the box with sensible defaults

## 📦 Installation

### 1. Add the workflow to your repository

Create `.github/workflows/dependabot-secure-flow.yml`:

```yaml
name: Dependabot Secure Flow

on:
  pull_request:
    types: [opened, synchronize]
    paths:
      - 'package.json'
      - 'package-lock.json'
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  auto-merge-to-securite:
    uses: EthanThePhoenix38/dependabot-secure-flow/.github/workflows/dependabot-secure-flow.yml@main
    secrets: inherit
```

### 2. Configure Dependabot

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
```

### 3. Add required scripts to `package.json`

```json
{
  "scripts": {
    "build": "echo 'Build step' && exit 0",
    "test": "echo 'Test step' && exit 0"
  }
}
```

## 🔧 How It Works

```mermaid
graph LR
    A[Dependabot PR] --> B{Build Valid?}
    B -->|Yes| C[Merge to securite]
    B -->|No| D[Close PR]
    C --> E[Batch with others]
    E --> F[Create PR to main]
    F --> G[Manual Review]
    G --> H[Merge to main]
```

1. **Dependabot creates a PR** with dependency updates
2. **Workflow validates** the build (`npm install && npm run build`)
3. **Auto-merge to `securite`** if validation passes
4. **Close PR** if validation fails (with label `skipped-vulnerability`)
5. **Batch updates** accumulate in the `securite` branch
6. **Create PR to `main`** for final review
7. **Update documentation** (README timestamp, CHANGELOG)

## 📋 Requirements

- Node.js project with `package.json`
- `npm run build` script (can be a no-op: `echo 'No build'`)
- GitHub repository with Dependabot enabled

## ⚙️ Configuration

### Optional: Customize the workflow

You can override default behavior by modifying the workflow file:

```yaml
jobs:
  check-interdependencies:
    steps:
      - name: Auto-Correction & Validation
        run: |
          npm install --prefer-offline --no-audit
          npm run build
          npm test  # Add your test command
```

### Branch Strategy

- `main`: Production-ready code
- `securite`: Staging area for dependency updates
- `dependabot/*`: Temporary branches (auto-deleted after merge)

## 🛡️ Security

This action follows security best practices:

- ✅ No external dependencies (uses only GitHub Actions)
- ✅ Validates all updates before merging
- ✅ Auto-closes PRs that fail validation
- ✅ Requires manual review before reaching `main`

## 📊 Example Output

When the workflow runs successfully:

```
✅ Merged into securite branch for batch processing.
📝 Updated CHANGELOG.md with timestamp
🎉 Created PR #42: chore: dependency updates batch
```

## Configuration Options

| Input | Description | Required | Default |
| :--- | :--- | :--- | :--- |
| `github-token` | Token to manage PRs (GITHUB_TOKEN) | Yes | N/A |
| `node-version` | Node versions to use | No | 20 |
| `test-command` | Command to run for validation | No | `npm install && npm run build` |

---

## 🤝 Contributing

Contributions are welcome! This action is maintained in the [AI-Pulse](https://github.com/ThePhoenixAgency/AI-Pulse) repository and automatically synced here.

## 📄 License

MIT License - see [LICENSE](LICENSE) for details

## 🔗 Links

- [GitHub Marketplace](https://github.com/marketplace/actions/dependabotsecureflow)
- [Source Repository](https://github.com/EthanThePhoenix38/dependabot-secure-flow)
- [Issue Tracker](https://github.com/EthanThePhoenix38/dependabot-secure-flow/issues)


---

## Support This Project

If this action helps secure your projects, support the development:

[![GitHub Sponsors](https://img.shields.io/badge/Sponsor_on-GitHub-ea4aaa?style=for-the-badge&logo=github)](https://github.com/sponsors/EthanThePhoenix38)
[![Patreon](https://img.shields.io/badge/Support_on-Patreon-F96854?style=for-the-badge&logo=patreon)](https://patreon.com/EthanThePhoenix)
[![PayPal](https://img.shields.io/badge/Support_via-PayPal-00457C?style=for-the-badge&logo=paypal)](https://www.paypal.com/paypalme/VanessaBernier)
[![Ko-fi](https://img.shields.io/badge/Support_on-Ko--fi-F16061?style=for-the-badge&logo=ko-fi)](https://ko-fi.com/EthanThePhoenix)


[![Support via Patreon](https://img.shields.io/badge/Patreon-Support%20Development-f96854?logo=patreon&logoColor=white)](https://www.patreon.com/EthanThePhoenix)

**Your support helps fund the server and AI development!**
In exchange, I will add a link to your GitHub profile in the Contributors section.

You can also :

- ⭐ **Star this repository**
- 🐛 **Report issues** to help improve it
- 🔀 **Fork it** to customize for your needs
- 🐛 **Report issues** to help improve it

---

## Professional Page
[https://thephoenixagency.github.io](https://thephoenixagency.github.io)

---

**Made with ❤️ by [ThePhoenixAgency](https://github.com/ThePhoenixAgency)**
