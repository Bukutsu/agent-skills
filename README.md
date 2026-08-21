# agent-skills

One lockfile for every agent. pi, agy, gemini, codex all use the same skills.

pi config is in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config). This repo is only skills.

## use

```bash
git clone https://github.com/Bukutsu/agent-skills
cp agent-skills/vercel/global.skill-lock.json ~/.agents/.skill-lock.json
npx skills update -g -y
cp -r agent-skills/skills/* ~/.agents/skills/
```

That's it. `update` puts the skills in `~/.pi/agent/skills`, `~/.agents/skills`, etc.

`skills/` holds my own skills (`git-peek`, `audit-loop`) as plain files. They're not in the lockfile, so the `cp -r` above is what installs them.

To add or remove a skill (maintainer):

```bash
npx skills add anthropics/skills -g --skill docx -y
npx skills remove docx -g --yes
cp ~/.agents/.skill-lock.json vercel/global.skill-lock.json
git commit -am "update skills" && git push
```
