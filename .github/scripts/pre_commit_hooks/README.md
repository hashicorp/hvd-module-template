# Pre-commit Hooks for Terraform

This directory contains custom bash scripts used as pre-commit hooks for Terraform module development. These hooks ensure code quality, consistency, and proper formatting before commits are made.

## Scripts Overview

### `terraform_fmt.sh`

Automatically formats Terraform files using `terraform fmt`.

**Features:**
- Processes multiple files passed as arguments
- Handles spaces in file paths by encoding/decoding them
- Runs `terraform fmt` on unique directory paths
- Specifically handles `.tfvars` files separately (as they're excluded by default `terraform fmt`)

**Usage:**
```bash
./terraform_fmt.sh <file1> <file2> ...
```

**What it does:**
1. Collects all unique directories from the provided file paths
2. Runs `terraform fmt` in each unique directory
3. Separately formats any `.tfvars` files explicitly

---

### `terraform_validate.sh`

Validates Terraform configuration files using `terraform validate`.

**Features:**
- Validates Terraform configurations only in directories containing `.tf` files
- Automatically finds the nearest parent directory with a `.terraform` directory (indicating initialization)
- Handles spaces in file paths
- Aggregates errors and exits with failure if any validation fails

**Usage:**
```bash
./terraform_validate.sh <file1> <file2> ...
```

**What it does:**
1. Collects all unique directories from the provided file paths
2. For each directory containing `.tf` files:
   - Searches upward to find the initialized Terraform directory (containing `.terraform/`)
   - Changes to the initialized directory
   - Runs `terraform validate` on the relative path
   - Reports any validation failures
3. Exits with error code 1 if any validation failed

---

### `terraform_tflint.sh`

Runs TFLint (Terraform linter) on Terraform files with optional arguments.

**Features:**
- Supports passing custom arguments to `tflint` via `--args` or `-a` flag
- Processes multiple files
- Handles spaces in file paths
- Runs `tflint` in each unique directory containing changed files
- Includes a pure-bash `getopt` implementation for argument parsing

**Usage:**
```bash
./terraform_tflint.sh [--args <tflint-args>] -- <file1> <file2> ...
```

**Example:**
```bash
./terraform_tflint.sh --args "--minimum-failure-severity=error" -- main.tf variables.tf
```

**What it does:**
1. Parses command-line arguments to extract optional `tflint` arguments
2. Collects all unique directories from the provided file paths
3. Runs `tflint` with the specified arguments in each unique directory

**Note:** This script includes a complete pure-bash implementation of `getopt` (version 1.4.3) to ensure cross-platform compatibility.

---

### `check_variables_consistency.sh`

Checks consistency between root `variables.tf` and example `variables.tf` files.

**Features:**
- Compares root `variables.tf` with all example configurations
- Shows colored diff output for review
- Non-blocking: exits successfully even when differences are found
- Informational only - helps reviewers spot potential inconsistencies

**Usage:**
```bash
./check_variables_consistency.sh <file1> <file2> ...
```

**What it does:**
1. Checks if root `variables.tf` or any `examples/*/variables.tf` files are being modified
2. If root `variables.tf` is modified:
   - Compares it with all example `variables.tf` files
   - Shows differences using `git diff --no-index`
3. If any example `variables.tf` is modified:
   - Compares it with the root `variables.tf`
4. Displays an informational message if differences are detected
5. Always exits successfully (exit code 0)

**Output Colors:**
- 🟡 Yellow: Indicates comparisons being performed or differences found
- 🟢 Green: Indicates files are identical
- 🔴 Red: (reserved for errors, not currently used)

---

## Integration with Pre-commit

These scripts are designed to be used with the [pre-commit](https://pre-commit.com/) framework. They should be referenced in your `.pre-commit-config.yaml` file.

### Example Configuration

```yaml
repos:
  - repo: local
    hooks:
      - id: terraform-fmt
        name: Terraform fmt
        entry: .github/scripts/pre_commit_hooks/terraform_fmt.sh
        language: script
        files: \.tf(vars)?$

      - id: terraform-validate
        name: Terraform validate
        entry: .github/scripts/pre_commit_hooks/terraform_validate.sh
        language: script
        files: \.tf$

      - id: terraform-tflint
        name: Terraform validate with tflint
        entry: .github/scripts/pre_commit_hooks/terraform_tflint.sh
        args: ['--args=--minimum-failure-severity=error']
        language: script
        files: \.tf$

      - id: check-variables-consistency
        name: Check variables.tf consistency
        entry: .github/scripts/pre_commit_hooks/check_variables_consistency.sh
        language: script
        files: variables\.tf$
```

## Requirements

- **Bash**: All scripts require bash shell
- **Terraform**: Must be installed and available in PATH
- **TFLint**: Required for `terraform_tflint.sh` (install from [tflint.github.io](https://github.com/terraform-linters/tflint))
- **Git**: Required for `check_variables_consistency.sh`

## Common Patterns

### Space Handling
All scripts handle spaces in file paths using the pattern:
```bash
file_with_path="${file_with_path// /__REPLACED__SPACE__}"
# ... processing ...
path="${path//__REPLACED__SPACE__/ }"
```

### Directory Deduplication
Scripts process unique directories to avoid running commands multiple times:
```bash
for path_uniq in $(echo "${paths[*]}" | tr ' ' '\n' | sort -u); do
  # ... process each unique path ...
done
```

### Error Handling
Most scripts use `set -e` to exit on errors, except `check_variables_consistency.sh` which is informational only.

## Development Notes

All scripts have been validated with [ShellCheck](https://www.shellcheck.net/) and follow shell scripting best practices:
- Variables are properly quoted to prevent word splitting
- Arrays are used appropriately
- Modern syntax like `((index+=1))` is preferred over `let`
- Exit codes are properly handled

## Contributing

When modifying these scripts:
1. Ensure they pass ShellCheck validation
2. Test with files containing spaces in their names
3. Verify error handling and exit codes
4. Update this README with any changes to functionality
