# quiet-build skills

Reusable [Agent Skills](https://agentskills.io) for AI coding agents (Claude Code, and any agent that supports the open SKILL.md format), maintained by **quiet-build**.

Each skill is a self-contained `SKILL.md` that teaches an agent a proven, repeatable workflow. They install in seconds and work across projects.

> **Prerequisite:** [Node.js](https://nodejs.org) (for `npx`) if you use the CLI installer. The manual method needs only `git`.

## Install via CLI (recommended)

Install with the [`skills`](https://skills.sh) installer — no clone required. It drops each skill into your agent's skills directory (e.g. `~/.claude/skills/` for Claude Code), where it's auto-discovered.

```bash
# Interactive — pick which skills and which agents
npx skills@latest add quiet-build/skills

# Just one skill, into Claude Code, no prompts
npx skills@latest add quiet-build/skills --skill rebase-and-verify --agent claude-code -y

# Everything, into every detected agent
npx skills@latest add quiet-build/skills --all
```

Other handy commands:

```bash
npx skills@latest add quiet-build/skills --list   # preview skills without installing
npx skills@latest use quiet-build/skills@rebase-and-verify   # try a skill without installing it
npx skills@latest update                          # pull the latest version of installed skills
```

## Install manually

Each skill is a self-contained folder — copy it into your agent's skills directory:

```bash
git clone https://github.com/quiet-build/skills.git
cd skills
cp -r skills/engineering/rebase-and-verify ~/.claude/skills/
```

> Note the nested path: the repo is named `skills`, and skills live under its `skills/<category>/` directory — hence `skills/engineering/rebase-and-verify` after `cd skills`.

## Available skills

| Skill | Category | What it does |
|-------|----------|--------------|
| [`rebase-and-verify`](skills/engineering/rebase-and-verify/SKILL.md) | engineering | Rebase a branch onto a moving target, resolve conflicts **by intent**, run every quality gate (lint, types, unit, e2e), and get an independent review before declaring it done. Pass `--simple` (alias `--fast`) for a quick low-risk rebase that runs lint/eslint only and skips type checks, unit/e2e tests, and the review. |
| [`reproduce-then-fix`](skills/engineering/reproduce-then-fix/SKILL.md) | engineering | Fix a bug the trustworthy way: write a **failing test that reproduces it first**, confirm it fails for the reported reason, trace the **root cause**, make the smallest fix, run the **full suite after every change**, treat any new failure as a regression you caused, and commit only when the repro passes with zero regressions — with before/after test output. |
| [`app-store-preflight`](skills/engineering/app-store-preflight/SKILL.md) | engineering | Review Apple App Store metadata, branding, privacy, purchases, release behavior and reviewer access against current official rules. Supports early planning, final preflight and rejection follow-up; reports concrete fixes and missing evidence. |

## App Store preflight

```bash
npx skills@latest add quiet-build/skills --skill app-store-preflight
```

For a global Codex installation, add `--agent codex --global`; for Claude Code, use `--agent claude-code --global`. Without `--global`, installation is project-scoped.

Example request: “Use app-store-preflight to review this app and its listing before submission. Report risks and proposed fixes.” The skill needs current Apple documentation and access to the relevant app artifacts; unavailable evidence is reported as unverified. It does not guarantee approval or authorize submission.

## Repository layout

```
.claude-plugin/
  plugin.json                 # plugin manifest — lists every skill path
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

## Publishing and maintenance

The public GitHub repository is the distribution source; no npm package is needed. Follow the folder, manifest and catalog pattern above, validate the skill, and review the complete diff before merging a PR into `main`.

- Keep skills self-contained and portable. Bundle the license with a skill so folder-only installs retain it; exclude credentials, account identifiers and private review records.
- Verify installer discovery with `npx skills@latest add . --list` before publishing, and repeat against `quiet-build/skills` after merging.
- For workflow changes, try representative requests: an early draft, a metadata rejection and a final review with missing runtime evidence. Structural validation alone does not prove behavior.
- Use `npx skills@latest check` to check installed skills and `npx skills@latest update` to update them. Review upstream changes before adopting them for release-critical work.
- For reproducible manual installs, clone this repository, check out a reviewed commit or release tag, and copy the selected skill folder. Record that revision; a moving `main` branch and `@latest` installer are not version pins.
- Recheck Apple sources during each app review. Update the skill when an observed failure warrants it, rather than copying changing policy text into a permanent checklist.

Publishing guidance checked against the [Agent Skills specification](https://agentskills.io/specification) and [skills installer documentation](https://github.com/vercel-labs/skills) on 2026-09-16.

## License

[MIT](LICENSE) © quiet-build
