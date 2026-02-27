# Repository Migration Guide (Stash ➜ GitHub)

This document describes how to migrate a repository from **Stash/Bitbucket Server** to **GitHub**, including:

- Cloning the source repo
- Fetching all refs and tags
- Creating local branches that track remote branches
- Adding a GitHub destination remote
- Applying push/network performance settings
- Migrating Git LFS objects
- Pushing all branches and tags to GitHub

---

## Prerequisites

- Git installed
- Network access to:
  - Source (Stash): `https://stash.intcx.net/scm/idspa/eclipse_fork_cleanup.git`
  - Destination (GitHub): `https://github.com/intcx/IDSPA.eclipse_fork_cleanup.git`
- Permissions:
  - Read access to the source repo
  - Push access to the destination repo
- (If used) Git LFS installed: `git lfs version`

---

## Step 1: Clone the source repository

```bash
git clone https://stash.intcx.net/scm/idspa/eclipse_fork_cleanup.git
```

---

## Step 2: Enter the repository directory

```bash
cd eclipse_fork_cleanup/
```

---

## Step 3: Fetch all remotes and tags

```bash
git fetch --all --tags
```

---

## Step 4: Create local branches tracking all remote branches

This step creates a local branch for every branch under `origin/*` (excluding `origin/HEAD`).

```bash
for b in $(git for-each-ref --format='%(refname:short)' refs/remotes/origin/ | grep -v -E '^origin/HEAD$'); do
  lb="${b#origin/}"
  git show-ref --verify --quiet "refs/heads/$lb" || git branch --track "$lb" "$b"
done
```

---

## Step 5: Add the GitHub destination remote

Add a new remote named `destination` pointing to GitHub:

```bash
git remote add destination https://github.com/intcx/IDSPA.eclipse_fork_cleanup.git
```

---

## Step 6: Configure Git for large pushes (recommended)

These settings can help avoid timeouts and improve stability for large repositories.

```bash
git config --global http.postBuffer 524288000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global core.compression 1
git config --global http.maxRequests 1
```

> Note: These are global settings. If you prefer to apply them only to this repo, remove `--global` and run the commands inside the repo.

---

## Step 7: Migrate Git LFS objects (if the repo uses LFS)

Install LFS hooks, fetch all LFS objects, and push them to the destination remote:

```bash
git lfs install
git lfs fetch --all
git lfs push --all destination
```

---

## Step 8: Push all branches to GitHub

This pushes every local branch to the `destination` remote with the same branch name.

```bash
for b in $(git for-each-ref --format='%(refname:short)' refs/heads/); do
  echo "Pushing branch: $b"
  git push destination "$b:$b" || { echo "Failed on $b"; exit 1; }
done
```

---

## Step 9: Push all tags to GitHub

```bash
git push destination --tags
```

---

## Validation / Post-migration Checks

After completion, confirm:

- All branches exist on GitHub
- All tags exist on GitHub
- Default branch is set correctly in GitHub repository settings
- LFS objects are accessible (if applicable)

Useful commands:

```bash
git branch -a
git tag
git remote -v
```

---

## Troubleshooting

### Authentication failures
- Ensure you have permission to push to the GitHub repo.
- If GitHub requires a token, use HTTPS with a token or configure a credential helper.

### Large push timeouts
- Keep the Step 6 settings.
- Retry pushing branches individually (Step 8 already pushes one-by-one).

### LFS missing objects
- Re-run:
  ```bash
  git lfs fetch --all
  git lfs push --all destination
  ```

---

## Summary

1. Clone from Stash  
2. Fetch everything (including tags)  
3. Create local tracking branches  
4. Add GitHub as `destination` remote  
5. Tune Git settings for large transfers  
6. Push LFS objects (if used)  
7. Push all branches  
8. Push all tags  
