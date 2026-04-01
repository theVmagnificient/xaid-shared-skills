# xaid-shared-skills — Agent Instructions

## Plugin Versioning

The plugin version is in `.claude-plugin/plugin.json`. Use semver:

- **PATCH** (1.0.0 → 1.0.1): bug fixes in skills, wording tweaks, typo corrections
- **MINOR** (1.0.0 → 1.1.0): new skills added, new capabilities in existing skills
- **MAJOR** (1.0.0 → 2.0.0): breaking changes to skill interface, removed skills

Bump the version in the same commit that updates the skills.
