# Skills

Shared [OpenCode agent skills](https://opencode.ai/docs/skills/) for Lightspeed Hospitality.

## Usage

Copy a skill into your global config:

```bash
cp -r skills/fix-vulnerabilities ~/.config/opencode/skill/fix-vulnerabilities
```

Or symlink from a local clone:

```bash
git clone git@github.com:lightspeed-hospitality/skills.git ~/skills
ln -s ~/skills/skills/fix-vulnerabilities ~/.config/opencode/skill/fix-vulnerabilities
```

Skills are automatically discovered by OpenCode and available via the `skill` tool.

## Available Skills

| Skill                                             | Description                                                               |
| ------------------------------------------------- | ------------------------------------------------------------------------- |
| [atlassian-cli-jira](skills/atlassian-cli-jira)   | Common workflows for Atlassian CLI (acli) Jira commands                   |
| [fix-vulnerabilities](skills/fix-vulnerabilities) | Detect, validate, and fix security vulnerabilities from Dependabot alerts |
| [github-cli](skills/github-cli)                   | Common workflows for GitHub CLI (gh) commands                             |

## Contributing

1. Create `skills/<skill-name>/SKILL.md` with frontmatter:

```markdown
---
name: my-skill
description: One-line description of what this skill does
---

Instructions for the agent...
```

2. Directory name must match the `name` in frontmatter
3. Lowercase alphanumeric with single hyphens, 1-64 characters
4. Open a PR

### What belongs here

Skills useful across multiple projects: CLI references, language best practices, workflow automations, framework guides.

### What doesn't belong here

Project-specific skills should live in that project's `.opencode/skills/` directory.
