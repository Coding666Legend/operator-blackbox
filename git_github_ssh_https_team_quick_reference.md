# GitHub Quick Reference — SSH, HTTPS & Team Collaboration

Ubuntu 24.04 / Kali Linux

Replace `USERNAME`, `REPOSITORY`, and branch names with your actual values. Run repository commands from inside the cloned project directory unless stated otherwise.

## 1. Install Git and Configure Identity

```bash
sudo apt update
sudo apt install git -y

git config --global user.name "Your Name"
git config --global user.email "you@example.com"

git --version
git config --global --list
```

## 2. SSH Setup (once per machine/account)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

Add the public key at **GitHub → Settings → SSH and GPG keys → New SSH key**.

```bash
ssh -T git@github.com
```

Never share `~/.ssh/id_ed25519` (the private key).

## 3. Clone a Repository

**SSH**
```bash
git clone git@github.com:USERNAME/REPOSITORY.git
cd REPOSITORY
```

**HTTPS**
```bash
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
```

Check remotes:

```bash
git remote -v
```

## 4. Connect an Existing Local Project

```bash
git init
git branch -M main
```

**SSH remote**
```bash
git remote add origin git@github.com:USERNAME/REPOSITORY.git
```

**HTTPS remote**
```bash
git remote add origin https://github.com/USERNAME/REPOSITORY.git
```

If `origin` already exists, use `git remote set-url origin URL` instead of `git remote add`.

## 5. Switch Remote Protocol

**HTTPS → SSH**
```bash
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```

**SSH → HTTPS**
```bash
git remote set-url origin https://github.com/USERNAME/REPOSITORY.git
```

Verify:

```bash
git remote -v
```

## 6. Daily Add, Commit and Push

```bash
git status
git add filename.txt       # Stage one file
git add .                  # Stage changes under current directory
git diff --cached          # Review staged changes
git commit -m "Describe changes"
git push -u origin BRANCH  # First push; sets upstream
git push                   # Later pushes to that upstream
```

## 7. Fetch, Pull and Push

```bash
git fetch origin           # Download remote refs/commits; do not integrate
git branch -r              # List remote-tracking branches
git log --oneline origin/main
git pull                   # Fetch + integrate configured upstream
git pull origin main       # Pull a specific remote branch into current branch
git push origin main       # Push local main to remote main
```

- **Fetch:** downloads updates; working files and current branch are not merged/changed.
- **Pull:** fetches and integrates changes into the current branch (merge or configured rebase).
- **Push:** sends local commits to a remote branch.
- Fetch before starting work and before pushing when teammates may have updated shared branches.

## 8. Branching Basics

```bash
git branch                 # List local branches
git branch -a              # List local and remote-tracking branches
git switch main             # Switch to existing local branch
git switch -c feature-x     # Create and switch to a new branch
git push -u origin feature-x
git branch -d feature-x     # Delete local branch (if merged)
```

Update local `main`:

```bash
git switch main
git pull origin main
```

Create a task branch from updated `main`:

```bash
git switch -c feature/short-description
```

## 9. Push to Main vs. Push to a Sub-Branch

**Push the local `main` branch to remote `main`** (only if team policy permits direct pushes):

```bash
git switch main
git pull origin main
git add .
git commit -m "Describe changes"
git push origin main
```

**Push a feature/sub-branch** (normal collaborative workflow):

```bash
git switch -c feature/login-page
git add .
git commit -m "Add login page"
git push -u origin feature/login-page
```

Then open a **Pull Request (PR)** on GitHub to merge `feature/login-page` into `main`. A branch is not automatically merged just because it was pushed.

## 10. Team Workflow — 1 Team Lead + 3 Teammates

Example branch plan:

| Person | Working branch | Typical responsibility |
|---|---|---|
| Team lead | `main` and review/merge PRs | Integration, review, releases |
| Teammate 1 | `feature/teammate-1` | Own assigned task |
| Teammate 2 | `feature/teammate-2` | Own assigned task |
| Teammate 3 | `feature/teammate-3` | Own assigned task |

**Recommended collaboration rule:** protect `main`; teammates push only to their own branches and submit PRs. The lead reviews and merges PRs. Give each person their own GitHub account and repository permissions—never share SSH private keys or account credentials.

### One-time setup for each teammate

Each teammate clones the same repository using SSH or HTTPS:

```bash
git clone git@github.com:ORG/REPOSITORY.git
cd REPOSITORY
git switch main
git pull origin main
git switch -c feature/teammate-1
```

Teammates 2 and 3 should use their own branch names, for example `feature/teammate-2` and `feature/teammate-3`.

### Each teammate's daily cycle

Replace the branch name with their own branch:

```bash
git switch feature/teammate-1
git switch main
git pull origin main
git switch feature/teammate-1
git merge main                 # Bring updated main into the feature branch
git status
git add .
git commit -m "Describe task completed"
git push -u origin feature/teammate-1  # First push
git push                       # Later pushes
```

If the feature branch already exists locally, do **not** recreate it with `git switch -c`. The `git merge main` step may cause conflicts; resolve them, test, then commit the resolution before pushing.

### Team lead's integration cycle

1. Review the teammate's PR and changes.
2. Request changes or approve.
3. Merge the PR into `main` using the team's chosen merge method.
4. Teammates update their local `main` and feature branches before continuing work.

After a PR is merged, teammates can update their local `main`:

```bash
git switch main
git pull origin main
```

### Important collaboration rules

- Do not have all teammates commit directly to `main`.
- Use separate branches for separate tasks; avoid two people editing the same files where possible.
- Pull/fetch frequently, and keep feature branches reasonably short-lived.
- Resolve merge conflicts deliberately; inspect files and run tests before committing.
- A successful `git push` means the remote branch was updated—not that the PR was approved or merged.

## 11. Inspect Changes, Branches and History

```bash
git status
git diff
git diff --cached
git log --oneline --graph --all
git remote -v
git branch -vv
git show --stat
```

## 12. Remove GitHub SSH Usage (Optional)

Switch the repository to HTTPS first:

```bash
git remote set-url origin https://github.com/USERNAME/REPOSITORY.git
```

Unload the key from the agent:

```bash
ssh-add -d ~/.ssh/id_ed25519
```

Then remove the key at **GitHub → Settings → SSH and GPG keys**. Delete the local keypair only if it is not used elsewhere:

```bash
rm ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
```

This does not uninstall SSH from Linux or change Git history.

## Quick Mental Model

- `clone` — copy repository and history locally
- `remote` — configure a remote URL
- `branch` — separate line of development
- `add` — stage changes
- `commit` — record staged changes locally
- `fetch` — download remote updates
- `pull` — fetch and integrate updates
- `push` — upload local commits
- **Pull Request** — request review and integration of a branch into another branch

**Authentication:** SSH uses a registered SSH key. HTTPS uses a credential manager or personal access token; GitHub account passwords are not accepted for Git HTTPS authentication.
