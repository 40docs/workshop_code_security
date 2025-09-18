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

![IaCvsPaC](images/magicians.webp)

> **Standard scanners perform basic security tricks. OPAL delivers real security magic - enforcing YOUR business
  requirements.**

Welcome to the FortiCNAPP OPAL Lab. This guide demonstrates why generic security scanning - while necessary - isn't sufficient for enterprise security. You need custom OPAL policies to enforce your organization's specific requirements.

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

### The Security Gap

**Surface-Level vs. Deep Analysis**

Standard IaC scanners operate like automated spell-checkers - they catch obvious errors but miss context and meaning:

**What Generic Scanners Find:**

- Open ports to 0.0.0.0/0 (obvious misconfigurations)
- Missing encryption (checkbox compliance)
- Public S3 buckets (well-known risks)

**What YOUR Business Actually Needs:**

- Production systems MUST use specific audit-logging security groups
- Payment processing MUST be in isolated VPCs
- Only platform team CAN modify production load balancers
- Financial data MUST use company-managed KMS keys

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

### Creating Your Business Logic Policy

Let's create a policy that enforces YOUR requirement: "All production instances must have the audit-logging-sg security group"

#### Step 1: Create Policy Structure

```bash
cd policies/opal
mkdir my_audit_policy
cd my_audit_policy

# Create metadata file
cat > metadata.yaml <<EOF
policy_id: "my-audit-requirement"
title: "Production Audit Logging Requirement"
severity: "High"
description: "All production instances must have audit logging security group"
resource_type: "aws_instance"
provider: "aws"
category: "Compliance"
EOF
```

#### Step 2: Write the Policy Logic

```bash
cat > policy.rego <<'EOF'
package policies.my_audit_policy

input_type := "tf"
resource_type := "aws_instance"
default allow = false

# Production instances must have audit-logging-sg
allow {
    # Check if this is a production instance
    input.tags.Environment == "production"

    # Check if audit-logging-sg is present
    has_audit_logging_sg
}

# Non-production instances don't need audit logging
allow {
    input.tags.Environment != "production"
}

# Helper to check for audit logging security group
has_audit_logging_sg {
    sg_ref := input.vpc_security_group_ids[_]
    contains(sg_ref, "audit_logging_sg")
}
EOF

# IMPORTANT: Copy policy to terraform directory for testing
cp policy.rego terraform/
```

---

## Part 3: Critical Step - Unit Testing Your Policy

!!! danger "Common Mistake"
    "My test passed, so my policy works!"

    **Reality**: A passing test only proves your policy accepts good configurations. It doesn't prove it rejects bad ones!

### Creating Comprehensive Tests

#### Create PASSING Test Cases

```bash
mkdir -p terraform/tests/pass

cat > terraform/tests/pass/compliant.tf <<'EOF'
# This SHOULD pass - production instance with audit logging
resource "aws_instance" "good_production" {
  ami           = "ami-12345"
  instance_type = "t3.micro"

  vpc_security_group_ids = [
    aws_security_group.web_sg.id,
    aws_security_group.audit_logging_sg.id  # Required for production
  ]

  tags = {
    Name        = "prod-web-01"
    Environment = "production"
  }
}

# Supporting security groups
resource "aws_security_group" "web_sg" {
  name = "web-sg"
}

resource "aws_security_group" "audit_logging_sg" {
  name = "audit-logging-sg"
}
EOF
```

#### Create FAILING Test Cases (CRUCIAL!)

```bash
mkdir -p terraform/tests/fail

cat > terraform/tests/fail/violations.tf <<'EOF'
# This SHOULD fail - production instance WITHOUT audit logging
resource "aws_instance" "bad_production" {
  ami           = "ami-12345"
  instance_type = "t3.micro"

  vpc_security_group_ids = [
    aws_security_group.web_sg.id
    # MISSING: aws_security_group.audit_logging_sg.id
  ]

  tags = {
    Name        = "prod-web-02"
    Environment = "production"  # Production requires audit logging!
  }
}

# Supporting security group
resource "aws_security_group" "web_sg" {
  name = "web-sg"
}
EOF
```

#### Test Your Policy

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

