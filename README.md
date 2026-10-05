# agent-skills

My personal skill archive for pi, agy, Gemini, and Codex. Every agent uses the same global install list.

Pi config lives in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config). This repo only contains skills.

## install

```bash
git clone https://github.com/Bukutsu/agent-skills
cd agent-skills

# install each source once
npx skills add addyosmani/agent-skills -g --skill frontend-ui-engineering -y
npx skills add anthropics/skills -g --skill docx --skill frontend-design --skill pdf --skill pptx --skill xlsx -y
npx skills add cursor/plugins -g --skill bro --skill how --skill typescript-best-practices --skill why -y
npx skills add mattpocock/skills -g --skill handoff --skill writing-for-agents -y
npx skills add mgranberry/mermaid-diagram-skill -g --skill mermaid-diagram -y
npx skills add vercel-labs/agent-browser -g --skill agent-browser -y
npx skills add vercel-labs/agent-skills -g --skill web-design-guidelines -y
npx skills add tinyfish-io/tinyfish-cookbook -g --skill use-tinyfish -y

# install local skills
mkdir -p ~/.agents/skills
for skill in audit-loop git-peek imprint perf-loop; do
  cp -r "skills/$skill" ~/.agents/skills/
done
```

Check the install with `npx skills ls -g`. The list above installs 20 skills: 16 external and 4 local.

The active local skills are `audit-loop`, `git-peek`, `imprint`, and `perf-loop`.
`docs-first` remains archived here but is not installed; its compact policy lives in global `AGENTS.md` in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config).

Browser automation also needs the CLI and Chrome:

```bash
npm install -g agent-browser
agent-browser install
```

Agent-browser runs headlessly by default. Use a separate persistent profile for agent logins.

## maintain

Add or remove a skill, then update the install list above:

```bash
npx skills add anthropics/skills -g --skill docx -y
npx skills remove docx -g -y
git commit -am "update skills" && git push
```
