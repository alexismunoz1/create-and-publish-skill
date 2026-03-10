---
name: publish-skill
description: >
  Publishes Claude Code skills to skills.sh by validating SKILL.md structure,
  creating a GitHub repository, and pushing the skill files. Handles both
  first-time publishing and updates to existing skills.
  Use when user mentions "publish skill", "publish to skills.sh", "release skill",
  "share skill", "push skill to GitHub", "make skill installable", or
  "deploy skill". Do NOT use for general git operations or non-skill publishing.
compatibility: Claude Code
license: MIT
metadata:
  author: alexismunoz1
  version: 1.0.0
---

# Publish Skill

Automates publishing Claude Code skills to [skills.sh](https://skills.sh). Validates skill structure, creates a GitHub repository, copies files to the publish directory, and pushes — handling both first-time publishing and updates.

## Installation

```bash
npx skills add alexismunoz1/publish-skill
```

## Workflow

When activated, follow these four phases in order. Stop and report errors at any phase — do not continue past a failure.

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
5. `description` field: present, max 1024 characters
6. Body content exists after the closing `---` (not just whitespace)
7. Any files referenced in the body (e.g., `references/something.md`) actually exist in `{SKILL_SRC}/references/`

**Non-blocking warnings:**

1. SKILL.md exceeds 500 lines — suggest extracting detailed content to `references/`
2. `metadata.version` field not present or not bumped since last publish — remind user to consider versioning

If there are blocking errors, list them all and stop. If there are only warnings, show them and ask whether to continue.

### Phase 2: Prepare GitHub Repository

1. **Check `gh` CLI:** Run `gh auth status`. If it fails, stop with instructions:
   ```
   Install: brew install gh
   Authenticate: gh auth login
   ```

2. **Check if repo exists:** Run `gh repo view alexismunoz1/{SKILL_NAME} --json name` (suppress stderr).
   - If it exists, note "updating existing repo".
   - If it doesn't exist, create it:
     ```bash
     gh repo create alexismunoz1/{SKILL_NAME} --public --description "{description from frontmatter}"
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
   npx skills add alexismunoz1/{SKILL_NAME}
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
   git remote add origin git@github.com:alexismunoz1/{SKILL_NAME}.git
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
   ✅ Published {SKILL_NAME} successfully!

   Repository: https://github.com/alexismunoz1/{SKILL_NAME}
   Install:    npx skills add alexismunoz1/{SKILL_NAME}
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

## Usage Examples

### "Publish my token-optimizer skill"

1. Identifies skill at `.agents/skills/token-optimizer/`
2. Validates SKILL.md structure and frontmatter
3. Checks/creates `alexismunoz1/token-optimizer` repo
4. Copies files to publish directory with correct structure
5. Pushes to GitHub
6. Reports install command

### "Update my published skill"

1. Detects existing publish directory and git repo
2. Copies updated files from source
3. Shows diff of changes
4. Commits and pushes the update
