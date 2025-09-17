# Code Security

## Overview

**Lacework FortiCNAPP Code Security** is a comprehensive suite designed to secure your application development lifecycle—from code to cloud. It helps identify insecure patterns, third-party vulnerabilities, and misconfigurations **before deployment**, enabling organizations to shift security left.

Key components:

- **SAST** – Identify insecure code patterns
- **SCA** – Detect vulnerable open-source packages, secrets detection, SBOM, License compliance.
- **IaC Scanning** – Catch misconfigurations in Terraform, CloudFormation, etc. additional capabilities via OPAL engine (IaC static analyzer based on OPA).

Code Security is accessible via:

- CLI tool
- Docker image
- IDE extensions (e.g., VS Code)
- SCM integrations (e.g., GitHub, GitLab)

This guide is intended for **developers, DevOps, and security engineers** managing code repositories and pipelines.

---

## Platform Support: CLI & IDEs

### CLI (SAST, SCA, IaC)

| Environment       | Supported? |
|-------------------|------------|
| Linux (ARM/AMD)   | ✅ Yes     |
| Windows           | ❌ No      |
| macOS             | ✅ Yes     |

### IDE Support (VS Code)

| IDE       | IaC | SCA | SAST |
|-----------|-----|-----|------|
| **VS Code**   | ✅ Yes | ✅ Yes | ✅ Yes |
| **IntelliJ**  | ❌ No | ❌ No | ❌ No |

---

## SCM Integration Support

Lacework supports SCA, SAST, and IaC scanning via direct SCM integrations.

| SCM Provider | Supported? | Permissions Required | Scope        |
|--------------|------------|----------------------|--------------|
| GitHub       | ✅ Yes     | Owner                | Organizations |
| GitLab       | ✅ Yes     | Owner/Maintainer     | Projects      |
| Bitbucket    | ✅ Yes     | Admin                | Workspaces    |
| Azure Repos  | ❌ No      | N/A                  | N/A           |

---

## CI/CD Integration Matrix

Lacework CLI tools are CI/CD-friendly for both **SCA/SAST** and **IaC scanning**.

### SCA & SAST CI Integration

- Fully supported via Lacework CLI in:
  - GitHub Actions
  - GitLab CI
  - Bitbucket Pipelines
  - Jenkins
  - Most custom CI/CD platforms

### IaC CI Integration

| CI/CD Provider | IaC Scan Supported? |
|----------------|---------------------|
| GitHub Actions | ✅ Yes              |
| GitLab CI      | ✅ Yes              |
| Azure DevOps   | ✅ Yes              |
| Jenkins        | ✅ Yes              |
| Atlantis       | ✅ Yes              |

---

## Summary

- **Use CLI** for flexible local or CI/CD scanning with full control.
- **Use VS Code extension** for real-time feedback during development.
- **Integrate with GitHub/GitLab/Bitbucket** to scan on PRs, block insecure merges, and automate remediation.
- **IaC scanning** is production-ready for major CI platforms, with broader support coming.
