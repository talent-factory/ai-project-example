# Contributing to AI Project Example

Vielen Dank für dein Interesse, zu diesem Projekt beizutragen! Diese Anleitung hilft dir, erfolgreich beizutragen.

## Code of Conduct

Wir verpflichten uns, ein respektvolles und inklusives Umfeld zu schaffen. Bitte sei höflich und konstruktiv in allen Interaktionen.

## Wie kann ich beitragen?

### 1. Bugs melden

Gefunden einen Bug? Erstelle ein Issue mit:
- **Beschreibung**: Was ist das Problem?
- **Reproduktion**: Schritte zum Nachstellen
- **Erwartetes Verhalten**: Was sollte passieren?
- **Aktuelles Verhalten**: Was passiert tatsächlich?
- **Umgebung**: OS, Claude Code Version, etc.

### 2. Features vorschlagen

Idee für ein neues Feature? Erstelle ein Issue mit:
- **Problem**: Welches Problem löst das Feature?
- **Lösung**: Wie würde die Lösung aussehen?
- **Alternativen**: Welche anderen Lösungen gibt es?
- **Zusätzlicher Kontext**: Screenshots, Mockups, etc.

### 3. Code beitragen

#### Workflow

1. **Fork** das Repository
2. **Clone** deinen Fork:
   ```bash
   git clone https://github.com/dein-username/ai-project-example.git
   cd ai-project-example
   ```
3. **Branch erstellen**:
   ```bash
   git checkout -b feature/dein-feature-name
   ```
4. **Änderungen vornehmen** und testen
5. **Commit** mit aussagekräftiger Nachricht:
   ```bash
   git commit -m "✨ feat: Füge neues Feature hinzu"
   ```
6. **Push** zu deinem Fork:
   ```bash
   git push origin feature/dein-feature-name
   ```
7. **Pull Request** erstellen

#### Commit-Message-Format

Wir verwenden Emoji Conventional Commits (Deutsch):

```
<emoji> <type>: <Beschreibung>

[Optional: Detaillierte Beschreibung]

[Optional: Footer mit Issue-Referenzen]
```

**Typen**:
- ✨ `feat`: Neue Funktionalität
- 🐛 `fix`: Fehlerbehebung
- 📚 `docs`: Dokumentation
- 💎 `style`: Formatierung
- ♻️ `refactor`: Code-Umstrukturierung
- ⚡ `perf`: Performance-Verbesserung
- 🧪 `test`: Tests
- 🔧 `chore`: Wartung, Build, Tools

**Beispiele**:
```
✨ feat: Füge Dark Mode Toggle hinzu
🐛 fix: Behebe Memory Leak in WebSocket-Verbindungen
📚 docs: Aktualisiere README mit neuen Commands
```

### 4. Dokumentation verbessern

Dokumentations-Beiträge sind sehr willkommen:
- README.md verbessern
- Command-Dokumentation erweitern
- Beispiele hinzufügen
- Typos korrigieren

## Development-Setup

### Voraussetzungen

- **Claude Code** installiert
- **Git** installiert
- **Node.js** (für Scripts)
- **Python 3.9+** (für Python-Scripts)

### Installation

```bash
# Repository clonen
git clone https://github.com/your-org/ai-project-example.git
cd ai-project-example

# Commands sind bereits im Repo
# Keine zusätzliche Installation nötig
```

## Code-Richtlinien

### Allgemein

- **Sprache**: Deutsch für Dokumentation, Englisch für Code/technische Begriffe
- **Formatierung**: Konsistent mit bestehendem Code
- **Kommentare**: Klare, prägnante Erklärungen
- **Tests**: Neue Features brauchen Tests

### Commands

Commands sollten:
- **Progressive Disclosure** verwenden (Haupt-Command + Detail-Dateien)
- **YAML-Frontmatter** haben (description, category, allowed-tools)
- **Dokumentation** mit Beispielen enthalten
- **Best Practices** dokumentieren

**Beispiel-Struktur**:
```
.claude/commands/
├── develop/
│   ├── new-command.md          # Haupt-Command (50-250 Zeilen)
│   └── new-command/            # Detail-Ordner
│       ├── best-practices.md
│       ├── examples.md
│       └── troubleshooting.md
```

### Agents

Agents sollten:
- **YAML-Frontmatter** haben (name, description, category, color, tools)
- **Color-Attribut** aus erlaubten Farben wählen:
  - `blue`: Code/Development
  - `green`: Testing/Validation
  - `red`: Security/Critical
  - `yellow`: Documentation
  - `purple`: Research/Analysis
  - `orange`: Build/Deployment
  - `cyan`: Data/Database
  - `magenta`: UI/UX
- **Klare Struktur** folgen (Rolle, Aktivierung, Prozess, Output)

**Validierung**:
```bash
/develop:check-agents .claude/agents/your-agent.md
```

### Dokumentation

- **Markdown-Format**: CommonMark-Spezifikation
- **Klare Struktur**: Überschriften, Listen, Code-Blöcke
- **Beispiele**: Konkrete, lauffähige Beispiele
- **Deutsche Sprache**: Technische Begriffe auf Englisch erlaubt

## Pull Request Checkliste

Bevor du einen PR erstellst:

- [ ] Code funktioniert und ist getestet
- [ ] Commit-Messages folgen dem Format
- [ ] Dokumentation ist aktualisiert
- [ ] Keine Secrets/API-Keys im Code
- [ ] `.gitignore` ist korrekt
- [ ] PR-Beschreibung ist klar und vollständig

## PR-Template

```markdown
## Beschreibung

[Beschreibe was dieser PR macht]

## Typ der Änderung

- [ ] Bug-Fix
- [ ] Neues Feature
- [ ] Breaking Change
- [ ] Dokumentation

## Wie wurde getestet?

[Beschreibe deine Tests]

## Checkliste

- [ ] Code folgt den Projekt-Richtlinien
- [ ] Tests sind vorhanden und laufen durch
- [ ] Dokumentation ist aktualisiert
- [ ] Commit-Messages folgen dem Format

## Screenshots (falls UI-Änderungen)

[Screenshots einfügen]

## Zusätzlicher Kontext

[Weitere Informationen]
```

## Testing

### Commands testen

```bash
# Command ausführen und Output prüfen
/your-command arg1 arg2

# Bei Fehlern: Logs prüfen
# Claude Code zeigt Fehler direkt an
```

### Agents testen

```bash
# Agent-Validierung
/develop:check-agents .claude/agents/your-agent.md

# Agent in Task nutzen
/develop:implement-fs-task task-xxx
# Prüfen ob Agent korrekt vorgeschlagen wird
```

### Dokumentation testen

```bash
# Markdown-Links prüfen
# Stelle sicher dass alle relativen Links funktionieren

# Command-Dokumentation testen
/develop:check-commands .claude/commands/your-command.md
```

## Release-Prozess

Releases werden von Maintainern erstellt:

1. Version-Bump in relevanten Dateien
2. Changelog aktualisieren
3. Git-Tag erstellen
4. GitHub Release erstellen

## Fragen?

Bei Fragen:
1. Schaue in die bestehende Dokumentation
2. Suche nach existierenden Issues
3. Erstelle ein neues Issue mit dem Label `question`

## Lizenz

Indem du zu diesem Projekt beiträgst, stimmst du zu, dass deine Beiträge unter der MIT-Lizenz lizenziert werden.

---

**Vielen Dank für deinen Beitrag! 🎉**
