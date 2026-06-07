# Ship: Branch, Commit, Push, and Open a PR

Derive a meaningful branch name, check out to it, stage and commit everything possible, push, and open a pull request — all in one step.

## Steps

1. **Check status** — run `git status` and `git diff` to understand what's changed. If there's nothing to commit and no commits ahead of the base branch, stop and tell the user.

2. **Create and checkout a branch** — derive a short, kebab-case branch name from the diff summary (e.g. `feat/add-user-auth`, `fix/null-pointer-login`, `refactor/split-payment-service`). Rules:
   - Prefix with `feat/`, `fix/`, `refactor/`, `chore/`, or `docs/` based on the nature of the changes.
   - Keep it under 50 characters, lowercase, no spaces.
   - If `$ARGUMENTS` were provided, use them as the primary signal for the name.
   - If already on a non-main/non-master branch with relevant changes, skip this step and stay on the current branch.
   - Run: `git checkout -b <branch-name>`

3. **Stage changes** — run `git add` for every modified/untracked file that isn't a secret. Skip `.env`, `*.pem`, `*credentials*`, `*.key`, and similar files — warn the user about any skipped files.

4. **Commit as much as possible** — split the staged changes into logical, atomic commits if the diff covers distinct concerns (e.g. separate commits for dependency updates, config changes, and feature code). For each commit:
   - Write a concise, imperative subject line (≤72 chars) that explains *why*, not just what.
   - No co-author lines.
   - Use the format:
     ```
     git commit -m "$(cat <<'EOF'
     <message>
     EOF
     )"
     ```
   - If all changes are tightly coupled, a single commit is fine.

5. **Push** — run `git push -u origin <current-branch>`.

6. **Open a PR** — use `gh pr create` targeting the default base branch (`main` or `master`) with:
   - A short title (≤70 chars) derived from the branch name / commit messages.
   - A body with a summary (2–3 bullets) and a test plan checklist.

   ```
   gh pr create --title "<title>" --body "$(cat <<'EOF'
   ## Summary
   - ...

   ## Test plan
   - [ ] ...

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

7. **Return the PR URL** so the user can open it immediately.

## Arguments

If the user passes `$ARGUMENTS`, use them as the primary signal for both the branch name and PR title. Example: `/ship add dark mode` → branch `feat/add-dark-mode`, title "Add dark mode".

## Safety rules

- Never use `--no-verify` or force-push unless the user explicitly asks.
- Never commit `.env`, `*.pem`, `*credentials*`, or similar secret files — warn instead.
- If already on `main` or `master` with no good branch to derive, warn the user and suggest a branch name before proceeding.
- If a pre-commit hook fails, fix the underlying issue and re-commit; never bypass the hook.