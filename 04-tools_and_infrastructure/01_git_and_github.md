# Git & GitHub Essentials for Linux

This guide covers the fundamental Git commands and GitHub workflows necessary for managing source code, collaborating on repositories, and tracking project history within a Linux environment (such as Debian, Ubuntu, or WSL 2).

---

## ⚙️ 1. Initial Setup and Authentication

Before committing code, you need to configure your identity and set up secure authentication with GitHub.

### Global Configuration

Set your username and email address. This information is attached to every commit you make.

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### SSH Key Authentication (Recommended)

For secure, passwordless authentication with GitHub, generate an Ed25519 SSH key.

```bash
# Generate the SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Start the SSH agent in the background
eval "$(ssh-agent -s)"

# Add your SSH private key to the ssh-agent
ssh-add ~/.ssh/id_ed25519

# Output the public key to copy and paste into your GitHub settings
cat ~/.ssh/id_ed25519.pub
```

## 🚀 2. Core Workflow Commands

These are the everyday commands used to track changes and sync local code with remote GitHub repositories.

| Command | Description | Example |
| ------- | ----------- | ------- |
| `git init` | Initializes a new, empty Git repository in the current directory. | `git init` |
| `git clone` | Downloads a complete copy of a remote repository to your local machine. | `git clone git@github.com:User/repo.git` |
| `git status` | Displays the state of the working directory and the staging area (shows modified, staged, and untracked files). | `git status` |
| `git add` | Adds file contents to the staging area, preparing them for the next commit. | `git add script.py` or `git add .` (for all) |
| `git commit` | Records the staged snapshots into the repository's history with a descriptive message. | `git commit -m "feat: integrate FastAPI endpoints"` |
| `git push` | Uploads local branch commits to the remote repository. | `git push origin main` |
| `git pull` | Fetches updates from the remote repository and immediately merges them into the current local branch. | `git pull origin main` |

## 🌿 3. Branching and Merging

Branches allow you to work on new features or bug fixes in an isolated environment without affecting the main production codebase.

* List all local branches:

    ```bash
    git branch
    ```

* Create and switch to a new branch:

    ```bash
    git checkout -b feature/update-database-schema
    # Alternatively, in newer Git versions:
    git switch -c feature/update-database-schema
    ```

* Switch to an existing branch:

    ```bash
    git checkout main
    ```

* Merge a branch into your current branch:
    (Example: Merging the feature branch into main)

    ```bash
    git checkout main
    git merge feature/update-database-schema
    ```

* Delete a local branch:

    ```bash
    git branch -d feature/update-database-schema
    ```

## 📦 4. Managing Large Datasets and Models (Git LFS)

Standard Git is not optimized for large binary files, deep learning models (like `.pt` or `.onnx`), or heavy dataset archives. *Git Large File Storage (LFS)* replaces these large files with text pointers inside Git, while storing the file contents on a remote server.

**Initial Setup (Run once per Linux system):**

```bash
sudo apt install git-lfs
git lfs install
```

**Tracking Large Files in a Project:**

```bash
# Tell LFS to track specific file types (e.g., PyTorch models or large datasets)
git lfs track "*.pt"
git lfs track "*.csv"

# Ensure the .gitattributes file is tracked by standard Git
git add .gitattributes
git commit -m "chore: configure Git LFS for model weights"
```

## 🔍 5. Inspecting History and Reverting

**View commit history:**

```bash
git log
# For a cleaner, single-line view:
git log --oneline --graph --all
```

**See what was modified in a file before staging:**

```bash
git diff
```

**Temporarily shelve uncommitted changes (Stashing):**

```bash
git stash         # Hides current changes to give you a clean working directory
git stash pop     # Restores the most recently stashed changes
```

**Unstage a accidentally added file:**

```bash
Bashgit restore --staged <file>
```
