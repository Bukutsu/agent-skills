# agent-skills

My personal skill archive for pi, agy, Gemini, and Codex. Every agent uses the same global install list.

Pi config lives in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config). This repo only contains skills.

## install

```bash
git clone https://github.com/Bukutsu/agent-skills
cd agent-skills

# install each source once
npx skills add anthropics/skills -g --skill docx --skill pdf --skill pptx --skill xlsx -y
npx skills add cursor/plugins -g --skill bro --skill how --skill typescript-best-practices --skill why -y
npx skills add mattpocock/skills -g --skill handoff --skill writing-for-agents -y
npx skills add mgranberry/mermaid-diagram-skill -g --skill mermaid-diagram -y
npx skills add Tencent/BrowserSkill -g --skill browser-skill -y
npx skills add vercel-labs/agent-skills -g --skill web-design-guidelines -y
npx skills add tinyfish-io/tinyfish-cookbook -g --skill use-tinyfish -y

# install local skills
cp -r skills/* ~/.agents/skills/
```

Check the install with `npx skills ls -g`. It should list 18 skills: 14 external and 4 local.

The local skills are `audit-loop`, `git-peek`, `imprint`, and `perf-loop`.

## maintain

Add or remove a skill, then update the install list above:

```bash
npx skills add anthropics/skills -g --skill docx -y
npx skills remove docx -g -y
git commit -am "update skills" && git push
```
