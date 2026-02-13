---
name: create-skill
description: Create a new skill in this repository following naming, structure, and documentation standards
---

# Create Skill

Add a new skill to this repository. Enforces naming conventions, frontmatter requirements, and keeps the README in sync.

## Required Information

Gather the following before proceeding. Ask for anything not provided.

### 1. Skill Name

- Lowercase alphanumeric with single hyphens, 1-64 characters
- Pattern: `^[a-z0-9]([a-z0-9-]{0,62}[a-z0-9])?$` (no leading/trailing/consecutive hyphens)
- Action-oriented when possible: `fix-vulnerabilities`, `setup-database`
- Don't prefix with `skill-` (redundant)

**If missing:** Ask "What should the skill be named? (kebab-case, e.g., add-api-endpoint)"

### 2. Description

- One line, under 100 characters
- Used in SKILL.md frontmatter and the README table

**If missing:** Ask "Provide a brief description of what this skill does"

### 3. Skill Type

- **procedural** — Step-by-step guide for completing a task (has numbered steps, checklist)
- **reference** — Documentation/patterns to follow (has sections, examples, best practices)

**If missing:** Ask "Is this a procedural skill (step-by-step) or reference skill (patterns/docs)?"

## Steps

### Step 1: Validate the name

Verify the name matches the naming convention:

```bash
echo "{name}" | grep -qE '^[a-z0-9]([a-z0-9-]{0,62}[a-z0-9])?$' && echo "valid" || echo "invalid"
```

Check it doesn't conflict with an existing skill:

```bash
ls skills/
```

### Step 2: Create the skill directory and SKILL.md

```bash
mkdir -p skills/{name}
```

Create `skills/{name}/SKILL.md` using the appropriate template below.

#### Procedural Template

```markdown
---
name: { name }
description: { description }
---

# {Title}

{What this skill does and when to use it.}

## Prerequisites

- {Required tool, access, or setup}

## Steps

### Step 1: {First Step}

{Instruction}

### Step 2: {Second Step}

{Instruction}

## Checklist

- [ ] {Verification item}
- [ ] {Verification item}
```

#### Reference Template

```markdown
---
name: { name }
description: { description }
---

# {Title}

{What patterns/conventions this documents.}

## Overview

{High-level explanation}

## Patterns

### {Pattern Name}

{Description with code example}

## Best Practices

| Do              | Don't          |
| --------------- | -------------- |
| {Good practice} | {Anti-pattern} |
```

### Step 3: Enforce CLI prerequisite rules

If the skill depends on CLI tools (e.g., `gh`, `acli`, `docker`), it **must** include:

1. A **Prerequisites** section listing required tools
2. **Install instructions** or a link for each tool — don't assume the user has it installed

Example:

```markdown
## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`) — install with `brew install gh` or see [installation docs](https://github.com/cli/cli#installation)
- Authenticated: `gh auth status`
```

### Step 4: Verify frontmatter

The SKILL.md must have YAML frontmatter with exactly these required fields:

```yaml
---
name: { name } # Must match directory name exactly
description: { desc } # Under 100 characters
---
```

Verify directory name matches frontmatter `name`:

```bash
head -4 skills/{name}/SKILL.md
# name: field must equal the directory name
```

### Step 5: Update README.md

Add the new skill to the **Available Skills** table in `README.md`. The table must remain sorted alphabetically by skill name.

| Skill                   | Description   |
| ----------------------- | ------------- |
| [{name}](skills/{name}) | {description} |

### Step 6: Final verification

Run through this checklist before committing:

## Checklist

- [ ] Directory name matches `name` in frontmatter
- [ ] Name follows convention: lowercase alphanumeric, single hyphens, 1-64 chars
- [ ] Frontmatter has `name` and `description`
- [ ] Description is under 100 characters
- [ ] CLI dependencies have install instructions in Prerequisites
- [ ] README.md Available Skills table is updated and alphabetically sorted
- [ ] SKILL.md has concrete examples (not just abstract instructions)
- [ ] Procedural skills end with a checklist section

## Writing Principles

| Do                                     | Don't                                 |
| -------------------------------------- | ------------------------------------- |
| Use imperative voice ("Add", "Create") | Use passive voice ("should be added") |
| Show concrete code examples            | Explain concepts abstractly           |
| Include copy-pasteable snippets        | Add placeholder-only examples         |
| Keep steps atomic and ordered          | Combine multiple actions in one step  |
| Be concise                             | Write verbose explanations            |
| Ask for missing required info          | Guess or assume values                |
