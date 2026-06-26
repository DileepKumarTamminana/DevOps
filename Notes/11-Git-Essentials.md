# 11 — Git Essentials

Git is the foundation under everything else — CI/CD triggers on it, branching strategies
are built on it ([[03-Infrastructure-and-Config]]), and pipelines check it out
([[01-CICD-and-Automation]]). This file covers it from basics.

---

## 1. What Git is (and isn't)

**Git** = a *distributed version control system*. It records snapshots of your files over
time so you can track history, work in parallel, and undo mistakes.

- **Git** = the tool (runs locally on your machine).
- **GitHub / GitLab / Azure Repos** = *hosting* services for Git repositories (remote
  copies + collaboration features like PRs and Actions).

"Distributed" means every clone is a **full copy** of the history — you can commit, branch,
and view history offline.

---

## 2. The three areas (the mental model)

```
 Working Directory  →  Staging Area (Index)  →  Local Repo  →  Remote (GitHub)
   (your files)         git add                 git commit      git push
```
| Area | What it holds | Command to move forward |
|---|---|---|
| **Working directory** | Files you're editing | `git add` |
| **Staging area** | Changes marked for the next commit | `git commit` |
| **Local repository** | Committed history on your machine | `git push` |
| **Remote** | Shared copy on GitHub | (`git pull` to come back) |

> **Interview-ready:** "Git has three areas — working directory, staging, and the local
> repo. `add` stages, `commit` records to local history, `push` sends to the remote."

---

## 3. Everyday commands

### Starting out
```bash
git init                    # turn a folder into a repo
git clone <url>             # copy an existing remote repo
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### The core loop
```bash
git status                  # what's changed / staged
git add file.txt            # stage one file
git add .                   # stage everything
git commit -m "message"     # record staged changes
git commit -am "message"    # stage tracked files + commit in one go
git log --oneline           # compact history
git diff                    # unstaged changes
git diff --staged           # staged changes
```

### Working with the remote
```bash
git remote -v               # list remotes
git remote add origin <url> # link a remote named 'origin'
git push -u origin main     # push and set upstream (first time)
git push                    # subsequent pushes
git pull                    # fetch + merge remote changes
git fetch                   # download remote changes WITHOUT merging
```

> `pull` = `fetch` + `merge`. Use `fetch` first when you want to inspect before merging.

---

## 4. Branching & merging

Branches let you work on features in isolation without touching `main`.

```bash
git branch                      # list branches
git branch feature/login        # create a branch
git checkout feature/login      # switch to it
git checkout -b feature/login   # create AND switch (shortcut)
git switch -c feature/login     # modern equivalent of checkout -b

# ... make commits on the branch ...

git checkout main               # go back to main
git merge feature/login         # bring the branch's work into main
git branch -d feature/login     # delete the merged branch
```

### Fast-forward vs merge commit
- **Fast-forward:** if `main` hasn't moved, Git just moves the pointer forward — no extra
  commit.
- **Merge commit:** if both branches advanced, Git creates a new "merge commit" tying the
  histories together.

> See branching *strategies* (GitHub Flow, Git Flow, trunk-based) in
> [[03-Infrastructure-and-Config]] §4.

---

## 5. Merge vs Rebase (classic interview question)

Both integrate changes from one branch into another, but differently:

| | `git merge` | `git rebase` |
|---|---|---|
| History | Preserves it; adds a merge commit | Rewrites it; replays your commits onto a new base |
| Result | Non-linear (shows the branch) | Linear, clean history |
| Safety | Safe on shared branches | **Never rebase commits already pushed/shared** |

```bash
# Merge: combine feature into main
git checkout main
git merge feature/login

# Rebase: replay feature's commits on top of the latest main
git checkout feature/login
git rebase main
```

> **Golden rule:** rebase only **local, unpushed** commits. Rewriting shared history breaks
> everyone else's clones.

> **Interview-ready:** "Merge preserves history with a merge commit; rebase rewrites it for
> a linear history. I rebase local work to clean it up but never rebase commits that are
> already pushed/shared."

---

## 6. Pull Requests (PRs) & code review

A **Pull Request** is a GitHub feature (not raw Git): you propose merging a branch into
`main`, and teammates **review** before it's merged.

Typical flow:
1. `git checkout -b feature/x` → make commits → `git push -u origin feature/x`
2. On GitHub: **Compare & pull request** → describe the change.
3. CI runs automatically (GitHub Actions); reviewers comment/approve.
4. Merge (often with **branch protection** requiring approvals + green checks).

This is how "code review gates" and "deployment approval workflows" on your resume are
enforced — via **branch protection rules** + required status checks.

---

## 7. Undoing things (the safety net)

```bash
git restore file.txt              # discard unstaged changes to a file
git restore --staged file.txt     # unstage (keep the edit)
git commit --amend                # fix the LAST commit (message or content)
git revert <commit>               # make a NEW commit that undoes a commit (safe, shared)
git reset --soft HEAD~1           # undo last commit, KEEP changes staged
git reset --hard HEAD~1           # undo last commit, DISCARD changes (dangerous!)
git stash                         # shelve changes temporarily
git stash pop                     # bring them back
```

| Goal | Use | Safe on shared branches? |
|---|---|---|
| Undo a pushed commit | `git revert` | ✅ Yes (adds a new commit) |
| Throw away local commits | `git reset --hard` | ❌ No (rewrites history) |
| Temporarily set work aside | `git stash` | ✅ Yes (local only) |

> **Interview-ready:** "To undo something already pushed, I use `git revert` — it adds a new
> commit and doesn't rewrite shared history. `reset --hard` is for local-only cleanup."

---

## 8. .gitignore

A `.gitignore` file lists patterns Git should **not** track — build output, dependencies,
secrets, local config:
```gitignore
node_modules/
*.log
.env                 # never commit secrets!
dist/
.DS_Store
```
> Critical for security: keep credentials and `.env` files out of source control entirely.

---

## 9. Handling merge conflicts

A conflict happens when two branches change the same lines. Git marks them:
```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> feature/login
```
To resolve: edit the file to the correct final content, remove the `<<<`/`===`/`>>>`
markers, then:
```bash
git add <file>
git commit          # completes the merge
```

---

## 10. Common terms quick reference
| Term | Meaning |
|---|---|
| **Repository (repo)** | A project tracked by Git |
| **Commit** | A snapshot of changes with a message + unique hash (SHA) |
| **Branch** | A movable pointer to a line of commits |
| **HEAD** | Pointer to your current commit/branch |
| **Origin** | Default name for the remote you cloned from |
| **Upstream** | The remote branch your local branch tracks |
| **Tag** | A fixed label on a commit (often a release version, `v1.2.0`) |
| **Fork** | Your own server-side copy of someone else's repo |
| **Clone** | A local copy of a repo |

---

## Practice checklist
- [ ] Init a repo, make 3 commits, view `git log --oneline`.
- [ ] Create a feature branch, commit, merge it back to `main`.
- [ ] Rebase a feature branch onto an updated `main`.
- [ ] Cause a merge conflict on purpose and resolve it.
- [ ] Use `git revert` to undo a pushed commit (vs `reset --hard` locally).
- [ ] Add a `.gitignore` that excludes `node_modules/` and `.env`.
- [ ] Open a Pull Request on GitHub and merge it after CI passes.
