<!-- markdownlint-disable -->

# Hardening Report: gabe565--setup-helm-docs-action/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **gabe565--setup-helm-docs-action/v1.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The expression `${{ runner.os }}` is directly interpolated inside a `run:` shell script in a `case` statement. Any `${{ ... }}` expression directly in a `run:` block constitutes a script-injection risk because the value is substituted into the shell command string before the shell parses it. The fix is to use the environment variable `$RUNNER_OS` instead (e.g., `case "$RUNNER_OS" in`), which is already available as a pre-set environment variable and does not require template interpolation.

Locations:

- `action.yml:35`

### script-injection (severity: high)

Sub-rule (b): The shell variable `$VERSION` is used unquoted in the command `gh release view --json=name,assets $VERSION`. `VERSION` is sourced from `${{ inputs.version }}` via the `env:` block, making it workflow-controllable/untrusted data. An unquoted expansion allows an attacker to inject shell metacharacters (`;`, `|`, `&`, whitespace, etc.) through the `inputs.version` value. The fix is to quote the expansion: `gh release view --json=name,assets "$VERSION"`.

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection issues in action.yml: (a) Replaced `${{ runner.os }}` with the pre-set environment variable `$RUNNER_OS` in the case statement (line 35), eliminating template interpolation in the shell script. (b) Quoted `$VERSION` as `"$VERSION"` in the `gh release view` command (line 47) to prevent shell metacharacter injection from the user-controlled `inputs.version` value.

