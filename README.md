# agent-skills

Global source of truth for agent skills — sync across `pi`, `agy` (Antigravity), `gemini-cli`, `codex`, `cursor` etc. via Vercel `skills` CLI.

`pi` settings stay in [`Bukutsu/pi-agent-config`](https://github.com/Bukutsu/pi-agent-config) (pi-sync). This repo is **only for skills**.

## Contents
- `vercel/global.skill-lock.json` — Vercel `skills` global lock (`~/.agents/.skill-lock.json`)
- `skills-lock.json` — project lock (when used at repo root, optional)

## Restore / update

```bash
git clone https://github.com/Bukutsu/agent-skills ~/Projects/agent-skills
# restore global skills
cp vercel/global.skill-lock.json ~/.agents/.skill-lock.json
npx skills update -g -y

# or add directly (writes to ~/.agents/.skill-lock.json)
npx skills add anthropics/skills -g -a pi,antigravity,gemini-cli --skill docx
npx skills add vercel-labs/agent-skills -g --all
npx skills list -g --json

# project (per-repo)
npx skills add <repo> --skill <name>   # writes ./skills-lock.json
git add skills-lock.json && git push
# teammate:
npx skills experimental_install
```
