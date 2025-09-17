---
buttons:
  - title: Hands on Lab - Download
    icon: material-file-download-outline
    attributes:
      class: md-content__button md-icon
      href: ../hands-on-labs.pdf
      target: _blank
---

# OPAL

Welcome to the FortiCNAPP OPAL Lab. This guide walks you through building, testing, and optionally uploading a custom OPAL policy using the FortiCNAPP CLI.

---

## Opal Overview

OPAL is FortiCNAPP’s infrastructure-as-code (IaC) static analyzer based on the **OPA** framework and **Rego** language. It evaluates AWS, Azure, GCP, and Kubernetes configurations for potential security and compliance violations **before deployment**.

You can create custom OPAL policies in Rego and scan your IaC with them using the FortiCNAPP CLI.

!!! info "Supported Frameworks"
    | Framework              | Format         |
    |------------------------|----------------|
    | Azure Resource Manager | JSON           |
    | CloudFormation         | JSON, YAML     |
    | Kubernetes             | YAML           |
    | Terraform              | HCL, JSON Plan |

OPAL supports CI/CD tools such as **Jenkins**, **CircleCI**, and **AWS CodePipeline** via the Lacework FortiCNAPP CLI.

### How It Works

- Converts your IaC into a normalized data structure
- Evaluates that structure against custom Rego policies
- Outputs results in **CLI**, **JSON**, or **JUnit.xml**
- Can be used standalone or integrated into pipelines
- Policy config can be toggled in `config.yaml` or the UI

---

## What You'll Learn

- How to install and configure the FortiCNAPP CLI
- How to generate a custom OPAL policy with tests
- How to run and debug policy test results
- How to upload policies and use them in scans

---

## Fast-Forward: Pressed on Time? Try This

Not interested in following the full guide and just want a working demo?

!!! tip "Skip the setup"
    A working OPAL policy, metadata, and test cases are included in a GitHub repo:

    🔗 [OPAL Demo](https://github.com/40docs/lab-forticnapp-opal)

Clone it and run:

```bash
gh repo create lab-forticnapp-opal --template 40docs/lab-forticnapp-opal.git
cd lab-forticnapp-opal/policies
lacework iac policy test -d opal/sample_custom_policy
```

## Setup

## Creating a Custom OPAL Policy

To begin using OPAL in FortiCNAPP, install the FortiCNAPP CLI and initialize a policy project.

## 1. Install CLI and IaC Component

!!! note
    Ensure the FortiCNAPP CLI is installed. OPAL is bundled within the IaC security component.

## 2. Start the Policy Wizard

```bash
lacework iac policy create
```

Respond to prompts with:

| Prompt                  | Value                  |
|------------------------|------------------------|
| Policies directory path | `../policies`         |
| Select provider        | `aws`                  |
| Select target          | `terraform`            |
| Policy name            | `sample_custom_policy` |
| Title                  | `Sample Custom Policy` |
| Category               | `logging`              |
| ResourceType           | `aws_s3_bucket`        |
| Severity               | `medium`               |

!!! info "Naming Convention"
    Policy names must be lowercase with underscores (e.g., `sample_custom_policy`).

A directory structure like this is created:

```text
../policies/opal/sample_custom_policy/terraform/
../policies/opal/sample_custom_policy/terraform/tests/
```

## 3. Review `metadata.yaml`

Located at:

```text
../policies/opal/sample_custom_policy/metadata.yaml
```

It includes:

```yaml
category: logging
checkTool: opal
checkType: terraform
description: "example policy"
provider: aws
severity: medium
title: "Sample Custom Policy"
```

## 4. Add OPAL Policy Logic

Edit `policy.rego`:

```rego
package policies.sample_custom_policy

input_type := "tf"
resource_type := "aws_s3_bucket"
default allow = false

allow {
  input.logging[_].target_bucket == "example"
}
```

## Demo

## Testing and running custom policies

Now that you've created your custom OPAL policy, test it with Terraform examples.

## 1. Create Test Inputs

### ✅ Passing Test

```bash
mkdir -p ../policies/opal/sample_custom_policy/terraform/tests/pass
```

Create `main.tf`:

```hcl
resource "aws_s3_bucket" "test" {
  bucket = "test-bucket"
  logging {
    target_bucket = "example"
  }
}
```

### ❌ Failing Test

```bash
mkdir -p ../policies/opal/sample_custom_policy/terraform/tests/fail
```

Create `main.tf`:

```hcl
resource "aws_s3_bucket" "test" {
  bucket = "test-bucket"
  logging {
    target_bucket = "bad-example"
  }
}
```

!!! note
    You can include either passing, failing, or both test types.

## 2. Run Policy Tests

```bash
lacework iac policy test -d ../policies/opal/sample_custom_policy
```

Expected output includes test results for each `pass/` and `fail/` case.

## 3. Debug with Rego Print

Add a print statement to your policy:

```rego
print(sprintf("target_bucket: %s", [input.logging[_].target_bucket]))
```

Output example:

```json
"print_statements": [
  {
    "Message": "target_bucket: example",
    "Line": 5,
    "PolicyFile": "/path/to/policy.rego"
  }
]
```

!!! warning "Avoid Committing Prints"
    Do not commit print statements to version control.

## 4. Upload Custom Policy

You can package and upload your entire policies directory:

```bash
lacework iac policy upload -d ../policies
```

This will replace all existing custom policies.

## 5. Use Policies with a Target Directory

Run an OPAL scan on any Terraform project:

```bash
lacework iac tf-scan opal --disable-custom-policies=false -d path/to/project
```

!!! tip
    If `-d` is not provided, the current directory is used.
