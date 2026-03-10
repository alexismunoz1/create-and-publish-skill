# The Complete Guide to Building Skills for Claude

> Source: [Anthropic — The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf)

## Introduction

A skill is a set of instructions — packaged as a simple folder — that teaches Claude how to handle specific tasks or workflows. Skills are one of the most powerful ways to customize Claude for your specific needs. Instead of re-explaining your preferences, processes, and domain expertise in every conversation, skills let you teach Claude once and benefit every time.

Skills are powerful when you have repeatable workflows: generating frontend designs from specs, conducting research with consistent methodology, creating documents that follow your team's style guide, or orchestrating multi-step processes. They work well with Claude's built-in capabilities like code execution and document creation. For those building MCP integrations, skills add another powerful layer helping turn raw tool access into reliable, optimized workflows.

## Chapter 1: Fundamentals

### What is a skill?

A skill is a folder containing:

- **SKILL.md** (required): Instructions in Markdown with YAML frontmatter
- **scripts/** (optional): Executable code (Python, Bash, etc.)
- **references/** (optional): Documentation loaded as needed
- **assets/** (optional): Templates, fonts, icons used in output

### Core Design Principles

**Progressive Disclosure** — Skills use a three-level system:

1. **First level (YAML frontmatter):** Always loaded in Claude's system prompt. Provides just enough information for Claude to know when each skill should be used without loading all of it into context.
2. **Second level (SKILL.md body):** Loaded when Claude thinks the skill is relevant to the current task. Contains the full instructions and guidance.
3. **Third level (Linked files):** Additional files bundled within the skill directory that Claude can choose to navigate and discover only as needed.

This progressive disclosure minimizes token usage while maintaining specialized expertise.

**Composability** — Claude can load multiple skills simultaneously. Your skill should work well alongside others, not assume it's the only capability available.

**Portability** — Skills work identically across Claude.ai, Claude Code, and API. Create a skill once and it works across all surfaces without modification, provided the environment supports any dependencies the skill requires.

### Skills + MCP (Connectors)

- **MCP** provides the professional kitchen: access to tools, ingredients, and equipment.
- **Skills** provide the recipes: step-by-step instructions on how to create something valuable.

| | MCP (Connectivity) | Skills (Knowledge) |
|---|---|---|
| Role | Connects Claude to your service | Teaches Claude how to use your service effectively |
| Provides | Real-time data access and tool invocation | Workflows and best practices |
| Defines | What Claude can do | How Claude should do it |

## Chapter 2: Planning and Design

### Start with Use Cases

Before writing any code, identify 2-3 concrete use cases your skill should enable.

**Category 1: Document & Asset Creation** — Creating consistent, high-quality output including documents, presentations, apps, designs, code, etc.

**Category 2: Workflow Automation** — Multi-step processes that benefit from consistent methodology, including coordination across multiple MCP servers.

**Category 3: MCP Enhancement** — Workflow guidance to enhance the tool access an MCP server provides.

### Define Success Criteria

**Quantitative metrics:**
- Skill triggers on 90% of relevant queries
- Completes workflow in X tool calls
- 0 failed API calls per workflow

**Qualitative metrics:**
- Users don't need to prompt Claude about next steps
- Workflows complete without user correction
- Consistent results across sessions

### Technical Requirements

#### File Structure

```
your-skill-name/
├── SKILL.md          # Required - main skill file
├── scripts/          # Optional - executable code
│   ├── process_data.py
│   └── validate.sh
├── references/       # Optional - documentation
│   ├── api-guide.md
│   └── examples/
└── assets/           # Optional - templates, etc.
    └── report-template.md
```

#### YAML Frontmatter

Minimal required format:

```yaml
---
name: your-skill-name
description: What it does. Use when user asks to [specific phrases].
---
```

#### Field Requirements

**name** (required):
- kebab-case only
- No spaces or capitals
- Should match folder name

**description** (required):
- MUST include BOTH what the skill does AND when to use it (trigger conditions)
- Under 1024 characters
- No XML tags (`<` or `>`)
- Include specific tasks users might say
- Mention file types if relevant

**license** (optional): Use if making skill open source. Common: MIT, Apache-2.0

**compatibility** (optional): 1-500 characters. Indicates environment requirements.

**metadata** (optional): Any custom key-value pairs. Suggested: author, version, mcp-server.

#### Critical Rules

- SKILL.md naming: Must be exactly `SKILL.md` (case-sensitive)
- Skill folder naming: Use kebab-case (e.g., `notion-project-setup`)
- Don't include README.md inside your skill folder
- All documentation goes in SKILL.md or references/

#### Security Restrictions

Forbidden in frontmatter:
- XML angle brackets (`<` `>`)
- Skills with "claude" or "anthropic" in name (reserved)

### Writing Effective Skills

#### The Description Field

Structure: `[What it does]` + `[When to use it]` + `[Key capabilities]`

Good examples:

```yaml
# Good - specific and actionable
description: Analyzes Figma design files and generates developer handoff documentation. Use when user uploads .fig files, asks for "design specs", "component documentation", or "design-to-code handoff".

# Good - includes trigger phrases
description: Manages Linear project workflows including sprint planning, task creation, and status tracking. Use when user mentions "sprint", "Linear tasks", "project planning", or asks to "create tickets".

# Good - clear value proposition
description: End-to-end customer onboarding workflow for PayFlow. Handles account creation, payment setup, and subscription management. Use when user says "onboard new customer", "set up subscription", or "create PayFlow account".
```

Bad examples:

```yaml
# Too vague
description: Helps with projects.

# Missing triggers
description: Creates sophisticated multi-page documentation systems.

# Too technical, no user triggers
description: Implements the Project entity model with hierarchical relationships.
```

#### Writing the Main Instructions

Recommended structure after frontmatter:

```markdown
---
name: your-skill
description: [...]
---

# Your Skill Name

## Instructions

### Step 1: [First Major Step]
Clear explanation of what happens.

```bash
python scripts/fetch_data.py --project-id PROJECT_ID
Expected output: [describe what success looks like]
```

(Add more steps as needed)

## Examples

Example 1: [common scenario]
User says: "Set up a new marketing campaign"
Actions:
1. Fetch existing campaigns via MCP
2. Create new campaign with provided parameters
Result: Campaign created with confirmation link

## Troubleshooting

Error: [Common error message]
Cause: [Why it happens]
Solution: [How to fix]
```

#### Best Practices for Instructions

**Be Specific and Actionable:**

```markdown
# Good
Run `python scripts/validate.py --input {filename}` to check data format.
If validation fails, common issues include:
- Missing required fields (add them to the CSV)
- Invalid date formats (use YYYY-MM-DD)

# Bad
Validate the data before proceeding.
```

**Reference bundled resources clearly:**

```markdown
Before writing queries, consult `references/api-patterns.md` for:
- Rate limiting guidance
- Pagination patterns
- Error codes and handling
```

**Use progressive disclosure:** Keep SKILL.md focused on core instructions. Move detailed documentation to `references/` and link to it.

**Include error handling:**

```markdown
## Common Issues
### MCP Connection Failed
If you see "Connection refused":
1. Verify MCP server is running: Check Settings > Extensions
2. Confirm API key is valid
3. Try reconnecting: Settings > Extensions > [Your Service] > Reconnect
```

## Chapter 3: Testing and Iteration

### Recommended Testing Approach

**1. Triggering tests** — Ensure your skill loads at the right times:

```
Should trigger:
- "Help me set up a new ProjectHub workspace"
- "I need to create a project in ProjectHub"
- "Initialize a ProjectHub project for Q4 planning"

Should NOT trigger:
- "What's the weather in San Francisco?"
- "Help me write Python code"
- "Create a spreadsheet" (unless skill handles sheets)
```

**2. Functional tests** — Verify the skill produces correct outputs:
- Valid outputs generated
- API calls succeed
- Error handling works
- Edge cases covered

**3. Performance comparison** — Prove the skill improves results vs. baseline.

### Iteration Based on Feedback

**Undertriggering signals:**
- Skill doesn't load when it should
- Users manually enabling it
- Solution: Add more detail and nuance to the description

**Overtriggering signals:**
- Skill loads for irrelevant queries
- Users disabling it
- Solution: Add negative triggers, be more specific

**Execution issues:**
- Inconsistent results, API call failures
- Solution: Improve instructions, add error handling

## Chapter 4: Distribution and Sharing

### Recommended Approach

1. **Host on GitHub** — Public repo for open-source skills with clear README and example usage
2. **Document in your MCP repo** — Link to skills from MCP documentation
3. **Create an installation guide**

### Positioning Your Skill

Focus on outcomes, not features:

```
# Good
"The ProjectHub skill enables teams to set up complete project workspaces
in seconds — including pages, databases, and templates — instead of
spending 30 minutes on manual setup."

# Bad
"The ProjectHub skill is a folder containing YAML frontmatter and
Markdown instructions that calls our MCP server tools."
```

**Important:** When distributing via GitHub, you'll want a repo-level README for human visitors — this is separate from your skill folder, which should NOT contain a README.md.

## Chapter 5: Patterns and Troubleshooting

### Pattern 1: Sequential Workflow Orchestration

Use when users need multi-step processes in a specific order. Key techniques: explicit step ordering, dependencies between steps, validation at each stage, rollback instructions for failures.

### Pattern 2: Multi-MCP Coordination

Use when workflows span multiple services. Key techniques: clear phase separation, data passing between MCPs, validation before moving to next phase, centralized error handling.

### Pattern 3: Iterative Refinement

Use when output quality improves with iteration. Key techniques: explicit quality criteria, iterative improvement, validation scripts, know when to stop iterating.

### Pattern 4: Context-Aware Tool Selection

Use when same outcome requires different tools depending on context. Key techniques: clear decision criteria, fallback options, transparency about choices.

### Pattern 5: Domain-Specific Intelligence

Use when your skill adds specialized knowledge beyond tool access. Key techniques: domain expertise embedded in logic, compliance before action, comprehensive documentation, clear governance.

### Troubleshooting

| Problem | Cause | Solution |
|---------|-------|----------|
| Skill won't upload | File not named exactly SKILL.md | Rename to SKILL.md (case-sensitive) |
| Invalid frontmatter | YAML formatting issue | Check `---` delimiters, unclosed quotes |
| Invalid skill name | Name has spaces or capitals | Use kebab-case |
| Skill doesn't trigger | Description too vague or missing triggers | Add specific trigger phrases |
| Skill triggers too often | Description too broad | Add negative triggers, be more specific |
| Instructions not followed | Too verbose, buried, or ambiguous | Concise bullets, critical info at top |
| MCP connection issues | Server not connected or auth invalid | Verify connection and API keys |
| Large context issues | Skill content too large | Move detailed docs to references/ |

## Reference A: Quick Checklist

### Before You Start
- [ ] Identified 2-3 concrete use cases
- [ ] Tools identified (built-in or MCP)
- [ ] Reviewed this guide and example skills
- [ ] Planned folder structure

### During Development
- [ ] Folder named in kebab-case
- [ ] SKILL.md file exists (exact spelling)
- [ ] YAML frontmatter has `---` delimiters
- [ ] name field: kebab-case, no spaces, no capitals
- [ ] description includes WHAT and WHEN
- [ ] No XML tags (`<` `>`) anywhere
- [ ] Instructions are clear and actionable
- [ ] Error handling included
- [ ] Examples provided
- [ ] References clearly linked

### Before Upload
- [ ] Tested triggering on obvious tasks
- [ ] Tested triggering on paraphrased requests
- [ ] Verified doesn't trigger on unrelated topics
- [ ] Functional tests pass
- [ ] Tool integration works (if applicable)

### After Upload
- [ ] Test in real conversations
- [ ] Monitor for under/over-triggering
- [ ] Collect user feedback
- [ ] Iterate on description and instructions
- [ ] Update version in metadata

## Reference B: YAML Frontmatter

### Required Fields

```yaml
---
name: skill-name-in-kebab-case
description: What it does and when to use it. Include specific trigger phrases.
---
```

### All Optional Fields

```yaml
name: skill-name
description: [required description]
license: MIT
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch"
metadata:
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
```

### Security Notes

**Allowed:**
- Any standard YAML types (strings, numbers, booleans, lists, objects)
- Custom metadata fields
- Long descriptions (up to 1024 characters)

**Forbidden:**
- XML angle brackets (`<` `>`) — security restriction
- Code execution in YAML (uses safe YAML parsing)
- Skills named with "claude" or "anthropic" prefix (reserved)

## Reference C: Complete Skill Examples

- **Document Skills** — PDF, DOCX, PPTX, XLSX creation
- **Example Skills** — Various workflow patterns
- **Partner Skills Directory** — Skills from Asana, Atlassian, Canva, Figma, Sentry, Zapier, and more

Public skills repository: [github.com/anthropics/skills](https://github.com/anthropics/skills)
