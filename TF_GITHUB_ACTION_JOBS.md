# Terraform GitHub Action Workflow - Jobs Summary

## Overview

This document describes the **Terraform Module CI** workflow (`.github/workflows/terraform-flow.yml`) that automatically validates Terraform code and keeps module documentation in sync on pull requests.

---

## Workflow Triggers

The workflow runs on **pull requests** when changes are made to:
- `**/*.tf` - Any Terraform files
- `.terraform-docs.yml` - Terraform-docs configuration
- `README.md` - Module documentation
- `.github/workflows/terraform-flow.yml` - The workflow file itself

---

## Jobs

### 1. **validate** Job

**Purpose:** Ensures Terraform code follows best practices and is syntactically valid.

**Steps:**
1. **Checkout** - Clone the PR branch
2. **Setup Terraform** - Install Terraform v1.6.6
3. **Terraform format & validate** - Runs:
   - `terraform fmt -recursive` - Auto-format all .tf files
   - `terraform init -backend=false` - Initialize without backend
   - `terraform validate` - Validate configuration syntax

**Success Criteria:** All Terraform files are properly formatted and valid.

---

### 2. **docs** Job

**Purpose:** Ensures README.md documentation is automatically generated from Terraform module source files and stays up-to-date.

**Steps:**

#### Step 1: Checkout PR Branch
```yaml
uses: actions/checkout@v4
with:
  ref: ${{ github.event.pull_request.head.ref }}
```
Checks out the PR branch to ensure latest code is tested.

#### Step 2: Install terraform-docs v0.20.0
```bash
tmp_dir=$(mktemp -d)
curl -sSL https://github.com/terraform-docs/terraform-docs/releases/download/v0.20.0/terraform-docs-v0.20.0-linux-amd64.tar.gz -o "$tmp_dir/terraform-docs.tar.gz"
tar -xzf "$tmp_dir/terraform-docs.tar.gz" -C "$tmp_dir"
sudo mv "$tmp_dir/terraform-docs" /usr/local/bin/terraform-docs
rm -rf "$tmp_dir"
```
- Downloads v0.20.0 binary
- Extracts to temp directory (prevents accidental file overwrites)
- Moves only the binary to `/usr/local/bin`
- Cleans up temp directory

**Critical Detail:** Uses **v0.20.0** to match local installation and ensure consistent formatting.

#### Step 3: Generate terraform-docs
```bash
terraform-docs .
```
Reads `.terraform-docs.yml` configuration and regenerates the Inputs/Outputs table in README.md between `<!-- BEGIN_TF_DOCS -->` and `<!-- END_TF_DOCS -->` markers.

#### Step 4: Verify README is Up-to-Date
```bash
if ! git diff --exit-code README.md; then
  echo "❌ README.md is not up-to-date with terraform-docs"
  echo ""
  echo "Changes detected:"
  git diff README.md
  echo ""
  echo "Please run 'terraform-docs .' locally and commit the changes"
  exit 1
fi
echo "✅ README.md is up-to-date"
```
- Compares local README.md with version in git index
- **Fails the job** if terraform-docs detected missing/incorrect variables
- Displays the diff to help developers see what changed
- **Passes** only if documentation is synchronized

**Success Criteria:** All Terraform variables and outputs are documented in README.md.

---

## terraform-docs Configuration (.terraform-docs.yml)

The workflow uses the following configuration:

```yaml
formatter: markdown table
output:
  file: README.md
  mode: inject
  template: |-
    <!-- BEGIN_TF_DOCS -->
    {{ .Content }}
    <!-- END_TF_DOCS -->
sections:
  hide:
    - providers
    - requirements
    - modules
    - resources
    - data-sources
settings:
  anchor: true
  indent: 2
  sort:
    enabled: true
    by: name
```

**Key Features:**
- **Formatter:** Markdown table format
- **Output Mode:** `inject` - Updates README.md between markers
- **Hidden Sections:** Only shows Inputs and Outputs
- **Anchors:** Enables clickable links in documentation table
- **Sorted:** Variables sorted alphabetically by name

---

## Problem-Solving Session Summary

### Issue 1: Terraform-docs GitHub Action Not Honoring Config File
**Problem:** The gh-action was using default settings instead of `.terraform-docs.yml`, and ignoring template configuration.

**Solution:** Replaced gh-action with direct CLI installation for explicit control over execution.

### Issue 2: Tarball Extraction Overwriting Repository Files
**Problem:** When extracting `terraform-docs-v0.20.0-linux-amd64.tar.gz` directly in the repo, it unpacked the entire terraform-docs project (including README.md), overwriting the module's README.

**Solution:** Extract to a temporary directory using `mktemp -d`, move only the binary to `/usr/local/bin`, then clean up.

```bash
tmp_dir=$(mktemp -d)
# Extract to temp
tar -xzf "$tmp_dir/terraform-docs.tar.gz" -C "$tmp_dir"
# Move only binary
sudo mv "$tmp_dir/terraform-docs" /usr/local/bin/terraform-docs
# Clean up
rm -rf "$tmp_dir"
```

### Issue 3: Version Mismatch Between Local and CI
**Problem:** Local environment had terraform-docs v0.20.0, but CI used v0.18.0. Different versions format HTML tags differently:
- v0.18.0: `<br>` (no trailing slash)
- v0.20.0: `<br/>` (self-closing tag)

This caused `git diff --exit-code` to always fail even when README.md was correctly updated.

**Solution:** Updated CI to use the same version (v0.20.0) as the local environment.

```yaml
curl -sSL https://github.com/terraform-docs/terraform-docs/releases/download/v0.20.0/terraform-docs-v0.20.0-linux-amd64.tar.gz
```

### Issue 4: Job Not Failing When README Missing Variables
**Problem:** Initial workflow didn't properly detect when README.md was out-of-sync.

**Solution:** Implemented explicit `git diff --exit-code` check with clear error messaging showing exactly what changed.

---

## Developer Workflow

When adding or modifying variables/outputs:

1. **Update Terraform files** (variables.tf, outputs.tf)
2. **Run locally:**
   ```bash
   terraform-docs .
   ```
3. **Commit changes:**
   ```bash
   git add variables.tf outputs.tf README.md
   git commit -m "feat: add new variable; updated docs"
   ```
4. **Push and create PR** - Workflow will verify docs are in sync

If the docs job fails:
- Review the diff shown in the workflow logs
- Run `terraform-docs .` locally again
- Commit and push the updated README.md
- Re-run the failed job in GitHub

---

## Maintenance Notes

- **terraform-docs version:** v0.20.0 - Keep CI and local installations aligned
- **Configuration file:** `.terraform-docs.yml` - Update to customize documentation output
- **Marker comments:** `<!-- BEGIN_TF_DOCS -->` and `<!-- END_TF_DOCS -->` - Do not remove
- **Line endings:** Ensure LF line endings for README.md (configured in git)

---

## Related Files

- [.github/workflows/terraform-flow.yml](.github/workflows/terraform-flow.yml) - Workflow definition
- [.terraform-docs.yml](.terraform-docs.yml) - Documentation generation config
- [README.md](README.md) - Module documentation (auto-generated between markers)
- [variables.tf](variables.tf) - Input variables (source for docs)
- [outputs.tf](outputs.tf) - Output values (source for docs)
