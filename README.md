# agent-skills

One install list for every agent. pi, agy, gemini, codex all use the same skills.

pi config is in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config). This repo is only skills.

## use

```bash
git clone https://github.com/Bukutsu/agent-skills
cd agent-skills

# one clone per source
npx skills add anthropics/skills -g --skill docx --skill pdf --skill pptx --skill xlsx -y
npx skills add blader/humanizer -g --skill humanizer -y
npx skills add cursor/plugins -g --skill how --skill typescript-best-practices --skill unslop --skill why -y
npx skills add exa-labs/agent-skills -g --skill build-with-exa --skill exa-contents --skill exa-search -y
npx skills add iamzhihuix/happy-claude-skills -g --skill video-processor -y
npx skills add mattpocock/skills -g --skill code-review --skill codebase-design --skill diagnosing-bugs --skill domain-modeling --skill grilling --skill handoff --skill prototype --skill research --skill resolving-merge-conflicts --skill tdd --skill wait-what --skill wizard --skill writing-for-agents -y
npx skills add mgranberry/mermaid-diagram-skill -g --skill mermaid-diagram -y
npx skills add vercel-labs/agent-browser -g --skill agent-browser -y
npx skills add vercel-labs/agent-skills -g --skill web-design-guidelines -y
npx skills add tinyfish-io/tinyfish-cookbook -g --skill use-tinyfish -y

# local skills (not in any registry)
cp -r skills/* ~/.agents/skills/
```

Verify: `npx skills ls -g` → 32 skills (30 from above + 2 local).

`skills/` holds my own skills (`git-peek`, `audit-loop`) as plain files.

To add or remove a skill (maintainer):

```bash
npx skills add anthropics/skills -g --skill docx -y
npx skills remove docx -g -y
# then update the list above in README.md
git commit -am "update skills" && git push
```
