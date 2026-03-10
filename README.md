# create-and-publish-skill

Creates and publishes Claude Code skills to [skills.sh](https://skills.sh) following [Anthropic's official skill-building guide](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf). Validates SKILL.md structure, creates a GitHub repository, and pushes the skill files. Handles both first-time publishing and updates to existing skills.

## Installation

```bash
npx skills add alexismunoz1/create-and-publish-skill
```

## What is this?

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that can be installed via [skills.sh](https://skills.sh).

This skill automates the full lifecycle of Claude Code skills:

1. **Create** — Generates properly structured skills following Anthropic's official guidelines (frontmatter, progressive disclosure, trigger phrases, references)
2. **Validate** — Checks SKILL.md against the official specification (name format, description requirements, file references, size limits)
3. **Publish** — Creates a GitHub repo, copies files to the correct dual structure (root + nested `.agents/`), and pushes
4. **Update** — Detects changes, shows diffs, and pushes updates to existing published skills

## Built on Anthropic's Official Guide

All skills created and validated by this tool follow the principles from [The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf):

- **Progressive Disclosure** — Three-level system (frontmatter → SKILL.md body → references/) to minimize token usage
- **Description-Driven Activation** — Skills trigger automatically based on well-crafted description fields with specific trigger phrases
- **Composability** — Skills designed to work alongside others
- **Proper Structure** — kebab-case naming, no README inside skill folders, references for detailed docs

The full guide is included as a reference file within the skill at `references/official-skill-building-guide.md`.

## Usage

```
"Publish my token-optimizer skill"
"Create a new skill called code-reviewer"
"Update my published skill"
```

## License

MIT
