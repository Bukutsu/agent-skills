# agent-skills

Single source of truth for agent skills — one lockfile, every harness.

Syncs to `pi` + `agy` (Antigravity) + `gemini-cli` + `codex` + `cursor` + 70 more via [Vercel `skills` CLI](https://skills.sh) (`npm:skills`). Pi config stays in [`Bukutsu/pi-agent-config`](https://github.com/Bukutsu/pi-agent-config) (this repo is **only for skills**).

## How it works

`npx skills add` writes the skill to every agent you target and records `source` + `sourceUrl` + `skillFolderHash` in a lockfile. `npx skills update` pulls latest from that source. Commit the lockfile to GitHub — `git clone` + one command restores everywhere.

```
vercel/global.skill-lock.json  →  ~/.agents/.skill-lock.json  →  ~/.pi/agent/skills/  +  ~/.agents/skills/  +  ~/.gemini/.../skills/
```

## How to use

### 1. Install / add
```bash
# global (all your projects) — to pi + agy
npx skills add vercel-labs/agent-skills -g --all
npx skills add anthropics/skills -g -a pi,antigravity --skill docx
npx skills add anthropics/skills -g -a pi,antigravity --skill pdf
npx skills add DietrichGebert/ponytail -g -a pi,antigravity --all

# project (this repo / team) — writes ./skills-lock.json
npx skills add vercel-labs/agent-skills --skill vercel-optimize -a pi
```

### 2. List / search
```bash
npx skills list -g --json          # global
npx skills list --json             # project (./skills-lock.json)
npx skills find typescript
npx skills find --owner vercel
npx skills add vercel-labs/agent-skills --list  # preview without installing
```

### 3. Update (keeps where we got it)
```bash
npx skills update -g -y            # update all global
npx skills update vercel-optimize -y
npx skills update -p -y            # project only
```

### 4. Remove
```bash
npx skills remove vercel-optimize -g --yes
npx skills remove --all -g --yes   # wipe all global
```

### 5. Sync this repo
```bash
# after add/update/remove, push the lockfile
cp ~/.agents/.skill-lock.json vercel/global.skill-lock.json
git add vercel/global.skill-lock.json
git commit -m "skills: add docx" && git push
```

### 6. Restore on a new machine
```bash
git clone https://github.com/Bukutsu/agent-skills ~/Projects/agent-skills

# global
cp ~/Projects/agent-skills/vercel/global.skill-lock.json ~/.agents/.skill-lock.json
npx skills update -g -y   # re-creates symlinks to ~/.pi/agent/skills etc.

# project (inside a cloned repo with skills-lock.json)
npx skills experimental_install
```

## Contents
- `vercel/global.skill-lock.json` — global lock (`~/.agents/.skill-lock.json`)
- `skills-lock.json` — project lock (created when you `add` without `-g` at repo root)

## Supported agents
`pi`, `antigravity`, `antigravity-cli`, `gemini-cli`, `codex`, `cursor`, `claude-code`, `opencode`, `zed`, +65 more — see `npx skills add --help` (`-a`).

Discover more at [skills.sh](https://skills.sh).
