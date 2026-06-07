# quiet-build skills

Reusable [Agent Skills](https://agentskills.io) for AI coding agents (Claude Code, and any agent that supports the open SKILL.md format), maintained by **quiet-build**.

Each skill is a self-contained `SKILL.md` that teaches an agent a proven, repeatable workflow. They install in seconds and work across projects.

## Install

Install all skills (or pick the ones you want) with the [`skills`](https://skills.sh) installer — no clone required:

```bash
npx skills@latest add quiet-build/skills
```

You'll be prompted to choose which skills and which agents to install into. The installer drops each skill into your agent's skills directory (e.g. `~/.claude/skills/` for Claude Code), where it's auto-discovered.

### Manual install

Prefer to copy by hand? Each skill is just a folder:

```bash
git clone https://github.com/quiet-build/skills.git
cp -r skills/skills/engineering/rebase-and-verify ~/.claude/skills/
```

## Available skills

| Skill | Category | What it does |
|-------|----------|--------------|
| [`rebase-and-verify`](skills/engineering/rebase-and-verify/SKILL.md) | engineering | Rebase a branch onto a moving target, resolve conflicts **by intent**, run every quality gate (lint, types, unit, e2e), and get an independent review before declaring it done. |

## Repository layout

```
.claude-plugin/
  plugin.json                 # marketplace manifest — lists every skill path
skills/
  <category>/<skill-name>/
    SKILL.md                  # the skill (YAML frontmatter + body)
README.md
LICENSE
```

## Adding a skill

1. Create `skills/<category>/<skill-name>/SKILL.md` with frontmatter:

   ```yaml
   ---
   name: your-skill-name
   description: Use when <specific triggering conditions and symptoms>.
   ---
   ```

   The `description` should state **when to use** the skill (triggers and symptoms), not summarize its steps — that's what an agent matches against to decide whether to load it.

2. Register the folder in [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json) under `skills`.
3. Add a row to the table above.
4. Open a PR.

## License

[MIT](LICENSE) © quiet-build
