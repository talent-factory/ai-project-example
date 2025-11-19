# Branch Protection Setup

Dieses Dokument erklärt, wie man Branch Protection Rules für `main` und `develop` Branches einrichtet, um direkte Pushes zu verhindern und Pull Requests zu erzwingen.

## Warum Branch Protection?

Branch Protection stellt sicher, dass:
- ✅ Alle Änderungen durch Pull Requests gehen
- ✅ Code-Reviews durchgeführt werden
- ✅ CI/CD-Tests laufen bevor gemerged wird
- ✅ Die Git-Historie sauber bleibt
- ✅ Nur autorisierte Personen mergen können

## Setup auf GitHub

### Schritt 1: Repository Settings öffnen

1. Gehe zu deinem Repository auf GitHub
2. Klicke auf **Settings** (oben rechts)
3. Im linken Menü: **Branches**

### Schritt 2: Branch Protection Rule erstellen

1. Klicke auf **Add rule** oder **Add branch protection rule**
2. **Branch name pattern**: `main` (für main branch)

### Schritt 3: Schutzregeln konfigurieren

#### ✅ Require a pull request before merging
- **Aktivieren**: ✓ Require a pull request before merging
- **Required approvals**: 1 (mindestens ein Review)
- Optional: ✓ Dismiss stale pull request approvals when new commits are pushed
- Optional: ✓ Require review from Code Owners

#### ✅ Require status checks to pass before merging
- **Aktivieren**: ✓ Require status checks to pass before merging
- **Status checks**: Wähle CI/CD-Checks aus (wenn vorhanden)
- Optional: ✓ Require branches to be up to date before merging

#### ✅ Require conversation resolution before merging
- **Aktivieren**: ✓ Require conversation resolution before merging
- Stellt sicher, dass alle PR-Kommentare aufgelöst sind

#### ✅ Do not allow bypassing the above settings
- **Aktivieren**: ✓ Do not allow bypassing the above settings
- Niemand kann diese Regeln umgehen (auch Admins nicht, es sei denn temporär deaktiviert)

#### ⚠️ Restrict who can push to matching branches
- **Optional**: Wenn du möchtest, dass nur bestimmte Teams/Personen pushen können
- Lasse dies leer, wenn PRs für alle erlaubt sein sollen

### Schritt 4: Regel speichern

Klicke auf **Create** oder **Save changes**

### Schritt 5: Regel für `develop` wiederholen

Erstelle eine zweite Branch Protection Rule mit:
- **Branch name pattern**: `develop`
- Gleiche Einstellungen wie für `main`

## Empfohlene Konfiguration für dieses Projekt

### Main Branch (`main`)

```
Branch name pattern: main

✓ Require a pull request before merging
  ✓ Require approvals: 1
  ✓ Dismiss stale pull request approvals when new commits are pushed
  ✓ Require review from Code Owners

✓ Require status checks to pass before merging
  ✓ Require branches to be up to date before merging

✓ Require conversation resolution before merging

✓ Do not allow bypassing the above settings

✓ Restrict who can push to matching branches
  → Nur Repository-Owner/Maintainer
```

### Develop Branch (`develop`) - Falls verwendet

```
Branch name pattern: develop

✓ Require a pull request before merging
  ✓ Require approvals: 1

✓ Require status checks to pass before merging

✓ Require conversation resolution before merging

✗ Do not allow bypassing (etwas lockerer für Development)
```

## CODEOWNERS einrichten

Die `.github/CODEOWNERS` Datei definiert, wer für welche Dateien verantwortlich ist und automatisch als Reviewer hinzugefügt wird.

**Siehe**: [`.github/CODEOWNERS`](CODEOWNERS)

## GitHub Actions für automatische Checks

Optional kannst du GitHub Actions einrichten, die bei jedem PR automatisch laufen:

- Linting (ESLint, Ruff, Checkstyle)
- Tests (Jest, pytest, JUnit)
- Build-Checks
- Security-Scans

**Siehe**: [`.github/workflows/`](workflows/)

## Für Studierende: Was bedeutet das?

### Was ändert sich?

❌ **Nicht mehr möglich**:
```bash
git push origin main              # Fehler: Branch geschützt
```

✅ **Stattdessen**:
```bash
# 1. Branch erstellen
git checkout -b feature/mein-feature

# 2. Änderungen committen
git add .
git commit -m "✨ feat: Mein neues Feature"

# 3. Branch pushen
git push origin feature/mein-feature

# 4. Pull Request auf GitHub erstellen
# 5. Review abwarten
# 6. Nach Approval: Merge via GitHub
```

### Workflow mit Branch Protection

```
Lokale Änderungen
       ↓
Feature-Branch erstellen
       ↓
Commits erstellen
       ↓
Push zu origin/feature-branch
       ↓
Pull Request erstellen auf GitHub
       ↓
CI/CD-Tests laufen automatisch
       ↓
Code-Review anfordern
       ↓
Review-Kommentare bearbeiten
       ↓
Approval erhalten
       ↓
Merge via GitHub
       ↓
Feature-Branch löschen
```

## Troubleshooting

### Problem: "You're not allowed to push to this branch"

**Lösung**: Du versuchst direkt auf `main` zu pushen. Erstelle einen Feature-Branch:
```bash
git checkout -b feature/my-fix
git push origin feature/my-fix
# Dann PR auf GitHub erstellen
```

### Problem: "Required status checks must pass"

**Lösung**: Warte bis CI/CD-Tests durchgelaufen sind. Bei Fehlern:
1. Logs ansehen
2. Fehler lokal fixen
3. Neuen Commit pushen
4. Tests laufen automatisch erneut

### Problem: "Review required"

**Lösung**:
1. Frage einen Reviewer um Review
2. Warte auf Approval
3. Bearbeite ggf. Kommentare
4. Nach Approval: Merge-Button wird aktiviert

## Alternative: GitLab

Für GitLab (falls verwendet):

1. **Settings** → **Repository** → **Protected Branches**
2. Branch auswählen: `main`
3. **Allowed to merge**: Maintainer
4. **Allowed to push**: No one
5. **Require approval from code owners**: ✓
6. **Save changes**

## Alternative: Bitbucket

Für Bitbucket (falls verwendet):

1. **Repository settings** → **Branch permissions**
2. **Add a branch permission**
3. Branch: `main`
4. Type: **Prevent all changes**
5. Exceptions: **Pull requests only**
6. **Save**

## Weitere Ressourcen

- [GitHub Branch Protection Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [CODEOWNERS Syntax](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub Actions](https://docs.github.com/en/actions)

---

**Status**: ⚠️ Branch Protection muss manuell auf GitHub eingerichtet werden (siehe oben)
