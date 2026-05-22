---
name: markdown-branch-push
description: Execute a full implementation workflow from one or more normal filesystem folders containing Markdown files. Use when the user provides folder paths with .md files and wants Codex to treat each Markdown file as one implementation unit, create one branch per file, implement, review, commit, push, and merge each branch into main.
---

# Markdown Branch Push

## Workflow

Treat this as an execution command, not a planning-only request. Keep moving until every ordered Markdown file in every requested folder has been implemented, reviewed, committed, branch-pushed, merged into main, and main-pushed, unless a blocker requires user input.

1. Resolve every folder path from the user request as a normal filesystem path.
2. Run preflight once in the target repository:
   - Inspect `git status --short --branch`, `git remote -v`, `git branch --show-current`, and `git fetch origin --prune`.
   - Verify an `origin` remote exists and either `origin/main` exists or local `main` can be safely pushed as the initial remote `main`. If histories are unrelated, merge with `--allow-unrelated-histories` only after confirming the remote contents are compatible with the local repository.
   - Inspect the available `/review` interface and `codex review --help` before review. Prefer `/review` when available. Use `codex review --uncommitted` only when that CLI path is known to be healthy in the current environment.
   - If using CLI review, use config value `model_reasoning_effort="xhigh"` for Extra High. Never use `extra_high`, and never append a prompt argument to `codex review --uncommitted`; apply design-focused review criteria when reading the result.
3. List Markdown files in each folder using `rg --files` when available, otherwise `find`.
   - Include `*.md` and `*.markdown`.
   - Ignore directories such as `.git`, `node_modules`, `dist`, `build`, `.venv`, and `venv`.
   - Read file contents directly from disk.
   - Detect image references in each Markdown file, including `![alt](path)`, HTML `<img src="...">`, reference-style image links, and Obsidian-style embeds such as `![[image.png]]`.
   - Resolve local image paths relative to the Markdown file first, then the requested folder. Inspect referenced local images with the available image viewing/perception tool before implementation.
   - Treat images as part of the requirements. If a required image is missing, unreadable, or remote-only and cannot be inspected, stop and ask for user input instead of guessing.
4. When multiple folders are provided, start one independent development lane per folder and run those lanes in parallel when the active environment and repository state make that safe.
   - Treat the user's multiple-folder request as permission to use parallel folder lanes.
   - Use one `git worktree` or isolated Codex CLI workspace per folder lane. Never run parallel lanes in the same dirty working tree.
   - If safe isolation is unavailable, process lanes sequentially and report that parallelism was skipped.
5. Within each folder lane, sort files by explicit ordering in the filename or title: numeric prefixes such as `1.`, `01`, `001`, then date prefixes, then lexical order as a fallback.
6. Treat each Markdown file as exactly one implementation unit. Do not split a file into smaller planned units unless the user explicitly asks.
7. For each file, create one working branch before editing:
   - Fetch remote state first.
   - Base the branch on the latest `origin/main`. If `origin/main` is missing but local `main` is the intended initial remote history, push local `main` to `origin/main` first, fetch, and then base the file branch on `origin/main`.
   - Use a short branch name derived from the folder and file order/title while preserving the repo's branch naming style when obvious.
   - Ensure the lane worktree is clean before editing that branch.
8. For each file branch:
   - Use GPT-5.5 Extra High for the initial implementation when model switching is available. If not available, use the strongest available model and report the fallback.
   - Read the Markdown file and any referenced images, infer the requested implementation from those requirements, inspect the existing code first, and make minimal, scoped edits.
   - Run `/review` before tests as the primary quality gate when available. If non-interactive CLI review is healthy and appropriate, run `codex review --uncommitted -c model="gpt-5.5" -c model_reasoning_effort="xhigh"` with no prompt argument.
   - Review must cover correctness, edge cases, public API compatibility, maintainability, cohesion, naming, abstraction boundaries, unnecessary coupling, and whether the code design is easy to evolve.
   - If `codex review --uncommitted` is known to hang or is currently blocked, do not run it; use `/review` or stop and report review as blocked.
   - Use GPT-5.5 Extra High for actionable review fixes.
   - Address all actionable review findings or explicitly justify non-actionable findings.
   - Run `/review` again after meaningful fixes and repeat until there are no actionable findings.
   - Skip broad validation by default. Run only a narrow, cheap check when review feedback or the touched code makes it clearly necessary.
   - Commit only the files for that Markdown file with a specific commit message.
   - Push the file branch to the configured remote.
9. After each file branch is pushed, merge it into `main` and push `main`.
   - Serialize all `main` integration and push steps across parallel folder lanes. Branch development may run in parallel; updating `main` is a critical section.
   - Use a clean integration worktree checked out from `origin/main` when possible.
   - Before integrating, run `git fetch origin --prune`. Stop if the local `main` branch has unpushed commits that were not created by this workflow; do not publish unrelated local `main` commits.
   - In the integration worktree, ensure `main` starts exactly at `origin/main` before merging the file branch.
   - Merge the file branch into `main` with the repository's normal merge style.
   - If conflicts occur, resolve them in favor of the final intended behavior, rerun focused review/checks for the conflict area when useful, complete the merge commit, and retry the `main` push.
   - If pushing `main` is rejected because the remote advanced, fetch the new `main`, merge `origin/main` into local `main`, resolve any conflicts, rerun focused review/checks when useful, and retry until `main` is pushed or a true blocker requires user input.
10. Continue with the next ordered file in that folder lane.

## Guardrails

- Never combine unrelated Markdown files into one branch or commit just to move faster.
- Never push a branch or `main` with unresolved review findings unless the final response names and justifies them.
- Never claim parallel execution, review, branch push, or `main` push happened unless the command/session actually completed successfully.
- Do not rewrite or revert user changes outside the current file's implementation scope.
- If the working tree is dirty before starting a file branch, inspect it and preserve user work. Ask only if those changes block branch creation or commit isolation.
- If a Markdown file is ambiguous, infer conservatively from the codebase and note the assumption in the commit or final summary.
- If safe parallelism is not available, process folder lanes sequentially and say why in the final response.
- Stop and ask for user input if authentication, branch protection, missing folders, empty Markdown folders, missing/unreadable required images, unavailable review command, or irreconcilable merge conflicts prevent a safe push.

## Final Response

Report each folder/file, branch name, commits created, branch push status, `main` merge/push status, reviews run, model/reasoning fallbacks, optional focused checks, conflict resolutions, whether lanes ran in parallel or sequentially, and any remaining risks.
