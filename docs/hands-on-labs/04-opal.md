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

Welcome to the FortiCNAPP OPAL Lab. This guide demonstrates why generic security scanning - while necessary - isn't sufficient for enterprise security. You need custom OPAL policies to enforce your organization's specific requirements.

![IaCvsPaC](images/magicians4.png)

> **Standard scanners perform basic security tricks. OPAL delivers real security magic - enforcing YOUR business
  requirements.**

---

## OPAL Overview

OPAL is FortiCNAPP's IaC static analyzer based on the **OPA framework** and **Rego** language. OPAL evaluates infrastructure as code (IaC) files for potential AWS, Azure, Google Cloud, and Kubernetes security and compliance violations prior to deployment.

!!! info "Supported Frameworks"
    | Framework                    | Format         |
    |------------------------------|----------------|
    | Azure Resource Manager (ARM) | JSON           |
    | CloudFormation               | JSON, YAML     |
    | Kubernetes                   | YAML           |
    | Terraform                    | HCL, JSON Plan |

OPAL is part of the IaC component within the FortiCNAPP CLI. The FortiCNAPP CLI works with CI/CD tools such as **Jenkins**, **Circle CI**, **GitHub Actions**, and **GitLab Pipelines**.

---

## The Security Gap

**Surface-Level vs. Deep Analysis**

> ![gap](images/gap.png)

Standard IaC scanners lack visibility into what actually matters - they're flying blind when it comes to YOUR unique requirements and constraints.

**What Generic Scanners Find:**

- Open ports to 0.0.0.0/0 (obvious misconfigurations)
- Missing encryption (checkbox compliance)
- Public S3 buckets (well-known risks)

**What YOUR Business Actually Needs:**

- Environment-Specific Rules: Production workloads MUST use YOUR approved configurations
- Organizational Standards: Resources MUST follow YOUR team's naming and tagging requirements
- Custom Compliance: Systems MUST integrate with YOUR specific security and monitoring tools
- Business Logic: Critical services MUST meet YOUR defined reliability and recovery standards

**The Reality:** Generic scanners are like having spell-check review your contracts - helpful, but they won't catch that you forgot a critical business clause.

---

## What You'll Learn

- Why standard IaC scanning isn't enough for enterprise security
- How to create custom OPAL policies that enforce YOUR business logic
- The critical importance of testing both pass AND fail scenarios
- How OPAL catches violations that standard scanners miss

---


## Part 1: The Problem - When Standard Scanning Isn't Enough

### Your Organization's Requirements

Let's say your company has specific security requirements:

- Production web servers MUST use specific security groups for audit logging
- Development and production resources must NEVER share security groups
- ALL production instances require an audit logging security group

### Exercise: Run Standard IaC Scan

First, examine infrastructure that looks secure to standard scanners:

```bash
# Clone the demo repository
git clone https://github.com/40docs/lab-forticnapp-opal.git
cd lab-forticnapp-opal

# Look at the "secure" infrastructure
cat terraform/looks_secure.tf

# Run standard IaC scan - see only failures
lacework iac scan -d terraform/ 2>&1 | grep "false.*false"
```

**What Standard Scanning Finds:**

- ✅ Most checks PASS
- ⚠️ Only 3 minor issues:
  - S3 bucket missing cross-region replication (Medium)
  - S3 bucket missing access logging (Medium)
  - EC2 not EBS-optimized (Low)

**What's Actually Wrong (that standard scanning missed):**

- ❌ Production server missing required audit-logging security group
- ❌ Security group exists but isn't attached to production instances
- ❌ Business compliance requirement completely ignored

!!! warning "The Gap"
    Standard scanners found 3 minor operational issues but missed a critical business security requirement!

---

## Part 2: The Solution - Custom Policies with OPAL

### Understanding the Custom Policy

The cloned repository already contains a custom OPAL policy that enforces YOUR requirement. Let's examine how it works:

#### Step 1: Explore the Existing Policy Structure

```bash
# From the lab-forticnapp-opal directory
ls -la policies/opal/my_audit_policy/

# Review the policy metadata
cat policies/opal/my_audit_policy/metadata.yaml

# This shows:
# - policy_id: "my-audit-requirement"
# - severity: "High"
# - description: Production instances require audit logging
```

#### Optional: Create Your Own Policy

If you want to create a new custom policy from scratch:

```bash
# Create a new policy directory
mkdir -p policies/opal/my_custom_policy
cd policies/opal/my_custom_policy

# Create metadata file
cat > metadata.yaml <<EOF
policy_id: "my-custom-requirement"
title: "My Organization's Custom Requirement"
severity: "High"
description: "Custom security requirement for our organization"
resource_type: "aws_instance"
provider: "aws"
category: "Compliance"
EOF
```

#### Step 2: Examine the Policy Logic

```bash
# Review the existing policy logic
cat policies/opal/my_audit_policy/policy.rego

# The policy enforces:
# - Production instances MUST have audit-logging-sg
# - Non-production instances are exempt
# - Uses helper function to check security groups
```

For creating your own custom policy, here's the logic:

```bash
# If creating a new policy (my_custom_policy)
cat > policy.rego <<'EOF'
package policies.my_custom_policy

input_type := "tf"
resource_type := "aws_instance"
default allow = false

# Your custom requirement logic here
allow {
    # Define your business rules
    input.tags.YourRequirement == "YourValue"
}
EOF
```

---

## Part 3: Critical Step - Unit Testing Your Policy

!!! danger "Common Mistake"
    "My test passed, so my policy works!"

    **Reality**: A passing test only proves your policy accepts good configurations. It doesn't prove it rejects bad ones!

### Understanding the Test Cases

The repository includes comprehensive test cases. Let's examine them:

#### Review PASSING Test Cases

```bash
# See what configurations should PASS
ls -la policies/opal/my_audit_policy/terraform/tests/pass/
cat policies/opal/my_audit_policy/terraform/tests/pass/compliant.tf

# This shows production instances WITH audit-logging-sg (correct)
# And development instances without it (also correct)
```

#### Review FAILING Test Cases (CRUCIAL!)

```bash
# See what configurations should FAIL
ls -la policies/opal/my_audit_policy/terraform/tests/fail/
cat policies/opal/my_audit_policy/terraform/tests/fail/violations.tf

# This shows production instances WITHOUT audit-logging-sg (violation!)
```

#### Optional: Create Your Own Test Cases

If you created a custom policy, add test cases:

```bash
# For your custom policy
mkdir -p policies/opal/my_custom_policy/terraform/tests/pass
mkdir -p policies/opal/my_custom_policy/terraform/tests/fail

# Create a passing test
cat > policies/opal/my_custom_policy/terraform/tests/pass/good.tf <<'EOF'
# Configuration that meets YOUR requirements
resource "aws_instance" "compliant" {
  # ... your compliant configuration
}
EOF

# Create a failing test
cat > policies/opal/my_custom_policy/terraform/tests/fail/bad.tf <<'EOF'
# Configuration that violates YOUR requirements
resource "aws_instance" "violation" {
  # ... your non-compliant configuration
}
EOF
```

#### Test the Policy

```bash
# Test your policy
lacework iac policy test -d policies/opal/my_audit_policy

# Expected output:
# ✅ Pass test: compliant.tf - PASSED (policy accepted good config)
# ✅ Fail test: violations.tf - PASSED (policy rejected bad config)
```

!!! success "Testing Insight"
    If your fail test doesn't actually fail, your policy has a bug! This validates that your policy logic correctly rejects non-compliant configurations.

---

## Part 4: Victory Lap - Catching Real Issues

Now let's apply your tested policy to the original infrastructure:

### Standard Scan vs OPAL Custom Policy

```bash
# Standard scan - misses business logic violations
echo "=== STANDARD SCAN - What it catches ==="
lacework iac scan -d terraform/ 2>&1 | grep "false.*false"

# Output: Only operational issues
# - S3 bucket missing replication (operational)
# - S3 bucket missing logging (operational)
# - EC2 not EBS-optimized (performance)

echo "=== OPAL CUSTOM POLICIES - What YOU need caught ==="
lacework iac scan -d terraform/ --upload=false --custom-policy-dir=policies 2>&1 | grep "c-opl-my-audit"

# Output:
# c-opl-my-audit-policy  High  false  Production Audit Logging Requirement
# ^^^ YOUR CRITICAL BUSINESS REQUIREMENT VIOLATION CAUGHT!
```

### The Clear Difference

**Standard Scanning**: Found operational and performance issues
**OPAL Custom Policies**: Found YOUR specific business security violations

!!! example "Real-World Impact"
    Your infrastructure passed 90%+ of standard security checks but was violating a critical compliance requirement that could result in audit failures or security incidents.

---

## Advanced Usage

### Upload Custom Policies

```bash
# Upload your entire policy set to FortiCNAPP
lacework iac policy upload -d policies
```

### Integrate with CI/CD

```bash
# Run OPAL scan in your pipeline
lacework iac scan -d /path/to/terraform/project --upload=false --custom-policy-dir=policies
```

### Policy Development Best Practices

1. **Always test both pass AND fail scenarios**
2. **Use descriptive policy and violation messages**
3. **Start with deny-by-default security model**
4. **Test against real-world configurations**
5. **Version control your policies like code**

---

## Key Takeaway

> **"Standard IaC scanning is spell-check. OPAL is having your security architect review every change."**

![IACPAC](images/assets_task_01k5d8ek2hfg88j0gsbgrh7cm4_1758160498_img_1.webp)

Your infrastructure is only as secure as the rules you enforce. Generic rules give you generic security. Custom rules give you custom security - the kind YOUR business actually needs.

## Next Steps

1. Identify YOUR organization's specific security requirements
2. Write OPAL policies that enforce them
3. Test thoroughly with both pass AND fail cases
4. Integrate into your CI/CD pipeline
5. Sleep better knowing YOUR requirements are enforced

---

