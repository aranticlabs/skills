<div align="center">
    <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/aranticlabs/arantic-docs/main/static/img/brand/arantic-logo-dark.svg" />
        <img src="https://raw.githubusercontent.com/aranticlabs/arantic-docs/main/static/img/brand/arantic-logo-light.svg" width="400" alt="Agentic Engineering" />
    </picture><br /><br />
    <p>A library of agents and skills from Arantic Labs.<br />
    Reusable, ready-to-install agents and skills for planning, specs, reviews, and AI-assisted development workflows.</p>
</div>

# Agentic Engineering

A curated collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) **agents** and **skills** built and maintained by Arantic Labs. Agents are specialized subagents Claude Code can delegate work to; skills are self-contained workflows Claude Code can load to extend its capabilities with domain-specific instructions.

## What's inside

- **`Agents/`** — Specialized subagent definitions (e.g. `system-architect`, `security-auditor`, `debug-detective`, `test-engineer`, `ux-ui-designer`). Each agent is a single Markdown file with frontmatter describing when Claude Code should delegate to it.
- **`Skills/`** — Self-contained skill directories (e.g. `write-prd`, `dead-code-scanner`, `Typescript`). Each skill ships a `SKILL.md` that Claude Code loads on demand.

More agents and skills will be added over time.

---

## Installing

Both agents and skills can be installed at the **user level** (available in every project) or at the **project level** (scoped to a single repo).

### Skills

User-level (global):

```bash
git clone https://github.com/aranticlabs/agentic-engineering.git
cp -r agentic-engineering/Skills/<skill-name> ~/.claude/skills/
```

Project-level:

```bash
mkdir -p .claude/skills
cp -r /path/to/agentic-engineering/Skills/<skill-name> .claude/skills/
```

### Agents

User-level (global):

```bash
mkdir -p ~/.claude/agents
cp agentic-engineering/Agents/<agent-name>.md ~/.claude/agents/
```

Project-level:

```bash
mkdir -p .claude/agents
cp /path/to/agentic-engineering/Agents/<agent-name>.md .claude/agents/
```

### Verify

Inside Claude Code, type `/` to see installed skills, or ask Claude to list available agents.

---

## Using

### Skills

Invoke a skill by name in a Claude Code session:

```
/<skill-name>
```

Or describe the task in natural language — Claude Code will trigger the relevant skill based on its `description` frontmatter.

### Agents

Agents are invoked by Claude Code automatically when a task matches the agent's description, or you can ask Claude to delegate explicitly (e.g. "use the security-auditor agent to review this").

---

## Format

### Skill

Each skill is a directory containing a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: skill-name
description: When to invoke this skill and what it does.
model: opus
---

# Skill Name

Instructions for Claude to follow when this skill is invoked...
```

### Agent

Each agent is a single Markdown file with YAML frontmatter:

```markdown
---
name: agent-name
description: When Claude Code should delegate to this agent.
tools: [...]
---

# Agent Name

System prompt and instructions for the agent...
```

The `description` field on both agents and skills is what Claude uses to decide when to invoke them, so it should clearly state the trigger conditions.

For a full guide to authoring agents and skills, see [Arantic Docs](https://github.com/aranticlabs/arantic-docs).

---

Built and maintained by [Arantic Digital](https://arantic.com).
