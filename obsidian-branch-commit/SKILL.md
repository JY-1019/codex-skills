---
name: obsidian-branch-commit
description: Execute the Markdown branch commit workflow from one or more Obsidian folder paths without pushing. Use when the user provides Obsidian folders or vault paths and wants Codex to retrieve Markdown requirements through the Obsidian CLI, treat each Markdown file as one implementation unit, create one branch per file, implement, review, and commit locally.
---

# Obsidian Branch Commit

## Workflow

Treat this as an execution command, not a planning-only request. Keep moving until every ordered Obsidian file in every requested folder has been implemented, reviewed, and committed locally. Do not push.

1. Resolve every Obsidian folder path from the user request.
2. Run preflight once in the target repository:
   - Inspect `git status --short --branch`, `git remote -v`, and `git branch --show-current`.
   - If an `origin` remote exists, run `git fetch origin --prune`.
   - Verify `origin/main` or `main` exists.
   - Inspect `codex review --help` or the available `/review` interface so the review command can actually be invoked with the strongest available model and design-focused instructions.
3. Use the installed Obsidian CLI to list Markdown files in each folder and read requirements from them.
   - Inspect `obsidian --help` first and use the CLI's native list/read commands when available.
   - Include `*.md` and `*.markdown`.
   - Ignore directories such as `.git`, `node_modules`, `dist`, `build`, `.venv`, and `venv`.
   - Read file contents through the Obsidian CLI when available. If the CLI cannot read file contents, use it to verify/resolve the vault or folder and then read files from disk as a fallback; report that fallback.
4. When multiple folders are provided, start one independent development lane per folder and run those lanes in parallel when the active environment and repository state make that safe.
   - Treat the user's multiple-folder request as permission to use parallel folder lanes.
   - Use one `git worktree` or isolated Codex CLI workspace per folder lane. Never run parallel lanes in the same dirty working tree.
   - If safe isolation is unavailable, process lanes sequentially and report that parallelism was skipped.
5. Within each folder lane, sort files by explicit ordering in the filename or title: numeric prefixes such as `1.`, `01`, `001`, then date prefixes, then lexical order as a fallback.
6. Treat each Obsidian-sourced Markdown file as exactly one implementation unit. Do not split a file into smaller planned units unless the user explicitly asks.
7. For each file, create one working branch before editing:
   - Fetch remote state first when an `origin` remote exists.
   - Base the branch on the latest `origin/main` when available, otherwise on local `main`.
   - Use a short branch name derived from the folder and file order/title while preserving the repo's branch naming style when obvious.
   - Ensure the lane worktree is clean before editing that branch.
8. For each file branch:
   - Use GPT-5.5 Extra High for the initial implementation when model switching is available. If not available, use the strongest available model and report the fallback.
   - Read the Obsidian-sourced Markdown file, infer the requested implementation from its contents, inspect the existing code first, and make minimal, scoped edits.
   - Run `/review` or `codex review --uncommitted` before tests as the primary quality gate. If the CLI supports model/reasoning flags, use GPT-5.5 Extra High. If the CLI supports a prompt or instructions field, include: "Review correctness, edge cases, public API compatibility, maintainability, cohesion, naming, abstraction boundaries, unnecessary coupling, and whether the code design is easy to evolve."
   - Use GPT-5.5 Extra High for actionable review fixes.
   - Address all actionable review findings or explicitly justify non-actionable findings.
   - Run `/review` again after meaningful fixes and repeat until there are no actionable findings.
   - Skip broad validation by default. Run only a narrow, cheap check when review feedback or the touched code makes it clearly necessary.
   - Commit only the files for that Obsidian-sourced Markdown file with a specific commit message.
9. Leave each file branch local and continue with the next ordered file in that folder lane.

## Guardrails

- Do not push.
- Never combine unrelated Obsidian-sourced Markdown files into one branch or commit just to move faster.
- Do not commit with unresolved review findings unless the final response names and justifies them.
- Never claim parallel execution, review, or commit happened unless the command/session actually completed successfully.
- Do not rewrite or revert user changes outside the current file's implementation scope.
- If the working tree is dirty before starting a file branch, inspect it and preserve user work. Ask only if those changes block branch creation or commit isolation.
- If an Obsidian-sourced Markdown file is ambiguous, infer conservatively from the codebase and note the assumption in the commit or final summary.
- If safe parallelism is not available, process folder lanes sequentially and say why in the final response.
- Stop and ask for user input if authentication for an existing remote, missing Obsidian folders, empty Obsidian Markdown folders, unavailable Obsidian CLI, unavailable review command, or irreconcilable branch conflicts prevent safe local commits.

## Final Response

Report each folder/file, branch name, commits created, reviews run, model/reasoning fallbacks, optional focused checks, whether lanes ran in parallel or sequentially, and any remaining risks. Explicitly state that no push or `main` integration was performed.
