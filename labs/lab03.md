# Lab 3: Git

In the group project you will share a repository with teammates: feature branches, pull requests, merge conflicts, and catching up with `main`. This lab practices those operations.

You cannot push to the course copy of minGPT (`mlip-cmu/minGPT_copy`) — you do not have write permission. You will **fork** it (create a copy under your GitHub account), push to that copy, and open a pull request. On the group project you will usually be collaborators on one shared repo, so you skip the fork, but you still use branches and PRs.

To receive credit for this lab, show your work to the TA during recitation. AI is allowed; you must be able to explain every command you ran.

## Deliverables
- [ ] Resolve a merge conflict on two feature branches (do not commit to `main`). Show `git log --oneline --graph --decorate` and explain what happened.
- [ ] Open a pull request into [mlip-cmu/minGPT_copy](https://github.com/mlip-cmu/minGPT_copy). Show the PR URL. Explain clone vs fork, and why we use pull requests.
- [ ] Undo a bad commit with `git revert`. Explain when `revert` is preferred over `reset`.
- [ ] Rebase a published feature branch, show that a normal `git push` is rejected, then update it with `git push --force-with-lease`. Explain when rebase / force-push is OK.

Opening the PR is enough; it does not need to be merged.

## Getting started

1. On GitHub, **fork** [mlip-cmu/minGPT_copy](https://github.com/mlip-cmu/minGPT_copy) (Fork button). This is your copy, so you can push to it.
2. Clone **your fork** (replace the username) and add the course repo as `upstream`:

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/minGPT_copy
cd minGPT_copy
git remote add upstream https://github.com/mlip-cmu/minGPT_copy
git remote -v
```

`origin` should be your fork. `upstream` should be `mlip-cmu/minGPT_copy`.

Create/edit only `lab_notes.md` unless a later step names another file. Do not edit `mingpt/`.

## Exercise 1: Merge conflict

Use this when two branches changed the same lines and Git cannot combine them automatically. Do not commit to `main` — team projects keep it protected.

1. From `main`, create `feature-a` and add `lab_notes.md` with:

```
# Lab 3 notes
Author: feature-a
```

```bash
git switch -c feature-a
git add lab_notes.md
git commit -m "Add lab notes on feature-a"
```

2. Switch back to `main`, create `feature-b`, and add the same file with `Author: feature-b`. Commit.

```bash
git switch main
git switch -c feature-b
```

3. Merge `feature-a` into `feature-b`:

```bash
git merge feature-a
```

`CONFLICT` is expected. Edit `lab_notes.md` to remove `<<<<<<<`, `=======`, and `>>>>>>>`, keep a coherent version, then:

```bash
git add lab_notes.md
git commit -m "Merge feature-a; resolve conflict"
git log --oneline --graph --decorate
```

**Questions:** What does the graph show? Why did you keep the version you kept? Why not merge into `main`?

## Exercise 2: Pull request

Use this to propose your changes for review instead of committing straight to `main`. Create this branch from `main`, **not** from `feature-b` (otherwise the PR includes the merge-conflict history).

```bash
git switch -c pull-request main
```

Replace `ANDREW_ID`, then commit and push to **your fork**:

```
# Lab 3 notes
Andrew ID: ANDREW_ID
```

```bash
git add lab_notes.md
git commit -m "Add lab notes with Andrew ID"
git push -u origin pull-request
```

On GitHub, open a pull request from your `pull-request` branch into `mlip-cmu/minGPT_copy` `main`. Give it a clear title and a 1–2 sentence description. Save the URL.

If GitHub says a PR from this branch already exists, pushing updated it — use that URL.

**Questions:** Clone vs fork? Why open a pull request instead of committing to `main`? What does `git remote -v` show?

## Exercise 3: Safe rollback with `git revert`

Use this to undo a commit you already pushed, without deleting it from history. Stay on `pull-request`. Add the line `TYPO: delete me` to `lab_notes.md` and commit it. Then undo it with a **new** commit:

```bash
git revert HEAD --no-edit
git push origin pull-request
git log --oneline --graph --decorate
```

Point out the revert commit. `--no-edit` skips the editor.

**Questions:** When is `revert` preferred over `reset`? (Think about commits you have already pushed.)

## Exercise 4: Rebase

Use this when `main` has new commits and you need your feature branch caught up before the PR can merge. Do this on a new branch `rebase-demo`, not on `pull-request`.

1. Start from the course `main`, add `student_work.md` with the line `feature work`, commit, and push:

```bash
git fetch upstream
git switch -c rebase-demo upstream/main
git add student_work.md
git commit -m "Add student_work.md"
git push -u origin rebase-demo
```

2. Simulate a teammate landing on `main` (use a different file so the rebase does not conflict):

```bash
git switch -c teammate-update upstream/main
```

Add `teammate_work.md` with the line `teammate work` and commit. Do not push this branch.

3. Replay your work on top of theirs:

```bash
git switch rebase-demo
git rebase teammate-update
```

4. `git push origin rebase-demo` should be **rejected** (non-fast-forward). Then:

```bash
git push --force-with-lease origin rebase-demo
```

Only force-push this feature branch, never `main`.

**Questions:** How is rebase different from merge? Why `--force-with-lease` instead of `--force`? When is force-push not OK?

## Additional resources
- [Learn Git Branching](https://learngitbranching.js.org/)
- [Git Handbook](https://guides.github.com/introduction/git-handbook/)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
