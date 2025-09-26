# Git LFS Complete Tutorial
## Mastering Large File Storage with Git

---

# Table of Contents

1. [Introduction to Git LFS](#1-introduction-to-git-lfs)
2. [Why Git LFS is Needed](#2-why-git-lfs-is-needed)
3. [Core Concepts](#3-core-concepts)
4. [Installation Guide](#4-installation-guide)
5. [Basic Usage](#5-basic-usage)
6. [Advanced Operations](#6-advanced-operations)
7. [Best Practices](#7-best-practices)
8. [Troubleshooting](#8-troubleshooting)
9. [Quick Reference](#9-quick-reference)

---

# 1. Introduction to Git LFS

## What is Git LFS?
Git Large File Storage (LFS) is a Git extension that replaces large files with text pointers inside Git while storing the actual file contents on a remote server.

## The Problem It Solves
- Git stores complete file snapshots for every version
- Large binary files bloat repository size<repository-url>
- Slow cloning and fetching operations
- Inefficient for assets like videos, designs, binaries

---

# 2. Why Git LFS is Needed

## Without Git LFS
```
Repository with 100MB video file:
- Commit 1: +100MB
- Commit 2: +100MB (another snapshot)
- Commit 3: +100MB (another snapshot)
- Total: 300MB for 3 versions!
```

## With Git LFS
```
Repository with 100MB video file:
- Commit 1: +1KB (pointer) + 100MB (LFS store)
- Commit 2: +1KB (pointer) + changes (LFS store)
- Commit 3: +1KB (pointer) + changes (LFS store)
- Git repo: Only stores tiny pointers!
```

---

# 3. Core Concepts

## Pointer Files
Git LFS replaces files with pointers that look like:
```
version https://git-lfs.github.com/spec/v1
oid sha256:4d7a214614ab2935c943f9e0ff69d223a943861
size 1575078
```

## Architecture
```
Working Directory ──────→ Git Repository ──────→ Remote Git
     (Large files)          (Pointers only)       (Pointers)
         │                         │
         └─────→ LFS Cache ────────┘
                         │
                         └─────→ LFS Server (Actual file storage)
```

## Key Components
1. **LFS Client**: `git-lfs` command line tool
2. **Pointer Files**: Text references to actual content
3. **LFS Server**: Remote storage (GitHub, GitLab, etc.)
4. **.gitattributes**: Configuration file

---

# 4. Installation Guide

## macOS
```bash
brew install git-lfs
git lfs install
```

## Ubuntu/Debian
```bash
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
git lfs install
```

## Windows
1. Download from [git-lfs.com](https://git-lfs.com)
2. Run installer
3. Open Git Bash or Command Prompt:
```bash
git lfs install
```

## Verify Installation
```bash
git lfs version
# Should show: git-lfs/x.x.x (GitHub; ...)
```

---

# 5. Basic Usage

## Step 1: Initialize New Repository
```bash
mkdir my-project
cd my-project
git init
git lfs install --local
```

## Step 2: Track File Types
```bash
# Track specific extensions
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "*.zip"

# Track directories
git lfs track "assets/**"
git lfs track "videos/*.mp4"

# View tracked patterns
git lfs track
```

## Step 3: First Commit
```bash
# .gitattributes is automatically created/modified
git add .gitattributes

# Add your large files
git add design.psd
git add presentation.mp4

git commit -m "Add design assets with LFS"
git log --oneline
```

## Step 4: Push to Remote
```bash
git remote add origin https://github.com/user/repo.git
git push -u origin main

# Watch the two-step process:
# 1. Git objects pushed (including pointers)
# 2. LFS files uploaded to LFS storage
```

---

# 6. Advanced Operations

## Cloning LFS Repositories
```bash
# Clone normally (LFS files download automatically if LFS installed)
git clone https://github.com/user/repo.git
cd repo

# If LFS files aren't downloaded:
git lfs pull

# Check LFS files status
git lfs ls-files
```

## Managing Tracked Files
```bash
# Add new file types
git lfs track "*.ai"
git add .gitattributes
git commit -m "Track Illustrator files"

# Stop tracking a pattern
# Remove line from .gitattributes or use:
git lfs untrack "*.zip"

# List all LFS files
git lfs ls-files

# List with sizes
git lfs ls-files --size
```

## Migration from Regular Git to LFS
```bash
# For new repositories (rewrites history)
git lfs migrate import --include="*.psd,*.mp4" --everything

# For existing shared repositories (safer approach):
# 1. Track new files with LFS going forward
# 2. Keep existing large files in Git history
# 3. Or use BFG Repo-Cleaner for careful history rewriting
```

## Team Collaboration
```bash
# When pulling changes that include LFS files
git pull origin main
git lfs pull  # Ensure all LFS files are downloaded

# Check repository health
git lfs fsck

# See LFS environment info
git lfs env
```

---

# 7. Best Practices

## File Tracking Strategy
```bash
# Good: Track specific binary formats
git lfs track "*.{psd,ai,indd}"
git lfs track "*.{mp4,mov,avi}"
git lfs track "*.{zip,rar,7z}"

# Good: Track asset directories
git lfs track "assets/**"
git lfs track "build/outputs/*.apk"

# Avoid: Tracking text files
# DON'T: git lfs track "*.txt"
# DON'T: git lfs track "*.java"
```

## .gitattributes Management
```gitattributes
# Sample .gitattributes file
*.psd filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
assets/** filter=lfs diff=lfs merge=lfs -text
*.ai filter=lfs diff=lfs merge=lfs -text
```

## Repository Organization
```
project/
├── src/           # Source code (regular Git)
├── assets/        # Large assets (LFS tracked)
│   ├── designs/
│   └── videos/
├── documentation/
└── builds/        # Build outputs (LFS tracked)
```

## CI/CD Integration
```yaml
# GitHub Actions example
name: Build
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        lfs: true  # Critical for LFS
    - name: Build
      run: |
        # LFS files are available
```

---

# 8. Troubleshooting

## Common Issues and Solutions

### Issue: LFS files show as pointers
```bash
# Files appear as small text files instead of actual content
git lfs install
git lfs pull
```

### Issue: File added without LFS tracking
```bash
# Accidentally added large file to regular Git
git rm --cached large_file.zip
git lfs track "*.zip"
git add large_file.zip
git commit -m "Move file to LFS tracking"
```

### Issue: LFS hooks not installed
```bash
# Hooks missing after clone
git lfs install
git lfs pull
```

### Issue: Push fails with LFS error
```bash
# Check LFS status
git lfs status

# Try pushing LFS files separately
git lfs push --all origin main

# Check available space on LFS server (via host UI)
```

## Maintenance Commands
```bash
# Clean up local LFS cache
git lfs prune

# Check consistency
git lfs fsck

# View LFS logs
git lfs logs last

# See storage info
git lfs env
```

## Debugging Tips
```bash
# Enable verbose output
GIT_LFS_VERBOSE=1 git lfs pull

# See what LFS is doing
git lfs status

# Check pointer files
git cat-file -p HEAD:path/to/file.psd
```

---

# 9. Quick Reference

## Essential Commands
| Command | Purpose |
|---------|---------|
| `git lfs track "*.ext"` | Track file type with LFS |
| `git lfs ls-files` | List LFS-tracked files |
| `git lfs pull` | Download LFS files |
| `git lfs status` | Check LFS status |
| `git lfs env` | Show LFS environment |

## Installation
```bash
# One-time setup
git lfs install

# Per-repository setup
git lfs install --local
```

## File Management
```bash
# Track multiple patterns
git lfs track "*.{psd,ai,indd}"
git lfs track "assets/**"

# View tracked patterns
git lfs track

# Untrack pattern
git lfs untrack "*.zip"
```

## Remote Operations
```bash
# Push with LFS
git push origin main

# Clone with LFS
git clone <url>
git lfs pull

# Fetch LFS files specifically
git lfs fetch
git lfs checkout
```

## Maintenance
```bash
# Cleanup
git lfs prune

# Verification
git lfs fsck

# Logs
git lfs logs last
```

---

# Cheat Sheet

## Quick Start for New Repository
```bash
git init
git lfs install --local
git lfs track "*.psd" "*.mp4" "assets/**"
git add .gitattributes
git add .
git commit -m "Initial commit with LFS"
git remote add origin <url>
git push -u origin main
```

## Daily Workflow
```bash
# Start work
git pull origin main
git lfs pull

# Make changes
git add .
git commit -m "Update assets"

# Share changes
git push origin main
```

## Team Member Joining
```bash
git clone <repository-url>
cd repository
# LFS files download automatically if LFS installed
# If not: git lfs pull
```

---

# Appendix: Host-Specific Limits

## GitHub LFS Limits
- **Bandwidth**: 1GB/month for personal, 10GB/month for organization
- **Storage**: 1GB free, additional storage purchasable
- **File Size**: Max 2GB per file

## GitLab LFS Limits
- **Storage**: 10GB free on SaaS, configurable on self-hosted
- **File Size**: Configurable, typically 5-10GB max

## Bitbucket LFS Limits
- **Storage**: 1-10GB depending on plan
- **Bandwidth**: Varies by plan

---

# Final Tips

1. **Track early**: Set up LFS before adding large files
2. **Commit .gitattributes**: Essential for team consistency
3. **Monitor quotas**: Keep an eye on LFS storage usage
4. **Use for binaries only**: Text files should stay in regular Git
5. **Train your team**: Ensure everyone has LFS installed

## Remember:
"Git LFS keeps your repository light by storing heavy files elsewhere!"

---

*This tutorial is available online at: https://github.com/git-lfs/git-lfs/wiki/Tutorial*
*Git LFS Documentation: https://git-lfs.com/*

**Happy version controlling! 🚀**

---

## To download this as a PDF:

1. **Copy the entire content above**
2. **Paste into a text editor** (VS Code, Sublime Text, etc.)
3. **Save as `git-lfs-tutorial.txt`**
4. **Use any of these methods to convert to PDF:**

### Method 1: Using Google Docs
1. Go to [Google Docs](https://docs.google.com)
2. Create new document → Paste content
3. File → Download → PDF Document (.pdf)

### Method 2: Using Microsoft Word
1. Open Microsoft Word
2. Paste content
3. File → Save As → Choose PDF format

### Method 3: Online Converters
1. Use free tools like:
   - [PDF24](https://tools.pdf24.org/en/txt-to-pdf)
   - [SmallPDF](https://smallpdf.com/txt-to-pdf)
2. Upload your `.txt` file
3. Download converted PDF

### Method 4: Command Line (Linux/macOS)
```bash
# Install pandoc first: https://pandoc.org/installing.html
pandoc git-lfs-tutorial.txt -o git-lfs-tutorial.pdf
```

**Pro Tip**: For the best formatting, consider using Markdown and converting with tools like Typora or Obsidian for beautiful PDF output!
