# Git Multi-Account Setup auf Windows

Diese Anleitung erklärt, wie du mehrere Git-Accounts (z.B. GitHub + Azure DevOps) sauber auf einem Windows-Rechner einrichtest – **ordnerbasiert**, mit einem definierten Standard-Account.

---

## Konzept

Zwei Dinge müssen konfiguriert werden – sie sind unabhängig voneinander:

| Was | Wozu | Wo konfiguriert |
|---|---|---|
| SSH Keys | Authentifizierung beim Server (push/pull) | `~/.ssh/config` |
| Name + Email | Commit-Identität (`git log`) | `~/.gitconfig` + separate Profil-Dateien |

`~` entspricht auf Windows: `C:\Users\<DeinBenutzername>\`

---

## Schritt 1 – SSH Keys generieren

PowerShell öffnen und folgende Befehle ausführen:

```powershell
# GitHub Account 1 (z.B. Work)
ssh-keygen -t ed25519 -C "work@github" -f $HOME\.ssh\id_github_work

# GitHub Account 2 (z.B. Personal)
ssh-keygen -t ed25519 -C "personal@github" -f $HOME\.ssh\id_github_personal

# Azure DevOps → muss RSA sein (Azure akzeptiert kein ed25519)
ssh-keygen -t rsa -b 4096 -C "work@azure" -f $HOME\.ssh\id_azure
```

> Bei der Frage nach einer Passphrase einfach **Enter** drücken.

Ergebnis prüfen – folgende Dateien sollten vorhanden sein:
```powershell
Get-ChildItem $HOME\.ssh\
```
```
id_github_work       id_github_work.pub
id_github_personal   id_github_personal.pub
id_azure             id_azure.pub
```

---

## Schritt 2 – Public Keys auf den Plattformen eintragen

Den Inhalt der `.pub`-Dateien jeweils kopieren und auf der Plattform eintragen:

```powershell
# Anzeigen zum Kopieren:
Get-Content $HOME\.ssh\id_github_work.pub
Get-Content $HOME\.ssh\id_github_personal.pub
Get-Content $HOME\.ssh\id_azure.pub
```

**GitHub** (für jeden Account separat einloggen):
→ [github.com/settings/keys](https://github.com/settings/keys) → „New SSH key"

**Azure DevOps**:
→ `dev.azure.com` → Avatar oben rechts → **User Settings** → **SSH Public Keys** → „Add"

---

## Schritt 3 – SSH Config einrichten

Datei öffnen (wird erstellt falls nicht vorhanden):
```powershell
notepad $HOME\.ssh\config
```

Inhalt:
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

> **Wichtig:** Für GitHub werden **Aliases** (`github-work`, `github-personal`) definiert, weil zwei Accounts denselben Hostnamen `github.com` haben. Azure DevOps hat nur einen Account, daher wird der echte Hostname direkt verwendet.

---

## Schritt 4 – Remote URLs anpassen (GitHub mit Aliases)

Weil GitHub-Repos jetzt über Aliases angesprochen werden, müssen die Remote-URLs in bestehenden Repos einmalig angepasst werden:

```bash
# Statt: git@github.com:organisation/repo.git
# Work:
git remote set-url origin git@github-work:organisation/repo.git

# Personal:
git remote set-url origin git@github-personal:deinname/repo.git
```

Neue Repos direkt mit dem Alias klonen:
```bash
git clone git@github-work:organisation/repo.git
```

Bei Azure DevOps bleiben die URLs unverändert, da der echte Hostname direkt verwendet wird.

---

## Schritt 5 – Git Identitäten (Name + Email) konfigurieren

### Ordnerstruktur (Beispiel)

```
Desktop\
  Arbeit Git\       ← GitHub Work Repos
  Arbeit Azure\     ← Azure DevOps Repos
  Privat Git\       ← GitHub Personal Repos
  Sonstiges\        ← alles andere → Standard-Account greift
```

### Haupt-gitconfig (`~/.gitconfig`)

```properties
# Standard-Account (greift überall wo kein includeIf passt)
[user]
  name = Dein Name
  email = dein@standard-email.com

# GitHub Work → überschreibt Standard in diesem Ordner
[includeIf "gitdir/i:C:/Users/<DeinBenutzername>/Desktop/Arbeit Git/"]
  path = ~/.gitconfig-github-work

# Azure DevOps → überschreibt Standard in diesem Ordner
[includeIf "gitdir/i:C:/Users/<DeinBenutzername>/Desktop/Arbeit Azure/"]
  path = ~/.gitconfig-azure

# GitHub Personal → überschreibt Standard in diesem Ordner
[includeIf "gitdir/i:C:/Users/<DeinBenutzername>/Desktop/Privat Git/"]
  path = ~/.gitconfig-github-personal
```

### Profil-Dateien erstellen

**`~/.gitconfig-github-work`**
```properties
[user]
  name = Dein Name
  email = du@firma.com
```

**`~/.gitconfig-azure`**
```properties
[user]
  name = Dein Name
  email = du@firma.com
```

**`~/.gitconfig-github-personal`**
```properties
[user]
  name = Dein Name
  email = du@privat.com
```

---

## Schritt 6 – Alles testen

### SSH Verbindungen testen
```bash
# GitHub Work
ssh -T git@github-work
# Erwartet: "Hi <username>! You've successfully authenticated..."

# GitHub Personal
ssh -T git@github-personal
# Erwartet: "Hi <username>! You've successfully authenticated..."

# Azure DevOps
ssh -T git@ssh.dev.azure.com
# Erwartet: "remote: Shell access is not supported." → Das ist korrekt und bedeutet Erfolg!
```

### Git Identität testen

In einem Repo unter dem jeweiligen Ordner:
```bash
# Zeigt welche Email aktiv ist und aus welcher Datei sie kommt:
git config --show-origin user.email
```

---

## Zusammenfassung – Wie es skaliert

```
Neuer Account?
  1. ssh-keygen → neuen Key generieren
  2. .pub Key auf der Plattform eintragen
  3. ~/.ssh/config → neuen Host-Eintrag hinzufügen
  4. ~/.gitconfig → neuen includeIf-Block hinzufügen
  5. ~/.gitconfig-<n> → neue Profil-Datei anlegen
```

Jeder Account ist vollständig isoliert. Der Standard-Account greift automatisch für alle Repos, die in keinem der definierten Ordner liegen.

---

## Schnellreferenz

| Account | SSH Host Alias | Profil-Datei | Ordner |
|---|---|---|---|
| Standard | – | `~/.gitconfig` direkt | überall sonst |
| GitHub Work | `github-work` | `~/.gitconfig-github-work` | `Desktop\Arbeit Git\` |
| Azure DevOps | `ssh.dev.azure.com` | `~/.gitconfig-azure` | `Desktop\Arbeit Azure\` |
| GitHub Personal | `github-personal` | `~/.gitconfig-github-personal` | `Desktop\Privat Git\` |