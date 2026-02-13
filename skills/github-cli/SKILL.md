---
name: github-cli
description: Common workflows for GitHub CLI (gh) commands
---

# GitHub CLI Workflows

Quick reference for common GitHub CLI operations. Requires `gh` to be installed and authenticated.

**Documentation:** https://cli.github.com/manual/

## Prerequisites

If `gh` is not installed or not authenticated, instruct the user to run:

```bash
# Install gh (https://cli.github.com)
# macOS: brew install gh
# Linux: see https://github.com/cli/cli/blob/trunk/docs/install_linux.md

# Authenticate
gh auth login

# Verify authentication
gh auth status
```

## Pull Requests

### Create a pull request

```bash
gh pr create --title "My PR" --body "Description of changes"
```

### Create a draft pull request

```bash
gh pr create --title "WIP: Feature" --body "Work in progress" --draft
```

### List open pull requests

```bash
gh pr list
```

### View a pull request

```bash
gh pr view 123
gh pr view 123 --web  # Open in browser
```

### Check out a pull request locally

```bash
gh pr checkout 123
```

### Review a pull request

```bash
gh pr review 123 --approve
gh pr review 123 --request-changes --body "Please fix..."
gh pr review 123 --comment --body "Looks good but..."
```

### Merge a pull request

```bash
gh pr merge 123 --squash
gh pr merge 123 --squash --delete-branch
```

### List PR checks/status

```bash
gh pr checks 123
```

### View PR diff

```bash
gh pr diff 123
```

### List PR comments

```bash
gh api repos/{owner}/{repo}/pulls/123/comments
```

## Issues

### Create an issue

```bash
gh issue create --title "Bug report" --body "Description"
```

### List issues

```bash
gh issue list
gh issue list --assignee @me
gh issue list --label bug
```

### View an issue

```bash
gh issue view 456
gh issue view 456 --web
```

### Close an issue

```bash
gh issue close 456
```

### Add a comment to an issue

```bash
gh issue comment 456 --body "My comment"
```

## Repository Operations

### Clone a repository

```bash
gh repo clone owner/repo
```

### View repository info

```bash
gh repo view
gh repo view owner/repo --web
```

### List repositories

```bash
gh repo list lightspeed-hospitality --limit 50
```

### Create a repository

```bash
gh repo create my-repo --private
```

## Releases

### List releases

```bash
gh release list
```

### View a release

```bash
gh release view v1.0.0
```

### Create a release

```bash
gh release create v1.0.0 --title "Release 1.0.0" --notes "Release notes"
```

## Actions / Workflows

### List workflow runs

```bash
gh run list
gh run list --workflow build.yml
```

### View a workflow run

```bash
gh run view 12345
gh run view 12345 --log  # View logs
```

### Re-run a failed workflow

```bash
gh run rerun 12345
gh run rerun 12345 --failed  # Only re-run failed jobs
```

### Watch a running workflow

```bash
gh run watch 12345
```

## API Access

### Make arbitrary API calls

```bash
# GET request
gh api repos/{owner}/{repo}

# POST request
gh api repos/{owner}/{repo}/issues --method POST --field title="New issue" --field body="Description"

# GraphQL query
gh api graphql -f query='{ viewer { login } }'
```

### Paginate API results

```bash
gh api repos/{owner}/{repo}/pulls --paginate
```

## Search

### Search code

```bash
gh search code "function_name" --repo owner/repo
```

### Search issues

```bash
gh search issues "bug" --repo owner/repo --state open
```

### Search PRs

```bash
gh search prs "feature" --repo owner/repo --state open
```

## Tips & Notes

- Use `--json` flag with `--jq` for structured output: `gh pr list --json number,title --jq '.[].title'`
- Use `--web` to open any resource in your browser
- Set `GH_REPO` environment variable to avoid specifying `--repo` each time
- Use `gh auth status` to check your authentication state
- Use `gh alias set` to create custom shortcuts
- GitHub CLI respects `GITHUB_TOKEN` environment variable for authentication
