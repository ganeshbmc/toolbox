# Git Best Practices for Multi‑Machine & Shared‑Branch Workflows

This guide captures safe, repeatable Git habits when you work on the same repository across multiple machines (laptop, desktop, WSL, Codespaces, agents) and when you have long‑running shared branches.


---

1. Mental Model (Critical)

Git tracks three different states:

Working Tree  ↔  Local Branch  ↔  Remote‑tracking Branch
(files)          (agy)            (origin/agy)

git status compares local branch ↔ local remote‑tracking branch

git fetch updates Git’s knowledge of the remote

git pull = git fetch + merge/rebase


👉 Never trust git status alone until you have fetched.


---

2. Golden Rule (Always)

> Before starting work on any machine: fetch → pull → then edit.




---

3. Starting Work on a Machine That Already Has the Repo

Recommended command:

git fetch --all --prune && git pull

This ensures:

Remote changes are discovered

Deleted remote branches are cleaned up

Your current branch is safely updated


Run this before opening your editor.


---

4. Branch Types and How to Pull

A. Shared, Long‑Running Branches

Examples:

main

develop

agy (in your setup)


Rule: ❌ Do NOT rebase these branches

Use:

git pull

Why:

Rebasing rewrites history

Shared branches must have stable history

Other machines/people/bots rely on it


Treat shared branches as history‑sacred.


---

B. Personal / Feature Branches

Examples:

ganesh/feature-x

experiment/nlp


Rule: ✅ Rebase is preferred

Use:

git pull --rebase

Why:

Clean, linear history

Easier reviews

No noisy merge commits



---

5. Recommended Workflow with a Shared Branch (agy)

Step 1: Sync shared branch

git switch agy
git fetch --all --prune
git pull

Step 2: Create a personal feature branch

git switch -c ganesh/feature-x

Step 3: Work and commit normally

Step 4: Keep feature branch updated

git fetch --all --prune
git rebase origin/agy

Step 5: Merge back into agy

Prefer PRs

Or merge locally (no rebase)



---

6. Global Git Configuration (Safe Setup)

A. Always clean up deleted remote branches

git config --global fetch.prune true

B. If you want rebase by default (optional)

Only do this if you explicitly override shared branches.

git config --global pull.rebase true

Then protect shared branches:

git config branch.main.rebase false
git config branch.agy.rebase false

Result:

git pull on shared branches → merge

git pull on personal branches → rebase



---

7. If You Already Have Local Changes

Option A: Commit first (best)

git add .
git commit -m "WIP"
git pull

Option B: Stash

git stash
git pull
git stash pop


---

8. One‑Line Habit to Memorize

git fetch --all --prune && git pull

Run it every time you sit down at a new machine or resume work.


---

9. Absolute Don’ts

❌ Rebase main or agy ❌ Edit files before fetching ❌ Trust git status without git fetch ❌ Rewrite history on shared branches


---

10. TL;DR

Shared branches → merge pulls only

Personal branches → rebase pulls

Multiple machines → always fetch first

When in doubt → do not rebase



---

This workflow scales cleanly to:

multiple laptops

Codespaces / WSL

CI bots

agentic coding workflows
