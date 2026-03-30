# xAID Shared Skills

Shared Claude Code skills for the xAID team.

## Install

In Claude Code, run:

```
/plugins install https://github.com/theVmagnificient/xaid-shared-skills.git
```

## Update

```
/plugins update xaid-shared-skills
```

## Available Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| *(add skills here as they're created)* | | |

## Adding a New Skill

1. Create a directory under `skills/<skill-name>/`
2. Add a `SKILL.md` with the frontmatter and prompt
3. Commit and push — team members run `/plugins update xaid-shared-skills`

### SKILL.md template

```markdown
---
name: skill-name
description: Use this skill when the user asks to "...", "...", or mentions "...". One sentence that tells Claude when to trigger this skill.
version: 1.0.0
---

# Skill Title

Detailed instructions for Claude...
```
