# Removing Large Files from a Git Repository using BFG Repo-Cleaner

BFG Repo-Cleaner is a fast, open-source alternative to `git filter-branch` for cleaning up Git repositories. It’s particularly useful for removing large files or sensitive data from the entire Git history.

**Warning:** BFG rewrites Git history. This will change git hashes in branches and tags.

---

## Prerequisites

- Java (version 6 or later)
- BFG Repo-Cleaner JAR file
- Git CLI installed
- A backup of your repository is recommended

---

## Removing Files

### 1. Download BFG

Run this curl command to download the 1.14.0 version of BFG. This will download the `bfg.jar` to your local directory.

```bash
curl -L -o bfg.jar https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar
```

### 2. Clone a Bare Copy of the Repository that you want to remove files from

```bash
git clone --mirror https://stash.intcx.net/scm/xeb/your-repo.git
cd your-repo.git
```

### 3. Remove Files

Run the `--delete-files` for all the files that need to be removed.

```bash
java -jar bfg.jar --delete-files "large-file.txt"
```

### 4. Clean and Repack the Repository

```bash
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

### 5. Push the Cleaned History (Warning: This will be change commit history in Bitbucket)

```bash
git push --force --all
git push --force --tags
```
