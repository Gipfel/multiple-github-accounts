# Multiple Git Accounts (GitHub + Azure DevOps)

This project provides practical setup guides for using **multiple Git identities on one machine** — for example a **work account** and a **private account** — without mixing SSH keys, remotes, or commit identities.

## What this project does

- Explains how to configure separate SSH keys per account
- Shows how to use SSH host aliases for multiple GitHub accounts
- Describes folder-based Git identity switching using `includeIf`
- Covers Azure DevOps alongside GitHub (and the same approach works for Git-only workflows)

## ✅ Why this is useful

When you work across private and company repositories, it is easy to accidentally:

- push with the wrong account
- commit with the wrong email/name
- reuse the wrong SSH key

These guides help you keep accounts cleanly isolated and make your default identity explicit.

## 👤 Who this is for

- Developers with both **private** and **work** projects
- Teams using **GitHub** for one context and **Azure DevOps** for another
- Anyone who needs predictable, folder-based account separation on Windows

## 📘 Setup guides

- English guide: [instructions.md](instructions.md)
- Deutsche Anleitung: [instructions-de.md](instructions-de.md)

## In short

You get a scalable, low-friction way to manage multiple Git accounts on one computer with clear authentication and commit identity boundaries.
