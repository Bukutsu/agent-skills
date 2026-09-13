# agent-skills

My personal archive of small agent skills for pi, agy, Gemini, and Codex.

`skills-lock.json` records the third-party and local skills in this repo.

## Fresh install

```bash
git clone https://github.com/Bukutsu/agent-skills
cd agent-skills
npx skills experimental_install
```

This restores all skills into the project `.agents/skills/` directory. That directory is ignored by git.

## Add a third-party skill

```bash
npx skills add owner/repo --skill skill-name -y
```

Commit the changed `skills-lock.json`.

## Add a local skill

Create `skills/my-skill/SKILL.md`, then add it to the lock file:

```bash
npx skills add ./skills --skill my-skill -y
```

## Update third-party skills

```bash
npx skills update -p
```

Commit the changed `skills-lock.json`.
