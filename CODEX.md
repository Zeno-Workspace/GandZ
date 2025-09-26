# ZENØ Codex Approval Flow

This document describes the ZENØ Codex approval workflow tool and how to use it for code review and integration.

## Overview

The `codex` tool provides an automated approval workflow system for the ZENØ project, designed to validate code changes, perform reviews, and execute clean squash-merge operations to maintain PR history integrity.

## Features

- **Approval Mode**: Full workflow automation from assignment to merge
- **Review Mode**: Interactive review process with integrity checks
- **Squash-Merge Only**: Maintains clean PR history per project constraints
- **Task Assignment**: Links workflows to specific task IDs (e.g., TSK-01)
- **Integrity Validation**: Ensures 100% system integrity before approval
- **ZENØ System Checks**: Validates core polyhedron system readiness

## Installation

The `codex` script is executable and self-contained. No additional dependencies required beyond standard Unix tools and Git.

```bash
# Make executable (if not already)
chmod +x codex

# Verify installation
./codex help
```

## Usage

### Basic Approval Workflow

Run the complete approval flow for a task:

```bash
# Run codex in approval mode for TSK-01
./codex approval --assign TSK-01
```

### Dry Run

See what the tool would do without executing:

```bash
./codex approval --assign TSK-01 --dry-run
```

### Review Only

Perform just the review process:

```bash
./codex review --assign TSK-01
```

### Verbose Output

Enable detailed logging:

```bash
./codex approval --assign TSK-01 --verbose
```

## Command Reference

```
USAGE:
    codex [MODE] [OPTIONS]

MODES:
    approval        Run in approval mode (default)
    review          Interactive review mode  
    help            Show help message

APPROVAL MODE OPTIONS:
    --assign TASK   Assign workflow to specific task (required)
    --merge-type    Merge type: squash (default), merge, rebase
    --dry-run       Show what would be done without executing
    --verbose       Enable verbose output
```

## Workflow Process

When running `codex approval --assign TSK-01`, the tool performs these steps:

1. **Environment Validation**
   - Verifies git repository status
   - Checks git user configuration
   - Validates working directory state

2. **Workflow Assignment**
   - Creates workflow file at `.codex/workflows/TSK-01.yml`
   - Records metadata (branch, commit, author, timestamp)
   - Sets approval mode and merge strategy

3. **Review Process**  
   - Runs integrity checks (validates 100% integrity)
   - Performs validation checks (ensures PASS status)
   - Checks ZENØ polyhedron system readiness
   - Verifies core HTML file exists and is accessible

4. **Squash-Merge**
   - Updates workflow status to "approved"
   - Creates clean commit with approval metadata
   - Maintains linear history per project constraints

## Workflow Files

Workflow assignments are stored in `.codex/workflows/` as YAML files:

```yaml
# Example: .codex/workflows/TSK-01.yml
workflow:
  id: TSK-01
  type: approval
  status: approved
  created: 2025-09-26T17:57:10Z
  
approval:
  mode: codex
  merge_strategy: squash
  required_checks:
    - integrity: 100%
    - validation: pass
    - review: approved
    
metadata:
  branch: copilot/fix-bb05fc64-2f86-4e79-b543-679b1034c1cd
  commit: 54c13c5566effced903f4011b8b905e8332c24b7
  author: copilot-swe-agent[bot]
  email: 198982749+Copilot@users.noreply.github.com
  completed: 2025-09-26T17:57:12Z
```

## Environment Variables

- `CODEX_CONFIG`: Path to codex configuration file (optional)
- `GIT_AUTHOR_NAME`: Override git author name if not configured globally
- `GIT_AUTHOR_EMAIL`: Override git author email if not configured globally

## Integration with ZENØ

The codex tool is specifically designed for the ZENØ Living Polyhedron System:

- Validates the core `zeno-polyhedron-refactored.html` file
- Ensures system integrity at 100%
- Maintains compatibility with the single-file architecture
- Respects the project's emphasis on clean, linear git history

## Constraints & Design

Per TSK-03 requirements:

- **Squash-merge only**: No rebase or merge commits allowed
- **Clean PR history**: Linear commit structure maintained
- **All checks pass**: 100% integrity required before merge
- **From repo root**: Tool must be run from repository root directory

## Troubleshooting

### Common Issues

**"Not in a git repository"**
- Ensure you're running from the repository root
- Verify `.git` directory exists

**"Git user.name not configured"** 
- Run `git config user.name "Your Name"`
- Or set `GIT_AUTHOR_NAME` environment variable

**"Repository has uncommitted changes"**
- Commit or stash changes before running approval flow
- Use `--dry-run` to test without making changes

**"Workflow file not found"**
- Ensure workflow was assigned first: `--assign TASK_ID`
- Check `.codex/workflows/` directory exists

### Validation Failures

If integrity or validation checks fail:

1. Verify the ZENØ HTML file is present and valid
2. Check repository status for unexpected changes  
3. Ensure all required files are committed
4. Run with `--verbose` for detailed diagnostics

## Examples

Complete approval workflow:
```bash
./codex approval --assign TSK-01
```

Test run without changes:
```bash
./codex approval --assign TSK-01 --dry-run
```

Review process only:
```bash
./codex review --assign TSK-01
```

Verbose approval with debugging:
```bash
./codex approval --assign TSK-01 --verbose
```

## Version History

- **v1.0.0**: Initial release with approval workflow, review mode, and squash-merge support