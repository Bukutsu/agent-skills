# agent-skills

My personal skill archive for pi, agy, Gemini, Codex, and Claude Code. Every agent uses the same global install list.

Pi config lives in [pi-agent-config](https://github.com/Bukutsu/pi-agent-config). This repo only contains skills.

## install

```bash
git clone https://github.com/Bukutsu/agent-skills
cd agent-skills

# install each source once
npx skills add anthropics/skills -g --skill docx --skill frontend-design --skill pdf --skill pptx --skill xlsx -y
npx skills add cursor/plugins/pstack -g --skill bro --skill typescript-best-practices -y
npx skills add mattpocock/skills -g --skill writing-for-agents -y
npx skills add vercel-labs/agent-browser -g --skill agent-browser -y
npx skills add vercel-labs/skills -g --skill find-skills -y
npx skills add tinyfish-io/tinyfish-cookbook -g --skill use-tinyfish -y

# link local skills so repo edits stay in sync
mkdir -p ~/.agents/skills ~/.claude/skills
for skill in agent-ready audit-loop contract-research imprint perf-loop; do
  ln -s "$PWD/skills/$skill" "$HOME/.agents/skills/$skill"
  ln -s "$HOME/.agents/skills/$skill" "$HOME/.claude/skills/$skill"
done
```

Check the install with `npx skills ls -g`. The list above installs 16 skills: 11 external and 5 local.

The local skills:

- `agent-ready` starts the repo as a fresh agent would, then fixes what blocks it.
- `audit-loop` runs read-only code audits in rounds until a round finds nothing new.
- `contract-research` checks unfamiliar external behavior against matching-version source or authoritative documentation.
- `imprint` writes prose the way you would. It builds a private voice profile from your own past agent sessions and saves it in your agent's memory or user state, outside this repo, so each person who clones it gets their own.
- `perf-loop` profiles and optimizes speed or memory in a loop.

`audit-loop`'s reviewer brief adapts [Heym's adversarial review prompt](https://heym.run/blog/ai-code-review), [Anthropic's code-reviewer agent](https://github.com/anthropics/claude-code/blob/main/plugins/pr-review-toolkit/agents/code-reviewer.md), Google's [code review guide](https://google.github.io/eng-practices/review/reviewer/looking-for.html), and [Linus Torvalds on good taste](https://www.ted.com/talks/linus_torvalds_the_mind_behind_linux).

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
