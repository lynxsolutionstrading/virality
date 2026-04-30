# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Virality is a single-page marketing site for an Instagram-growth consultancy. The entire site lives in `index.html` (inline `<style>`, no build step). It's served locally during development by `live-server` on port 8081.

Reference design: `virality-mobile.html` is the original mobile mockup the live page was rebuilt from. Brand assets are in `Logo/` (SVG/PNG/EPS — same logo, different formats). Logo is wired into the header, footer, favicon, and OG image as an inline `<symbol>` so letters can be tinted via `currentColor`.

## Workflow rules

### Auto-push after every change

After completing any user request that modified files in the working tree, push automatically as the last step of your response. Don't ask for confirmation. Same procedure as below — write your own commit subject from the diff, commit, push, and include the resulting SHA in the response.

Skip the auto-push only when:
- No files were modified (nothing to commit).
- The user explicitly said "don't push" / "noch nicht pushen" for this change.
- A push attempt fails (rejected, network error). Report the failure and stop — don't retry or force.

### "push" → push everything to GitHub

When the user writes **`push`** (alone, or as an obvious instruction like "push das" / "jetzt push"), treat it as a request to:

1. Prepend git to the PATH (it's installed at `C:\Program Files\Git\cmd\git.exe` but not on the default session PATH):
   ```powershell
   $env:Path = "C:\Program Files\Git\cmd;$env:Path"
   ```
2. `git status --short` — if clean, reply "Working tree clean — nothing to push." and stop.
3. `git add -A`, then look at `git diff --cached --stat` (and a quick `git diff --cached`) to understand the change.
4. Write a short imperative commit subject yourself (one line, present tense, no period). Only add a body if the change is genuinely large.
   - If the user provided their own message after the word "push" (e.g. `push fix nav spacing`), use it verbatim instead.
5. `git commit -m "<subject>"` — for messages with `$`, backticks, or quotes use a single-quoted PowerShell here-string (`@'...'@`, closing `'@` at column 0).
6. `git push origin master` — note: PowerShell paints git's progress lines red as `NativeCommandError`; that's cosmetic, not a real failure. Look for `[new branch]`, `Everything up-to-date`, or a real `rejected`.
7. Verify with `git log -1 --oneline` and confirm `origin/master` matches local HEAD.
8. Report the SHA, subject, and the GitHub URL: https://github.com/lynxsolutionstrading/virality

**Don't:**
- Don't make an empty commit.
- Don't force-push. If the push is rejected, report the error and stop — let the user decide.
- Don't skip hooks (`--no-verify`) or signing.

The same logic is also wired up as the `/push` slash command in `.claude/commands/push.md` (takes an optional message arg).

## Tooling on this machine

- **Node.js** (v24.15) is installed at `C:\Program Files\nodejs\` but not on the default PATH. Prepend it the same way as git when needed.
- **live-server** runs via `npx --yes live-server --port=8081` from the project root.
- **Git Credential Manager** is configured — `git push` works non-interactively without prompting for credentials.
- Default branch on the GitHub remote is **`master`** (not `main`).
