# agent-skills

One lockfile for every agent. pi, agy, gemini, codex all use the same skills.

pi config is in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config). This repo is only skills.

## use

```bash
git clone https://github.com/Bukutsu/agent-skills
cp agent-skills/vercel/global.skill-lock.json ~/.agents/.skill-lock.json
npx skills update -g -y
```

That's it. `update` puts the skills in `~/.pi/agent/skills`, `~/.agents/skills`, etc.

To add or remove a skill (maintainer):

```bash
npx skills add anthropics/skills -g --skill docx -y
npx skills remove docx -g --yes
cp ~/.agents/.skill-lock.json vercel/global.skill-lock.json
git commit -am "update skills" && git push
```
