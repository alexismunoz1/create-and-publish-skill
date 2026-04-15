---
name: create-and-publish-skill
description: >
  Creates, validates, and publishes Claude Code skills to skills.sh following
  Anthropic's official skill-building guide. Guides users through structured
  skill creation with interview-driven design, writes effective SKILL.md files,
  validates structure and description quality, creates a GitHub repository, and
  pushes the skill files. Handles both first-time publishing and updates.
  Use when user mentions "publish skill", "publish to skills.sh", "release skill",
  "share skill", "push skill to GitHub", "make skill installable", "create skill",
  "create a new skill", "build a skill", "deploy skill", "validate my skill",
  "check my skill", or "turn this into a skill".
  Do NOT use for general git operations or non-skill publishing.
compatibility: Claude Code
license: MIT
metadata:
  author: amunozdev
  version: 1.1.0
---

# Create and Publish Skill

Automates creating, validating, and publishing Claude Code skills to [skills.sh](https://skills.sh). All skills created with this tool follow [Anthropic's official skill-building guide](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf).

## Installation

```bash
npx skills add amunozdev/create-and-publish-skill
```

## Skill Design Philosophy

All skills MUST follow Anthropic's official guidelines. Before creating or validating any skill, consult `references/official-skill-building-guide.md` for the full specification. Key principles:

- **Progressive Disclosure:** Frontmatter is always loaded (level 1), SKILL.md body loads on relevance (level 2), references/ load on demand (level 3). This minimizes token usage.
- **Composability:** Skills should work alongside others, never assume exclusivity.
- **Portability:** Skills work across Claude.ai, Claude Code, and API.
- **Description is critical:** It determines when Claude loads the skill. Must include WHAT + WHEN (trigger phrases) + key capabilities. Max 1024 chars, no XML tags.
- **Explain the why:** Instead of heavy-handed MUSTs, explain _why_ a rule exists. Claude follows reasoning better than blind commands. For example: "Use kebab-case because the skill loader normalizes folder names" is more robust than "MUST use kebab-case".
- **Make descriptions pushy:** Skills undertrigger far more often than they overtrigger. A description should actively "sell" the skill to Claude so it loads when relevant. If in doubt, add more trigger phrases.
- **Bundle repeated work:** If a skill runs the same shell commands repeatedly, put them in `scripts/` to save tokens and reduce error surface.
- **Keep instructions lean:** SKILL.md should stay under 5,000 words. Move detailed content to `references/`. Every extra word costs tokens on every invocation.
- **Generalize, don't overfit:** Write skills for the category of problem, not one specific instance. A skill that handles "any barrel-export migration" is more valuable than one hardcoded to a single repo.
- **No README.md inside skill folder:** All docs go in SKILL.md or references/. README is only for the GitHub repo (for human visitors).
- **Folder naming:** kebab-case only, must match the `name` field.

## Skill Writing Guide

Use this guide when creating or reviewing SKILL.md files.

### Writing the Description

The description is the most important field — it controls when Claude loads your skill. Structure it as:

**WHAT** (what the skill does) + **WHEN** (trigger phrases) + **Capabilities** (key features)

Rules for effective descriptions:
- **Be pushy:** The description should actively convince Claude to load the skill. Undertriggering is far worse than overtriggering — an unloaded skill provides zero value.
- **Include realistic trigger phrases:** Use phrases real users actually say, not formal technical terms. "fix barrel exports" and "bundle size too big" are better than "optimize module resolution patterns".
- **Add negative triggers:** Explicitly state when NOT to use the skill to prevent overtriggering. Example: `Do NOT use for general import/export questions.`
- **Aim for 150-300 characters minimum.** Descriptions under 100 chars almost always undertrigger.

### Writing Instructions

The body of SKILL.md tells Claude _how_ to execute the skill. Follow these principles:

- **Explain the why:** For every rule or step, include a brief reason. "Validate frontmatter first _because invalid YAML causes silent failures at install time_" is more resilient than "Validate frontmatter first."
- **Use imperative form:** Write "Run the validation script" not "You should consider running the validation script". Be direct.
- **Include concrete examples:** Show exact commands, expected outputs, and sample inputs. Abstract instructions produce inconsistent results.
- **Progressive disclosure in the body:** Put the most common workflow first. Move edge cases, troubleshooting, and advanced options to later sections or `references/`.
- **Add error handling for each step:** Tell Claude what to do when things fail, not just when they succeed.

### Recommended Body Structure

```markdown
# Skill Name
Brief one-liner explaining the skill's purpose.

## Workflow / Instructions
Step-by-step phases, ordered by execution sequence.

## Edge Cases
Table of scenario → behavior mappings.

## Usage Examples
2-3 concrete examples showing trigger → action → result.

## Reference Materials
Links to references/ files for deep knowledge.
```

## Workflow

When activated, follow these phases in order. If the user wants to **create** a new skill, start at Phase C. If publishing or validating an existing skill, start at Phase 0. Stop and report errors at any phase — do not continue past a failure.

### Phase C: Create a New Skill

Use this phase when the user wants to create a new skill from scratch, or when they say "turn this into a skill" about something discussed in conversation.

**Step 1: Capture Intent**

Extract or ask for these four pieces of information:
1. **What does the skill do?** (core purpose in one sentence)
2. **Who is it for?** (target users and their context)
3. **What triggers it?** (phrases a user would say to invoke it)
4. **What does success look like?** (expected output or behavior)

If the user said "turn this into a skill", extract these answers from the preceding conversation context rather than asking again.

**Step 2: Interview & Research**

Dig deeper with targeted questions:
- What edge cases or failure modes should it handle?
- What input formats does it expect? What output formats does it produce?
- Does it depend on external tools (MCP servers, CLI tools, APIs)?
- Are there things it should explicitly NOT do? (negative triggers)
- What quality criteria determine if the skill executed correctly?

Consult `references/official-skill-building-guide.md` for structural requirements and patterns.

**Step 3: Write SKILL.md**

Generate the complete SKILL.md following this structure:

1. **Frontmatter:** Include `name` (kebab-case), `description` (pushy, WHAT+WHEN+capabilities, 150-300+ chars), `compatibility`, `license`, and `metadata` (author, version).
2. **Body:** Follow the Recommended Body Structure from the Skill Writing Guide above. Use imperative instructions, explain the why for each rule, and include concrete examples.
3. **References:** If the skill needs detailed guides (>50 lines of specialized knowledge), create `references/` files and link to them from the body.

Create the skill directory at `.agents/skills/{skill-name}/` with:
```
{skill-name}/
├── SKILL.md
└── references/          (if needed)
    └── *.md
```

**Step 4: Validate the Draft**

Run Phase 1 (Validate SKILL.md) on the newly created skill. Fix any blocking errors or warnings before proceeding.

**Step 5: Suggest Testing**

Recommend the user test the skill before publishing:
- Try 3-5 trigger phrases that should activate it
- Try 2-3 phrases that should NOT activate it
- Run through one complete workflow to verify output quality
- If available, suggest using a skill-testing tool for systematic validation

After testing, the user can proceed to Phase 0 → Phase 3 to publish.

### Phase 0: Identify the Skill

1. If the user specified a skill name, use it. Otherwise, list directories in `.agents/skills/` and ask which one to publish.
2. Set `SKILL_NAME` to the chosen name.
3. Set `SKILL_SRC` to `/Users/mac/dev/skill-dev/.agents/skills/{SKILL_NAME}/`.
4. Confirm the directory exists. If not, stop with an error.

### Phase 1: Validate SKILL.md

Run all checks before reporting. Collect errors (blocking) and warnings (non-blocking) separately.

**Blocking checks (any failure stops publishing):**

1. `{SKILL_SRC}/SKILL.md` exists
2. File starts with `---` and has a closing `---` (valid frontmatter delimiters)
3. Frontmatter contains valid YAML between the delimiters
4. `name` field: present, max 64 characters, matches regex `^[a-z0-9][a-z0-9-]*[a-z0-9]$`, matches the directory name
5. `name` field: does not contain "claude" or "anthropic" (reserved names per Anthropic's security restrictions)
6. `description` field: present, max 1024 characters
7. `description` field: does not contain XML angle brackets (`<` or `>`) — these cause parsing failures
8. Body content exists after the closing `---` (not just whitespace)
9. Any files referenced in the body (e.g., `references/something.md`) actually exist in `{SKILL_SRC}/references/`

**Non-blocking warnings (description quality):**

1. SKILL.md exceeds 500 lines — suggest extracting detailed content to `references/`
2. `metadata.version` field not present or not bumped since last publish — remind user to consider versioning
3. Description missing WHAT component — no clear statement of what the skill does. Suggest adding a sentence starting with a verb (e.g., "Analyzes...", "Creates...", "Manages...")
4. Description missing WHEN component — no trigger phrases found. Suggest adding: `Use when user mentions "phrase1", "phrase2", ...`
5. Description too short (under 100 characters) — likely to undertrigger. Recommend expanding to at least 150 chars with WHAT+WHEN+capabilities
6. Description not pushy enough — if it lacks action verbs and specific trigger phrases, warn that the skill may not load when relevant. Suggest adding explicit user phrases and a `Do NOT use for...` clause
7. Triggers too generic — if trigger phrases are single common words (e.g., "help", "create", "fix") without qualification, warn about overtriggering. Suggest more specific phrases

If there are blocking errors, list them all and stop. If there are only warnings, show them with specific fix suggestions and ask whether to continue.

### Phase 2: Prepare GitHub Repository

1. **Check `gh` CLI:** Run `gh auth status`. If it fails, stop with instructions:
   ```
   Install: brew install gh
   Authenticate: gh auth login
   ```

2. **Check if repo exists:** Run `gh repo view amunozdev/{SKILL_NAME} --json name` (suppress stderr).
   - If it exists, note "updating existing repo".
   - If it doesn't exist, create it:
     ```bash
     gh repo create amunozdev/{SKILL_NAME} --public --description "{description from frontmatter}"
     ```

3. **Set publish directory:** `PUBLISH_DIR=/Users/mac/dev/skill-dev/{SKILL_NAME}/`

4. **Check for uncommitted changes:** If `PUBLISH_DIR` exists and is a git repo, run `git -C {PUBLISH_DIR} status --porcelain`. If there are uncommitted changes, warn the user and ask whether to continue.

### Phase 3: Publish

**First-time publish** (PUBLISH_DIR does not exist):

1. Create directory structure:
   ```
   {SKILL_NAME}/
   ├── SKILL.md                              (copy from source)
   ├── references/                           (copy from source if exists)
   ├── README.md                             (auto-generate)
   └── .agents/skills/{SKILL_NAME}/
       ├── SKILL.md                          (copy from source)
       └── references/                       (copy from source if exists)
   ```

2. Auto-generate `README.md` with this template:
   ```markdown
   # {SKILL_NAME}

   {description from frontmatter}

   ## Installation

```bash
npx skills add amunozdev/{SKILL_NAME}
```

   ## What is this?

   A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that can be installed via [skills.sh](https://skills.sh).

   ## License

   MIT
   ```

3. Git operations:
   ```bash
   cd {PUBLISH_DIR}
   git init
   git add -A
   git commit -m "Initial publish of {SKILL_NAME} skill"
   git branch -M main
   git remote add origin git@github.com:amunozdev/{SKILL_NAME}.git
   git push -u origin main
   ```

**Update publish** (PUBLISH_DIR already exists):

1. Copy files from `SKILL_SRC` to both locations:
   - `{PUBLISH_DIR}/SKILL.md`
   - `{PUBLISH_DIR}/references/` (if exists in source)
   - `{PUBLISH_DIR}/.agents/skills/{SKILL_NAME}/SKILL.md`
   - `{PUBLISH_DIR}/.agents/skills/{SKILL_NAME}/references/` (if exists in source)

2. Check for changes: `git -C {PUBLISH_DIR} diff --stat`
   - If no changes, inform user "No changes to publish" and stop.

3. Show the diff to the user for review: `git -C {PUBLISH_DIR} diff`

4. Stage and commit:
   ```bash
   cd {PUBLISH_DIR}
   git add -A
   git commit -m "Update {SKILL_NAME} skill to v{version}"
   git push
   ```

### Phase 4: Verify & Report

1. Confirm the push succeeded (check git push exit code).

2. Display a summary:
   ```
   Published {SKILL_NAME} successfully!

   Repository: https://github.com/amunozdev/{SKILL_NAME}
   Install:    npx skills add amunozdev/{SKILL_NAME}
   ```

3. **Create symlinks** if they don't already exist:
   ```bash
   # .agent/skills/ symlink
   mkdir -p /Users/mac/dev/skill-dev/.agent/skills
   ln -sf ../../.agents/skills/{SKILL_NAME} /Users/mac/dev/skill-dev/.agent/skills/{SKILL_NAME}

   # .claude/skills/ symlink
   mkdir -p /Users/mac/dev/skill-dev/.claude/skills
   ln -sf ../../.agents/skills/{SKILL_NAME} /Users/mac/dev/skill-dev/.claude/skills/{SKILL_NAME}
   ```

## Edge Cases

| Scenario | Behavior |
|----------|----------|
| No changes to publish | Inform user, do not create empty commit |
| `gh` not installed or not authenticated | Provide installation/auth instructions and stop |
| Skill name has invalid characters | Fail with the regex pattern and a fix suggestion |
| Uncommitted changes in publish dir | Warn user and ask before proceeding |
| Referenced file missing | Blocking error listing which files are missing |
| SKILL.md > 500 lines | Warning suggesting extraction to `references/` |
| "Turn this into a skill" | Extract intent from conversation context, skip redundant interview questions, proceed to Phase C Step 3 |
| Description missing WHEN triggers | Warning with concrete suggestion: `Use when user mentions "phrase1", "phrase2"...` |
| Name contains "claude" or "anthropic" | Blocking error: these are reserved names per Anthropic's security restrictions |

## Usage Examples

### "Create a new skill"

1. Starts Phase C: captures intent (what, who, triggers, success criteria)
2. Interviews for edge cases, formats, dependencies
3. Writes SKILL.md with pushy description, imperative instructions, concrete examples
4. Validates the draft against Phase 1 checks
5. Suggests testing before publishing

### "Validate my skill"

1. Identifies skill at `.agents/skills/{skill-name}/`
2. Runs Phase 1 with all blocking checks and quality warnings
3. Reports issues with specific fix suggestions for each

### "Publish my token-optimizer skill"

1. Identifies skill at `.agents/skills/token-optimizer/`
2. Validates SKILL.md structure, frontmatter, and description quality
3. Checks/creates `amunozdev/token-optimizer` repo
4. Copies files to publish directory with correct structure
5. Pushes to GitHub
6. Reports install command

### "Update my published skill"

1. Detects existing publish directory and git repo
2. Copies updated files from source
3. Shows diff of changes
4. Commits and pushes the update

## Reference Materials

- `references/official-skill-building-guide.md` — Anthropic's complete guide to building skills for Claude, including frontmatter specification, design patterns, testing methodology, distribution best practices, and troubleshooting
