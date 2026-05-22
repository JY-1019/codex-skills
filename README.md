# codex-skills

Personal Codex skills.

## Included Skills

- `obsidian-branch-push`: Retrieve requirements from Obsidian Markdown files, then run the Markdown branch push workflow.
- `obsidian-branch-commit`: Retrieve requirements from Obsidian Markdown files, then run the Markdown branch commit workflow without pushing.

## Local Codex Install Location

Codex loads local skills from:

```bash
$CODEX_HOME/skills
```

If `CODEX_HOME` is not set, the usual local path is:

```bash
~/.codex/skills
```

Each skill must be stored as its own directory directly under that `skills` directory, with `SKILL.md` at the top level:

```text
~/.codex/skills/
  obsidian-branch-push/
    SKILL.md
    agents/openai.yaml
  obsidian-branch-commit/
    SKILL.md
    agents/openai.yaml
```

## Install Or Update

Clone this repository, then copy or sync the skill folders into Codex's local skills directory:

```bash
git clone git@github.com:JY-1019/codex-skills.git
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
rsync -a codex-skills/obsidian-branch-push/ "${CODEX_HOME:-$HOME/.codex}/skills/obsidian-branch-push/"
rsync -a codex-skills/obsidian-branch-commit/ "${CODEX_HOME:-$HOME/.codex}/skills/obsidian-branch-commit/"
```

You can also keep them linked instead of copied:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
ln -sfn "$PWD/codex-skills/obsidian-branch-push" "${CODEX_HOME:-$HOME/.codex}/skills/obsidian-branch-push"
ln -sfn "$PWD/codex-skills/obsidian-branch-commit" "${CODEX_HOME:-$HOME/.codex}/skills/obsidian-branch-commit"
```

Restart Codex after installing or updating skills so the skill list is refreshed.
