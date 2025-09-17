---
buttons:
  - title: Hands on Lab - Download
    icon: material-file-download-outline
    attributes:
      class: md-content__button md-icon
      href: ../hands-on-labs.pdf
      target: _blank
---

# Static Application Security Testing (SAST)

Lacework FortiCNAPP’s **SAST engine** analyzes application source code to detect insecure coding patterns, logic flaws, and potential injection points—before they reach runtime.

---

## What Is SAST?

SAST (Static Application Security Testing) reviews your code without executing it, scanning for:

* ❌ Hardcoded secrets and credentials
* ❌ Command and SQL injection
* ❌ Insecure crypto usage
* ❌ Dangerous APIs (e.g., `eval`, `os.system`)
* ETC

!!! tip "Shift Security Left"
    Lacework SAST integrates directly into GitHub, CI/CD, and VS Code, helping developers fix issues where they code.

---

## Supported Languages

| Language   | File Types                   | Scan Modes     |
| ---------- | ---------------------------- | -------------- |
| Python     | `.py`                        | GitHub, VSCode |
| JavaScript | `.js`, `.jsx`, `.ts`, `.tsx` | GitHub, VSCode |
| Go         | `.go`                        | GitHub, VSCode |
| Java       | `.java`, `.kt`               | GitHub         |
| Shell/Bash | `.sh`                        | GitHub         |

!!! note
    Language support varies by integration. VS Code may support more granular detection than GitHub alone.

---

## Integration Options

### SCMs

* ✅ GitHub (via OAuth App)
* ✅ GitLab (OAuth or Git integration)
* ✅ Bitbucket (via app)

### Dev Environments

* ✅ Visual Studio Code Extension
* ✅ Lacework CLI (SAST support in preview)

---

## Hands-On

This walkthrough demonstrates SAST findings using a vulnerable Python app.

```txt
lab_forticnapp_code_security/
├── app/
│   ├── app.py              # Flask app with all insecure routes
│   ├── config.py           # ❌ Hardcoded AWS secret
│   ├── Dockerfile          # ❌ Insecure container config
│   ├── requirements.txt    # ❌ Known vulnerable packages
│   └── vuln_app.py         # ❌ Insecure helper functions (SAST triggers)
├── terraform/
│   ├── resource_aws_s3_bucket.tf   # ❌ Public S3 bucket
│   └── resource_aws_subnet.tf      # ❌ Public subnet
├── README.md
```

---

### Step 1: Create a New Project from the Template

```bash
gh repo create lab_forticnapp_code_security --template 40docs/lab_forticnapp_code_security --public
cd lab_forticnapp_code_security
```

---

### Step 2: Open in VS Code

1. Launch **Visual Studio Code**
2. Open the project folder
3. Install the **Lacework Security** extension
4. Sign in with **OAuth**
5. Click the **shield icon** in the sidebar
6. Choose **Start SAST Scan**

!!! tip "Inline Results"
    Findings appear inline with red highlights. Hover to see explanations and **SmartFix** suggestions.

---

### Code Example (Python)

```python
# app/vuln_app.py

import os
import sqlite3
import random
import hashlib
import pickle

# ❌ Command injection
def list_files(user_input):
    os.system(f"ls {user_input}")

# ❌ SQL Injection
def get_user(username):
    conn = sqlite3.connect('example.db')
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE name = '%s'" % username)
    return cursor.fetchall()

# ❌ Weak cryptography
def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

if __name__ == "__main__":
    print("Token:", generate_token())
```

---

### Step 3: Run a SAST Scan via CLI

Use the **Lacework CLI** to run a SAST scan locally.

#### Requirements

* [Lacework CLI](00-prerequisites.md) installed and API Credentials configured

---

#### Run a Basic SAST + SCA Scan

```bash
lacework sca scan ./app
```

This scans the `app/` directory for:

* ❌ SAST issues (e.g., SQL injection, command injection, weak crypto)
* 📦 Vulnerable dependencies (SCA)
* 🔒 Secrets (if enabled)

!!! note "Default Behavior"
    `--sast` is enabled by default. You can disable it with `--sast=false`.

---

#### Example with Output Formats

```bash
lacework sca scan ./app --formats=lw-json,sarif,md-summary --output=./sca-results
```

This produces:

* A **Lacework JSON** results file (`lw-json`)
* A **SARIF** file for GitHub Code Scanning
* A **Markdown summary** for CI logs or docs

---

#### Disable Secrets or License Checks (Optional)

```bash
lacework sca scan ./app --secret=false --license-detection=false
```

---

#### Scan a GitHub Repo via URL

```bash
lacework sca scan https://github.com/your-org/your-repo.git --basic-auth=git:your_token
```

!!! warning "Git Auth Required"
    Use `--basic-auth`, `--ssh-auth`, or environment variables to authenticate when scanning remote Git URLs.

---

### After the Scan

* Review CLI output or generated files
* View line-level SAST issues and associated CVEs
* Apply **SmartFix** recommendations where available

!!! tip "SmartFix in CLI"
    SCA findings include SmartFix upgrade suggestions in `lw-json` output and in the Lacework Console if `--save-results=true` is used.

---

## Best Practices

* ✅ Run SAST in PRs and local development
* ✅ Use branch protection + FortiCNAPP checks to block unsafe merges
* ✅ Adopt SmartFix to reduce triage time
