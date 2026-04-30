---
description: Stage all changes, commit with a sensible message, and push to GitHub master
argument-hint: [optional commit message]
---

Push everything in the working tree to the GitHub `master` branch (`origin`).

If the user passed a commit message after the command, it is here: `$ARGUMENTS`

## Steps

1. **Ensure git is reachable in this PowerShell session.** Git is installed but is not on the default PATH on this machine, so prepend it once at the start:

   ```powershell
   $env:Path = "C:\Program Files\Git\cmd;$env:Path"
   ```

2. **Detect changes.** Run `git status --short`. If the output is empty, reply with `Working tree clean — nothing to push.` and stop. Don't make an empty commit.

3. **Inspect what will be committed.** Stage everything with `git add -A`, then run `git diff --cached --stat` and (briefly) `git diff --cached` so you understand the scope of the change. Don't dump the full diff to the user — just use it to write a good message.

4. **Decide the commit message.**
   - If `$ARGUMENTS` is non-empty, use it verbatim as the commit subject. Don't add a body.
   - Otherwise, write your own short imperative subject line based on the staged diff (e.g. `Fix nav spacing on tablet`, `Tighten hero copy`, `Add /push slash command`). One line, present tense, no trailing period. Add a body only if the change is genuinely large enough to need one — most pushes don't.

5. **Commit.** Use `git commit -m "<subject>"`. If the subject contains characters PowerShell would mangle (backticks, `$`, double quotes), use a single-quoted here-string per the project conventions:

   ```powershell
   git commit -m @'
   Subject line here
   '@
   ```

   The closing `'@` must be at column 0.

6. **Push.** `git push origin master`.

   PowerShell will print git's progress lines as red `NativeCommandError` text — that is cosmetic, not a real failure. Check the actual command outcome (look for `[new branch]`, `Everything up-to-date`, or a real error like `rejected`).

7. **Verify and report.**
   - Run `git log -1 --oneline` and `git rev-parse origin/master` to confirm local HEAD matches remote.
   - Report to the user: the commit SHA, the subject, and that it's live on https://github.com/lynxsolutionstrading/virality.

## What not to do

- Don't force-push. If `git push` is rejected (e.g. remote is ahead), report the error verbatim and stop — let the user decide whether to pull/rebase.
- Don't skip hooks (`--no-verify`) or skip GPG signing.
- Don't `git add` selectively unless the user explicitly asked. Default is `-A` (everything tracked + new files, respecting `.gitignore`).
- Don't poll, sleep, or loop. Each git command runs once and you act on the result.
