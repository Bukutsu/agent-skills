# agent-skills

One lockfile for every agent. pi, agy, gemini, codex all use the same skills.

pi config is in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config). This repo is only skills.

## how it works

`npx skills add` puts the skill in each agent you choose and saves where it came from in the lockfile. `npx skills update` pulls the latest. Push the lockfile to GitHub and a fresh clone is one command.

```
vercel/global.skill-lock.json -> ~/.agents/.skill-lock.json -> ~/.pi/agent/skills, ~/.agents/skills
```

## use

Add a skill to pi and agy:

```bash
npx skills add anthropics/skills -g -a pi -a antigravity --skill docx -y
npx skills add vercel-labs/agent-skills -g --all -y
```

List and update:

```bash
npx skills list -g
npx skills update -g -y
```

Remove:

```bash
npx skills remove docx -g --yes
```

Save and push:

```bash
cp ~/.agents/.skill-lock.json vercel/global.skill-lock.json
git add vercel/global.skill-lock.json && git commit -m "add docx" && git push
```

Restore on a new machine:

```bash
git clone https://github.com/Bukutsu/agent-skills ~/Projects/agent-skills
cp ~/Projects/agent-skills/vercel/global.skill-lock.json ~/.agents/.skill-lock.json
npx skills update -g -y
```

Project skills (per repo) use `./skills-lock.json` instead:

```bash
npx skills add vercel-labs/agent-skills --skill vercel-optimize -a pi -y
npx skills experimental_install
```

More agents and skills at [skills.sh](https://skills.sh). See `npx skills add --help` for the full list.
