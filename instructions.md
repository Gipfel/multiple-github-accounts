# Git Multi-Account Setup on Windows

This guide explains how to cleanly set up multiple Git accounts (e.g. GitHub + Azure DevOps) on a Windows machine — **folder-based**, with a defined default account.

---

## Concept

Two things need to be configured — they work independently of each other:

| What | Purpose | Where configured |
|---|---|---|
| SSH Keys | Authentication with the server (push/pull) | `~/.ssh/config` |
| Name + Email | Commit identity (`git log`) | `~/.gitconfig` + separate profile files |

`~` on Windows translates to: `C:\Users\<YourUsername>\`

---

## Step 1 – Generate SSH Keys

Open PowerShell and run the following commands:

```powershell
# GitHub Account 1 (e.g. Work)
ssh-keygen -t ed25519 -C "work@github" -f $HOME\.ssh\id_github_work

# GitHub Account 2 (e.g. Personal)
ssh-keygen -t ed25519 -C "personal@github" -f $HOME\.ssh\id_github_personal

# Azure DevOps → must be RSA (Azure does not accept ed25519)
ssh-keygen -t rsa -b 4096 -C "work@azure" -f $HOME\.ssh\id_azure
```

> When asked for a passphrase, just press **Enter** to skip.

Verify the result — the following files should exist:
```powershell
Get-ChildItem $HOME\.ssh\
```
```
id_github_work       id_github_work.pub
id_github_personal   id_github_personal.pub
id_azure             id_azure.pub
```

---

## Step 2 – Register Public Keys on each Platform

Copy the content of each `.pub` file and register it on the respective platform:

```powershell
# Display for copying:
Get-Content $HOME\.ssh\id_github_work.pub
Get-Content $HOME\.ssh\id_github_personal.pub
Get-Content $HOME\.ssh\id_azure.pub
```

**GitHub** (log into each account separately):
→ [github.com/settings/keys](https://github.com/settings/keys) → "New SSH key"

**Azure DevOps**:
→ `dev.azure.com` → Avatar top right → **User Settings** → **SSH Public Keys** → "Add"

---

## Step 3 – Set up SSH Config

Open the file (it will be created if it doesn't exist):
```powershell
notepad $HOME\.ssh\config
```

Content:
```
# GitHub Work Account
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_github_work

# GitHub Personal Account
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_github_personal

# Azure DevOps
Host ssh.dev.azure.com
  HostName ssh.dev.azure.com
  User git
  IdentityFile ~/.ssh/id_azure
```

> **Important:** For GitHub, **aliases** (`github-work`, `github-personal`) are used because two accounts share the same hostname `github.com`. Azure DevOps only has one account here, so the real hostname is used directly.

---

## Step 4 – Update Remote URLs (GitHub with Aliases)

Since GitHub repos are now accessed via aliases, existing repos need their remote URLs updated once:

```bash
# Instead of: git@github.com:organisation/repo.git
# Work:
git remote set-url origin git@github-work:organisation/repo.git

# Personal:
git remote set-url origin git@github-personal:yourname/repo.git
```

For new repos, clone using the alias from the start:
```bash
git clone git@github-work:organisation/repo.git
```

Azure DevOps URLs remain unchanged since the real hostname is used directly.

---

## Step 5 – Configure Git Identities (Name + Email)

### Folder Structure (Example)

```
Desktop\
  Work Git\         ← GitHub Work repos
  Work Azure\       ← Azure DevOps repos
  Personal Git\     ← GitHub Personal repos
  Other\            ← everything else → default account applies
```

### Main gitconfig (`~/.gitconfig`)

```properties
# Default account (applies everywhere no includeIf matches)
[user]
  name = Your Name
  email = your@default-email.com

# GitHub Work → overrides default in this folder
[includeIf "gitdir/i:C:/Users/<YourUsername>/Desktop/Work Git/"]
  path = ~/.gitconfig-github-work

# Azure DevOps → overrides default in this folder
[includeIf "gitdir/i:C:/Users/<YourUsername>/Desktop/Work Azure/"]
  path = ~/.gitconfig-azure

# GitHub Personal → overrides default in this folder
[includeIf "gitdir/i:C:/Users/<YourUsername>/Desktop/Personal Git/"]
  path = ~/.gitconfig-github-personal
```

### Create Profile Files

**`~/.gitconfig-github-work`**
```properties
[user]
  name = Your Name
  email = you@company.com
```

**`~/.gitconfig-azure`**
```properties
[user]
  name = Your Name
  email = you@company.com
```

**`~/.gitconfig-github-personal`**
```properties
[user]
  name = Your Name
  email = you@personal.com
```

---

## Step 6 – Test Everything

### Test SSH connections
```bash
# GitHub Work
ssh -T git@github-work
# Expected: "Hi <username>! You've successfully authenticated..."

# GitHub Personal
ssh -T git@github-personal
# Expected: "Hi <username>! You've successfully authenticated..."

# Azure DevOps
ssh -T git@ssh.dev.azure.com
# Expected: "remote: Shell access is not supported." → This is correct and means it worked!
```

### Test Git identity

Inside a repo under the respective folder:
```bash
# Shows which email is active and which file it comes from:
git config --show-origin user.email
```

---

## Summary – How it scales

```
New account?
  1. ssh-keygen → generate a new key
  2. Register .pub key on the platform
  3. ~/.ssh/config → add a new Host entry
  4. ~/.gitconfig → add a new includeIf block
  5. ~/.gitconfig-<name> → create a new profile file
```

Every account is fully isolated. The default account automatically applies to all repos that don't fall under any of the defined folders.

---

## Quick Reference

| Account | SSH Host Alias | Profile File | Folder |
|---|---|---|---|
| Default | – | `~/.gitconfig` directly | everywhere else |
| GitHub Work | `github-work` | `~/.gitconfig-github-work` | `Desktop\Work Git\` |
| Azure DevOps | `ssh.dev.azure.com` | `~/.gitconfig-azure` | `Desktop\Work Azure\` |
| GitHub Personal | `github-personal` | `~/.gitconfig-github-personal` | `Desktop\Personal Git\` |