---
name: fix-vulnerabilities
description: Detect, validate, and fix security vulnerabilities from GitHub Dependabot alerts
---

# Fix Vulnerabilities Skill

A global skill to detect, validate, and fix security vulnerabilities reported by GitHub Dependabot.

## Trigger

Use this skill when the user asks to:

- Fix security vulnerabilities
- Address Dependabot alerts
- Update vulnerable dependencies
- Run `/fix-vulnerabilities`

## Prerequisites

- `gh` CLI authenticated with repo access
- Git repository with GitHub remote
- Write access to the repository

## Workflow Overview

```
1. Detect Ecosystem → 2. Fetch Vulnerabilities → 3. Validate Each → 4. Fix by Severity → 5. Verify → 6. Commit → 7. Summary
```

## Step 1: Detect Project Ecosystem

Detect the package ecosystem by checking for lockfiles AND project files together:

| Ecosystem | Lockfile            | Project File                     |
| --------- | ------------------- | -------------------------------- |
| pnpm      | `pnpm-lock.yaml`    | `package.json`                   |
| npm       | `package-lock.json` | `package.json`                   |
| yarn      | `yarn.lock`         | `package.json`                   |
| cargo     | `Cargo.lock`        | `Cargo.toml`                     |
| go        | `go.sum`            | `go.mod`                         |
| bundler   | `Gemfile.lock`      | `Gemfile`                        |
| pip       | `requirements.txt`  | `requirements.txt` or `setup.py` |
| poetry    | `poetry.lock`       | `pyproject.toml`                 |
| pipenv    | `Pipfile.lock`      | `Pipfile`                        |
| composer  | `composer.lock`     | `composer.json`                  |

**Detection command:**

```bash
ls -la | grep -E "(pnpm-lock|package-lock|yarn.lock|Cargo|go\.(mod|sum)|Gemfile|requirements|poetry|Pipfile|composer)"
```

**IMPORTANT:** Only handle ONE ecosystem per run. If multiple detected, ask user which to address.

## Step 2: Fetch Vulnerabilities from GitHub

```bash
# Get repo info
gh repo view --json nameWithOwner -q '.nameWithOwner'

# Fetch all open Dependabot alerts with full details
gh api repos/{owner}/{repo}/dependabot/alerts \
  --jq '.[] | select(.state == "open") | {
    number: .number,
    package: .dependency.package.name,
    ecosystem: .dependency.package.ecosystem,
    manifest: .dependency.manifest_path,
    severity: .security_advisory.severity,
    vulnerable_range: .security_vulnerability.vulnerable_version_range,
    patched_version: .security_vulnerability.first_patched_version.identifier,
    advisory_id: .security_advisory.ghsa_id,
    summary: .security_advisory.summary,
    url: .html_url
  }'
```

**Error handling for gh API:**

- If auth fails: "GitHub CLI authentication issue. Run `gh auth status` to check, then `gh auth login` to fix."
- If rate limited: "GitHub API rate limit hit. Wait a few minutes and retry."
- If repo not found: "Repository not found or no access. Verify you have access to this repo's security alerts."

Offer to retry if the user confirms. If persistent, investigate the specific error message.

## Step 3: Validate Each Vulnerability

For each alert, check if it's already fixed in the current lockfile:

### JavaScript/Node.js (pnpm/npm/yarn)

```bash
# Check installed version
pnpm why <package-name>
# or
npm ls <package-name>
# or
yarn why <package-name>
```

### Rust (cargo)

```bash
cargo tree -p <package-name>
```

### Go

```bash
go list -m all | grep <package-name>
```

### Python (pip/poetry/pipenv)

```bash
pip show <package-name>
# or
poetry show <package-name>
```

### Ruby (bundler)

```bash
bundle show <gem-name>
```

### PHP (composer)

```bash
composer show <package-name>
```

**Compare installed version against `patched_version`:**

- If installed >= patched → Already fixed, offer to dismiss
- If installed < patched → Needs fixing

## Step 4: Handle Already-Fixed Vulnerabilities

For vulnerabilities already patched in the lockfile, ask before dismissing:

**Use AskUserQuestion tool with this format:**

```
Question: "Dismiss vulnerability GHSA-xxxx (package-name)?"

Options:
1. "Yes, dismiss as inaccurate" - The lockfile already has package@1.2.3 which is >= the patched version 1.2.0. This alert is outdated.
2. "No, keep open" - Keep the alert open for manual review
3. "Skip for now" - Don't dismiss, continue to next vulnerability
```

**If user confirms dismissal:**

```bash
gh api repos/{owner}/{repo}/dependabot/alerts/{number} \
  -X PATCH \
  -f state=dismissed \
  -f dismissed_reason=inaccurate \
  -f dismissed_comment="Vulnerability already patched in lockfile. Installed version: {version} >= patched version: {patched}"
```

## Step 5: Fix Vulnerabilities by Severity

Process in order: **HIGH → MEDIUM → (report LOW)**

### Severity Handling

| Severity | Action                                          |
| -------- | ----------------------------------------------- |
| critical | Fix immediately (treat as high)                 |
| high     | Fix automatically                               |
| medium   | Fix automatically                               |
| low      | Skip and report, ask user for further procedure |

### Fix Strategies by Ecosystem

#### JavaScript/Node.js

**Strategy 1: Direct dependency update**

```bash
# pnpm
pnpm update <package-name>

# npm
npm update <package-name>

# yarn
yarn upgrade <package-name>
```

**Strategy 2: Transitive dependency override (if direct update doesn't resolve)**

For pnpm - add to `package.json`:

```json
{
  "pnpm": {
    "overrides": {
      "<package-name>": "^<patched-version>"
    }
  }
}
```

For npm - add to `package.json`:

```json
{
  "overrides": {
    "<package-name>": "^<patched-version>"
  }
}
```

For yarn - add to `package.json`:

```json
{
  "resolutions": {
    "<package-name>": "^<patched-version>"
  }
}
```

Then reinstall:

```bash
pnpm install  # or npm install / yarn install
```

**IMPORTANT:** Use caret (`^`) ranges for overrides, not unbounded (`>=`), to prevent unexpected major version upgrades.

#### Rust (cargo)

```bash
cargo update -p <package-name>
```

If that doesn't work, edit `Cargo.toml` to specify minimum version or use `[patch]` section.

#### Go

```bash
go get <package>@v<patched-version>
go mod tidy
```

#### Python

**pip:**

```bash
pip install --upgrade <package-name>>=<patched-version>
pip freeze > requirements.txt
```

**poetry:**

```bash
poetry update <package-name>
```

**pipenv:**

```bash
pipenv update <package-name>
```

#### Ruby (bundler)

```bash
bundle update <gem-name>
```

Or edit `Gemfile` to specify minimum version.

#### PHP (composer)

```bash
composer update <package-name>
```

### Breaking Changes

If a fix requires a **major version bump**, STOP and call out:

**Use AskUserQuestion:**

```
Question: "Fixing {package} requires major version upgrade ({current} → {patched}). This may introduce breaking changes."

Options:
1. "Attempt upgrade" - Try the upgrade and rely on quality checks to catch issues
2. "Skip this vulnerability" - Leave unfixed, document in summary
3. "Show changelog" - I'll fetch the changelog/release notes first (if available)
```

If user chooses to attempt and it breaks tests, analyze the failure and propose next steps.

## Step 6: Run Quality Checks

### Discover Available Checks

**For JavaScript/Node.js**, check `package.json` scripts:

```bash
cat package.json | jq '.scripts | keys[]' | grep -E "(typecheck|type-check|tsc|lint|test|build|check)"
```

Common scripts to look for: `typecheck`, `lint`, `test`, `build`, `check`, `ci`

**For other ecosystems:**

| Ecosystem  | Common Check Commands                             |
| ---------- | ------------------------------------------------- |
| cargo      | `cargo check`, `cargo test`, `cargo clippy`       |
| go         | `go build ./...`, `go test ./...`, `go vet ./...` |
| poetry/pip | `pytest`, `mypy`, `flake8`, `ruff`                |
| bundler    | `bundle exec rspec`, `bundle exec rubocop`        |
| composer   | `composer test`, `phpunit`, `phpstan`             |

**Also check:**

- `README.md` for documented test/build commands
- `Makefile` for common targets
- CI config files (`.github/workflows/*.yml`) for test commands

### Run Checks

Run discovered checks in order (typically: typecheck → lint → test → build).

**If checks fail:**

1. Analyze the error output
2. Determine if it's related to the dependency update
3. Propose options:

**Use AskUserQuestion:**

```
Question: "Quality checks failed after updating {package}. Error: {brief_error_summary}"

Options:
1. "Show full error" - Display complete error output for analysis
2. "Rollback this fix" - Revert changes to {package}, continue with other fixes
3. "Attempt auto-fix" - Try to resolve the issue (if it looks fixable)
4. "Keep changes anyway" - Proceed despite failing checks (not recommended)
```

## Step 7: Commit Changes

### Branch Naming

```
security/{primary-advisory-id}
```

Example: `security/GHSA-xxxx-yyyy` or `security/high-severity-2024-01-29` if multiple.

For multiple unrelated fixes, consider separate branches:

- `security/axios-dos-fix`
- `security/lodash-prototype-pollution`

### Commit Style

Use conventional commits format:

```
fix: {brief description of security fix}

{Detailed list of what was fixed}

Co-Authored-By: Claude <noreply@anthropic.com>
```

Example:

```
fix: address high-severity security vulnerabilities

- Update axios to ^1.12.0 (GHSA-4hjh-wcwx-xvwj: DoS via data: URLs)
- Add pnpm override for qs ^6.14.1 (GHSA-6rw7-vpxm-498p: arrayLimit bypass)
- Add pnpm override for glob ^11.1.0 (GHSA-5j98-mcp5-4vw2: CLI command injection)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Grouping Strategy

Group commits by:

1. **Complexity** - Simple version bumps together, overrides/patches separately
2. **Risk** - Keep potentially breaking changes isolated
3. **Related vulnerabilities** - Multiple vulns in same package = one commit

**Rationale:** Easier to review, test, and rollback if needed.

### Commit Process

1. Stage specific files (not `git add -A`):

```bash
git add package.json pnpm-lock.yaml  # or equivalent for ecosystem
```

2. Create commit with conventional format

3. Offer to create PR:

**Use AskUserQuestion:**

```
Question: "Changes committed to branch security/{name}. Create a pull request?"

Options:
1. "Yes, create PR" - Create PR with summary of fixes
2. "No, just push branch" - Push branch only, I'll create PR manually
3. "No, local only" - Keep changes local for now
```

### PR Title Convention

Use conventional commits format for the PR title:

**PR creation command:**

```bash
gh pr create --title "fix: address {severity}-severity security vulnerabilities" --body "..."
```

## Step 8: Generate Summary

Output a markdown summary with tables:

```markdown
## Security Vulnerability Fix Summary

### Fixed Vulnerabilities

| Severity | Package | Advisory            | Fix Applied           |
| -------- | ------- | ------------------- | --------------------- |
| High     | axios   | GHSA-4hjh-wcwx-xvwj | Updated to ^1.12.0    |
| High     | qs      | GHSA-6rw7-vpxm-498p | pnpm override ^6.14.1 |
| Medium   | lodash  | GHSA-xxxx-yyyy      | Updated to ^4.17.21   |

### Dismissed (Already Fixed)

| Package | Advisory            | Reason                        |
| ------- | ------------------- | ----------------------------- |
| glob    | GHSA-5j98-mcp5-4vw2 | Already at 11.1.0 in lockfile |

### Skipped (Low Severity)

| Package | Advisory       | Summary                       |
| ------- | -------------- | ----------------------------- |
| vite    | GHSA-xxxx-zzzz | Middleware file serving issue |

### Failed/Requires Manual Review

| Package  | Advisory       | Reason                                  |
| -------- | -------------- | --------------------------------------- |
| some-pkg | GHSA-aaaa-bbbb | Major version bump required, breaks API |

### Next Steps

- [ ] Review and merge PR #XXX
- [ ] Consider addressing low-severity issues in future sprint
- [ ] Monitor for new advisories
```

### Low Severity Follow-up

After reporting low severity issues, ask:

**Use AskUserQuestion:**

```
Question: "Found {N} low-severity vulnerabilities. How would you like to proceed?"

Options:
1. "Fix them now" - Attempt to fix low-severity issues as well
2. "Create tracking issue" - Create a GitHub issue to track these for later
3. "Ignore for now" - Document in summary only, no further action
4. "Review individually" - Go through each one and decide
```

## Monorepo Handling

If multiple `package.json` files exist (workspaces):

1. Identify which packages have vulnerabilities (from `manifest_path` in alert)
2. Group fixes by affected package
3. Handle each workspace's vulnerabilities separately if needed
4. Keep related fixes together when they affect the same vulnerability across packages

## Error Recovery

### Partial Success

If some fixes succeed and others fail:

**Use AskUserQuestion:**

```
Question: "Partial success: {N} vulnerabilities fixed, {M} failed. How to proceed?"

Options:
1. "Commit successful fixes" - Commit what worked, document failures in summary
2. "Rollback everything" - Revert all changes, investigate failures first
3. "Show failure details" - Review what went wrong before deciding
```

### Unexpected States

- If lockfile is corrupted after changes: Offer to restore from git and retry
- If package registry is down: Report and suggest retry later
- If vulnerability has no patched version: Report as "no fix available yet"

## Example Invocations

```
User: "Fix the security vulnerabilities in this repo"
User: "Address the Dependabot alerts"
User: "/fix-vulnerabilities"
User: "Can you fix the high severity security issues?"
```

## Notes

- Always verify the repo has a GitHub remote before starting
- Check `gh auth status` if API calls fail
- Prefer minimal changes - don't refactor or "improve" unrelated code
- Keep detailed notes of what was attempted for the summary
- If in doubt, ask the user rather than making assumptions
