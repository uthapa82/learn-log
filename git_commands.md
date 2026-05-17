# Git Command Reference

This document provides a collection of useful Git commands and operations.

## Table of Contents
1. [Renaming Branches](#renaming-branches)
2. [Pulling Specific Commit](#pulling-specific-commit)
3. [Android Studio Git Setup](#android-studio-git-setup)
4. [General Git Operations](#general-git-operations)
5. [Undoing Commits](#undoing-commits)
6. [Git Log Operations](#git-log-operations)
7. [Git Architecture](#git-architecture)
8. [Working with Files](#working-with-files)
9. [Git with a Real Project](#git-with-a-real-project)
   
---

## Renaming Branches

- **Rename current branch:**
    ```bash
    $ git branch -m <newname>
    ```

- **Rename a branch while pointed to any branch:**
    ```bash
    $ git branch -m <oldname> <newname>
    ```

---

## Pulling Specific Commit

1. View logs:
    ```bash
    $ git log
    ```

2. Fetch the commit:
    ```bash
    $ git fetch origin <sha-from-git-log>
    ```

3. Checkout the commit:
    ```bash
    $ git checkout FETCH_HEAD
    ```

---

## Android Studio Git Setup

1. Initialize a new repository:
    ```bash
    cd <Android Studio project folder>
    git init
    git remote add origin <link-to-repo>
    git fetch origin
    git checkout master
    ```

2. Create a new branch and push:
    ```bash
    git branch <new-branch>
    git checkout <new-branch>
    git push -u origin <new-branch>
    ```

---

## General Git Operations

1. Initialize a new git repository:
    ```bash
    cd to folder
    git init
    git add .
    git commit -m "Comment"
    git branch -m master main
    git remote add origin <URL>
    git push -u origin master  # If error occurs, use `git push -f origin main`
    ```

2. Create a new branch:
    ```bash
    git branch main  # create a new branch 'main'
    git branch  # list all branches
    git checkout main  # switch to the main branch
    git branch -d master  # delete the master branch
    ```

3. Merge into the new default `main` branch (if you have issues):
    ```bash
    git checkout master
    git branch main master -f
    git checkout main
    git push origin main -f
    ```

---

## Undoing Commits

1. **Undo last commit if pushed:**
    ```bash
    git log --oneline
    git reset --hard <commit-sha>  # or
    git reset --hard HEAD~1  # undo last commit
    ```

2. **Undo last commit if not pushed:**
    ```bash
    git reset HEAD~1
    ```

---

## Git Log Operations

1. View log:
    ```bash
    git log
    ```

2. View the most recent 5 logs:
    ```bash
    git log -n 5
    ```

3. View logs since a specific date:
    ```bash
    git log --since="2021-01-08"
    ```

4. View logs by a specific author:
    ```bash
    git log --author="thapa"
    ```

5. Search logs using a regular expression:
    ```bash
    git log --grep="Initi"
    ```

---

## Git Architecture

Git uses a three-tier architecture:
- **Repository** -> **Staging Index** -> **Working Directory**

Process:
- From **working directory**: `git add file.txt` → **staging index**.
- From **staging index**: `git commit file.txt` → **repository**.

**Hash value (SHA-1):**
- Git uses a checksum algorithm (SHA-1) for data integrity. Each change generates a unique 40-character hexadecimal string.

**Example SHA-1 hash**:
5c15e8bd540c113cd2d9eac6f64cacbc5ff6fe9c

yaml
Copy

---

## Working with Files

- **Check the status:**
    ```bash
    git status
    ```

- **Add files:**
    ```bash
    git add .  # Add all files
    git add <filename>  # Add specific file
    ```

- **Commit changes:**
    ```bash
    git commit -m "Adding all files"
    ```

- **Edit files:**
    ```bash
    git diff  # View differences
    git diff --color-words  # View word-level differences
    ```

- **Move or rename files:**
    ```bash
    git mv <source-file> <destination-file>  # Move/rename a file
    ```

- **Remove files:**
    ```bash
    git rm <filename>  # Remove file
    git rm -f <filename>  # Force remove
    ```

---

## Git with a Real Project

- **Initialize Git in a project:**
    ```bash
    git init
    ```

- **Commit all changes:**
    ```bash
    git commit -am "Comment"
    ```

- **View previous commits:**
    ```bash
    git show <commit-sha>
    git show <commit-sha> --color-words
    ```

- **Compare commits:**
    ```bash
    git diff <old-commit-sha>..<new-commit-sha>
    git diff <old-commit-sha>..<new-commit-sha> --color-words
    git diff <old-commit-sha>..HEAD --color-words
    ```

---

### Multiline Commit Messages
To write multiline commit messages, use the following:
```bash
git commit -m "Title of the commit" -m "Detailed description of changes"

* Mistakenly pushed the commit to all 

    ```
    git reflog
    git reset --hard <hash>
    git push --force origin main
    ```