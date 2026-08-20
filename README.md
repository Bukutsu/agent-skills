# agent-skills

Global source of truth for agent skills — sync across `pi`, `agy` (Antigravity), `gemini-cli`, `codex`, `cursor` etc. via Vercel `skills` CLI.

This repo stores **two truths** (migrating from `pi` packages to Vercel):

* `pi/` — `~/.pi/agent/settings.json` + `AGENTS.md` (pi's native `packages[]` + filtered `skills`)
* `vercel/` — `~/.agents/.skill-lock.json` (Vercel `skills` global lock, single source for all harnesses)

## Pi config (legacy, still active)
- `pi/settings.json` — 14 packages (anthropics/skills, exa-labs, ponytail, mattpocock, etc.)
- `pi/AGENTS.md` — global agent guidelines

Restore pi on new machine:
```bash
# pi-sync is disabled for skills, so copy manually or via dotfiles:
cp pi/settings.json ~/.pi/agent/settings.json
cp pi/AGENTS.md ~/.pi/agent/AGENTS.md
pi update --all
```

## Vercel skills (new source of truth for skills)
- `vercel/global.skill-lock.json` — global lock (`npx skills list -g`)

Restore / update skills on any machine:
```bash
# global (user-level, all projects)
cp vercel/global.skill-lock.json ~/.agents/.skill-lock.json
npx skills update -g -y
# or re-add directly:
npx skills add anthropics/skills -g -a pi,antigravity,gemini-cli --skill docx
npx skills add vercel-labs/agent-skills -g --all

# project (commit `skills-lock.json` at repo root)
npx skills add <repo> --skill <name>   # writes ./skills-lock.json
git add skills-lock.json && git push
# teammate:
npx skills experimental_install
```

## Current pi packages migrated
`pi/settings.json` packages will be gradually moved to `vercel` via `npx skills add -g -a pi,antigravity --all` so one `skills update -g` syncs everything.

See `pi/settings.json` for the full list to migrate.
