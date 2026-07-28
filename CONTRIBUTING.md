# Contributing to Linux Notes & Reference Guide

First off, thank you for considering contributing to this repository! This guide is meant to be a living document, and community contributions are what keep it accurate, comprehensive, and useful for everyone.

Whether you are fixing a typo, adding a new configuration guide for WSL 2, or sharing an obscure but helpful terminal command, your help is appreciated.

## 📋 Table of Contents

* [How Can I Contribute?](#how-can-i-contribute)
* [Submission Workflow](#submission-workflow)
* [Style Guide & Formatting](#style-guide--formatting)
* [Code of Conduct](#code-of-conduct)

---

## 🛠️ How Can I Contribute?

There are several ways you can contribute to this project:

* **Add New Notes**: Have a great cheat sheet for Docker, a Neovim configuration guide, or a fix for Debian package managers? We want it.
* **Update Existing Content**: Linux evolves quickly. If a command is deprecated or a setup guide is out of date, please submit an update.
* **Fix Formatting or Typos**: Clean, readable documentation is key. Minor corrections are always welcome.
* **Suggest Topics**: If you don't have the time to write a guide but want to request one, feel free to open an Issue.

---

## 🔄 Submission Workflow

To submit a contribution, please follow the standard GitHub Pull Request (PR) workflow:

1. **Fork the Repository**: Click the "Fork" button at the top right of the repository page.
2. **Clone Your Fork**:
   ```bash
   git clone [https://github.com/YOUR_USERNAME/linux-notes.git](https://github.com/YOUR_USERNAME/linux-notes.git)
   cd linux-notes
   ```
3. **Create a Branch:** Create a uniquely named branch for your feature or fix.

    ```bash
    git checkout -b add-git-lfs-guide
    ```
4. **Make Your Changes:** Add or edit the Markdown files in the appropriate directory (e.g., docs/04-tools/).
5. **Commit Your Changes:** Write a clear, concise commit message.

    ```bash
    git commit -m "docs: add guide for tracking large files with Git LFS"
    ```
6. **Push to Your Fork:**

    ```bash
    git push origin add-git-lfs-guide
    ```
7. **Open a Pull Request:** Navigate to the original repository and click "Compare & pull request." Provide a brief description of what you added or fixed.

## 📝 Style Guide & Formatting

To keep the repository clean and easily scannable, please adhere to the following formatting guidelines when writing your Markdown files:

1. **File Naming**
    
    Use lowercase letters and hyphens for file names.

        Good: wsl2-ubuntu-setup.md
        Bad: WSL2_Ubuntu Setup.md

2. **Headings and Structure**
    
    Start every new document with a single H1 (#) title, followed by a brief description. Use H2 (##) and H3 (###) for subsequent sections.

3. **Code Blocks**
    
    Always use syntax highlighting for code blocks. Specify the language (e.g., bash, yaml, json, vim).

    Example:
    ```bash
    # Update Debian package signatures
    sudo apt update && sudo apt upgrade -y
    ```
4. **Context is Key**
    
    When adding a command, briefly explain what it does and why it is useful.

        Good: chown -R user:group /var/www — Recursively changes the owner and group of the web directory, which is necessary when setting up local development environments.

        Bad: Run chown -R user:group /var/www.

5. **Categorization**
    
    Place your note in the most relevant folder inside docs/. If you are adding a guide on configuring Zsh plugins like Oh My Zsh, it belongs in docs/05-terminal/. If you are unsure where a file belongs, just place it in the root of docs/, and it can be moved during the PR review.

## 🤝 Code of Conduct

This project is an open and welcoming environment. Please be respectful and constructive in your PR descriptions, issue comments, and code reviews.