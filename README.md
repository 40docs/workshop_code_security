# FortiCNAPP Code Security Documentation

This repository contains comprehensive hands-on lab documentation for **Lacework FortiCNAPP Code Security**, providing practical guidance for implementing security-first development practices.

## 🔒 What is FortiCNAPP Code Security?

**Lacework FortiCNAPP Code Security** is a comprehensive suite that secures your application development lifecycle from code to cloud. It identifies insecure patterns, third-party vulnerabilities, and misconfigurations **before deployment**, enabling organizations to shift security left.

### Key Capabilities

- **🔍 SAST** – Static Application Security Testing to identify insecure code patterns
- **📦 SCA** – Software Composition Analysis for vulnerable packages, secrets detection, SBOM generation, and license compliance
- **🏗️ IaC Scanning** – Infrastructure as Code security validation for Terraform, CloudFormation, and more
- **🤖 SmartFix** – Intelligent vulnerability remediation recommendations
- **⚡ OPAL Engine** – Advanced IaC static analysis based on Open Policy Agent (OPA)

## 📚 Documentation Structure

### Hands-On Labs

| Lab | Topic | Description |
|-----|-------|-------------|
| [00-prerequisites](hands-on-labs/00-prerequisites.md) | Setup | CLI installation, API configuration, VS Code extension setup |
| [01-sca](hands-on-labs/01-sca.md) | Software Composition Analysis | Dependency scanning, SBOM generation, license compliance, SmartFix |
| [02-iac](hands-on-labs/02-iac.md) | Infrastructure as Code | Terraform/CloudFormation security scanning |
| [03-sast](hands-on-labs/03-sast.md) | Static Application Security Testing | Code pattern analysis and vulnerability detection |
| [04-opal](hands-on-labs/04-opal.md) | OPAL Engine | Advanced policy-based IaC analysis |

### Content Features

- **🎯 Platform Support Matrix** – CLI, IDE, and SCM integration coverage
- **🛠️ Practical Examples** – Working code samples and configuration files
- **🔗 Integration Guides** – GitHub, GitLab, VS Code, and CI/CD workflows
- **📊 Real-World Scenarios** – Vulnerability remediation and compliance workflows

## 🚀 Quick Start

### 1. Install the Lacework CLI

```bash
# macOS/Linux
curl https://raw.githubusercontent.com/lacework/go-sdk/main/cli/install.sh | sudo bash

# Or via Homebrew
brew install lacework/tap/lacework-cli
```

### 2. Configure API Access

```bash
lacework configure
# Follow the interactive prompts or use: lacework configure -j /path/to/key.json
```

### 3. Install Components

```bash
lacework component install iac  # Infrastructure as Code scanning
lacework component install sca  # Software Composition Analysis
```

### 4. Run Your First Scan

```bash
# Software Composition Analysis
lacework sca scan ./

# Infrastructure as Code
lacework iac scan ./
```

## 🧪 Hands-On Lab Environment

### Create Test Repository

```bash
# Clone the vulnerable test application
gh repo create lab_forticnapp_code_security --template 40docs/lab_forticnapp_code_security --public
cd lab_forticnapp_code_security
```

### VS Code Integration

1. Install the **Lacework Security** extension
2. Configure with your API credentials
3. Open your project and run scans directly in the IDE
4. View inline vulnerability annotations and SmartFix recommendations

### GitHub Integration

```bash
# Trigger SmartFix demonstration via Pull Request
echo "paramiko==2.4.1" >> app/requirements.txt
git checkout -b trigger-smartfix
git add app/requirements.txt
git commit -m "Add vulnerable dep to trigger SmartFix"
git push -u origin trigger-smartfix
gh pr create --fill
```

## 🌐 Platform Coverage

### Language Support
- **C/C++** (Conan)
- **.NET** (DotNet Core, NuGet) 
- **Go** (Go modules)
- **Java** (Maven, Gradle, Bazel)
- **Node.js** (NPM, Yarn, PNPM)
- **PHP** (Composer)
- **Python** (Pip, Pipenv, Poetry)
- **Ruby** (Bundler)
- **Rust** (Cargo)

### Integration Points
- **IDEs**: VS Code extension with real-time feedback
- **SCM**: GitHub, GitLab, Bitbucket native integrations
- **CI/CD**: GitHub Actions, GitLab CI, Azure DevOps, Jenkins
- **CLI**: Cross-platform command-line interface

## 🏗️ Architecture Context

This repository is part of the **40docs platform** - an enterprise Documentation as Code ecosystem. The content here is:

- **Deployed via GitOps** using Flux v2 to Kubernetes
- **Built with MkDocs** for responsive documentation
- **Integrated with Azure** for scalable hosting
- **Version controlled** with Git submodules

## 🤝 Contributing

This documentation follows the 40docs platform standards:

- **Markdown**: Use proper front-matter and Material Design classes
- **Images**: Store screenshots in `hands-on-labs/images/`
- **Testing**: Validate all CLI commands and configurations
- **Security Focus**: Emphasize defensive security practices

## 📖 Additional Resources

- [Lacework FortiCNAPP Documentation](https://docs.lacework.com/)
- [FortiCNAPP Console](https://lacework.net/)
- [CLI Installation Guide](https://github.com/lacework/go-sdk)
- [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=lacework-security.lacework)

## 🛡️ Security Notice

This documentation includes examples of vulnerable code and configurations for educational purposes. These examples should:

- **Never be used in production environments**
- **Only be deployed in isolated test environments** 
- **Be used to understand vulnerability patterns and remediation techniques**

---

*Part of the 40docs Documentation as Code platform*