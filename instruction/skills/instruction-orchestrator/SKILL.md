---
name: Instruction Orchestrator
description: This skill should be used when the user invokes "/instruction" to consolidate all CLAUDE.md files. Provides the complete deterministic workflow for orchestrating parallel sub-agents that extract and merge documentation index files into a canonical root CLAUDE.md.
version: 1.0.0
---

# CLAUDE.md Konsolidierungs-Workflow

Orchestriere die deterministische Konsolidierung aller CLAUDE.md-Dateien in eine kanonische Root-CLAUDE.md.

## Übersicht

Dieser Skill steuert:
1. Rekursive Bestandsaufnahme aller CLAUDE.md-Dateien
2. Partitionierung in 10er-Gruppen
3. Parallele Sub-Agent-Spawning mit Two-Part-Prompts
4. Ergebnis-Sammlung und Verifikation
5. Konflikt-Erkennung und Konsolidierung

## KRITISCHE REGEL

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║  ⚠️  DU (Hauptagent) DARFST KEINE CLAUDE.md-DATEIEN SELBST LESEN!  ⚠️        ║
║                                                                               ║
║  Die Verarbeitung erfolgt AUSSCHLIESSLICH durch Sub-Agents.                   ║
║  Du orchestrierst nur - du liest/analysierst nicht.                           ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## Workflow-Schritte

### STEP 1: Bestehende Root-CLAUDE.md löschen

```bash
rm -f ./CLAUDE.md
```

Lösche die bestehende CLAUDE.md im aktuellen Verzeichnis bevor du fortfährst.

### STEP 2: Rekursive Bestandsaufnahme

Finde ALLE CLAUDE.md-Dateien rekursiv mit stabiler, alphabetischer Sortierung:

```bash
find . -name "CLAUDE.md" -type f 2>/dev/null | sort
```

Zähle die Gesamtanzahl:

```bash
find . -name "CLAUDE.md" -type f 2>/dev/null | wc -l
```

Speichere die vollständige, sortierte Liste. Die Reihenfolge MUSS deterministisch sein.

### STEP 3: Prüfung auf leere Ergebnisse

Falls keine CLAUDE.md-Dateien gefunden:

```
❌ Keine CLAUDE.md-Dateien gefunden.
Verzeichnis: [aktuelles Verzeichnis]
```

→ Workflow beenden.

### STEP 4: Partitionierung in 10er-Gruppen

Berechnung:
- N = Gesamtanzahl der Dateien
- Gruppen = ceil(N / 10)
- Jede Gruppe erhält exakt 10 Dateien
- NUR die letzte Gruppe darf weniger als 10 haben

**Beispiel für N=37:**

| Gruppe | Dateien | Anzahl |
|--------|---------|--------|
| 1 | 1-10 | 10 |
| 2 | 11-20 | 10 |
| 3 | 21-30 | 10 |
| 4 | 31-37 | 7 |

### STEP 5: Sub-Agents parallel spawnen

Für JEDE Gruppe verwende das Task-Tool:

```javascript
Task({
  subagent_type: "general-purpose",
  model: "haiku",
  prompt: [Two-Part-Prompt - siehe unten]
})
```

**WICHTIG**: Starte ALLE Task-Aufrufe in einer EINZIGEN Nachricht für parallele Ausführung!

---

## Two-Part Prompt Template

Der Prompt für jeden Sub-Agent besteht aus zwei Teilen:

### Teil 1: Skill-Referenz (Statisch - für alle gleich)

```
INSTRUCTIONS:
Du bist ein Extraktions-Sub-Agent. Deine EINZIGE Aufgabe ist das Lesen und
Extrahieren von Informationen aus CLAUDE.md-Dateien.

Lies zuerst den Extraktions-Skill mit den detaillierten Anweisungen:

Read file: ${CLAUDE_PLUGIN_ROOT}/skills/claude-extractor/SKILL.md

Befolge ALLE Anweisungen in diesem Skill exakt.

REGELN:
1. Verwende NUR das Read-Tool
2. Lies NUR die dir zugewiesenen Dateien
3. Erfinde KEINE Inhalte
4. Gib IMMER den Quelldateipfad an
5. Befolge das Output-Format aus dem Skill exakt
```

### Teil 2: Spezifische Dateipfade (Dynamisch - variiert pro Sub-Agent)

```
---

DEINE ZUGEWIESENEN DATEIEN (Gruppe X von Y):

1. [Pfad 1]
2. [Pfad 2]
3. [Pfad 3]
...
10. [Pfad 10]

Lies JEDE dieser Dateien mit dem Read-Tool und extrahiere die Informationen
gemäß dem Skill. Gib das Ergebnis im vorgeschriebenen Format zurück:

=== EXTRACTION: [Dateipfad] ===
DOC_PATH: [...]
GENERATED: [...]
STRUCTURE:
[...]
=== END EXTRACTION ===
```

### Vollständiges Prompt-Template

```
INSTRUCTIONS:
Du bist ein Extraktions-Sub-Agent. Deine EINZIGE Aufgabe ist das Lesen und
Extrahieren von Informationen aus CLAUDE.md-Dateien.

Lies zuerst den Extraktions-Skill mit den detaillierten Anweisungen:

Read file: ${CLAUDE_PLUGIN_ROOT}/skills/claude-extractor/SKILL.md

Befolge ALLE Anweisungen in diesem Skill exakt.

REGELN:
1. Verwende NUR das Read-Tool
2. Lies NUR die dir zugewiesenen Dateien
3. Erfinde KEINE Inhalte
4. Gib IMMER den Quelldateipfad an
5. Befolge das Output-Format aus dem Skill exakt

---

DEINE ZUGEWIESENEN DATEIEN (Gruppe {X} von {Y}):

1. {PATH_1}
2. {PATH_2}
...
{N}. {PATH_N}

Lies JEDE dieser Dateien mit dem Read-Tool und extrahiere die Informationen
gemäß dem Skill. Gib das Ergebnis im vorgeschriebenen Format zurück.
```

---

## STEP 6: Ergebnisse sammeln

Warte auf ALLE Sub-Agent-Ergebnisse bevor du fortfährst.

Sammle alle `=== EXTRACTION: ... ===` Blöcke aus den Rückgaben.

### Extraktions-Parsing

Jeder Block hat diese Struktur:

```
=== EXTRACTION: [Quelldateipfad] ===
DOC_PATH: [Dokumentationspfad]
GENERATED: [Timestamp]
STRUCTURE:
[Verzeichnisstruktur]
=== END EXTRACTION ===
```

Parse jeden Block und extrahiere:
- `source_file`: Der Quelldateipfad
- `doc_path`: Der DOC_PATH Wert
- `generated`: Der GENERATED Timestamp
- `structure`: Die komplette STRUCTURE

---

## STEP 7: Konflikt-Erkennung

Prüfe auf folgende Konflikttypen:

### 7.1 Duplikate

Gleiche `DOC_PATH` in verschiedenen Quelldateien.

**Beispiel:**
```
./backend/CLAUDE.md → DOC_PATH: /Users/home/project/api
./services/CLAUDE.md → DOC_PATH: /Users/home/project/api
```

→ **KONFLIKT**: Zwei Quellen dokumentieren denselben Pfad.

### 7.2 Inkonsistenzen

Widersprüchliche Informationen für gleiche Verzeichnisse/Dateien.

→ Markiere und liste alle erkannten Inkonsistenzen.

---

## STEP 8: Konsolidierung erstellen

Erstelle die kanonische Root-CLAUDE.md mit diesem Aufbau:

```markdown
# Consolidated Documentation Index

Generated: {AKTUELLER_ISO_TIMESTAMP}
Source files: {ANZAHL_VERARBEITETER_DATEIEN}
Processed by: {ANZAHL_SUBAGENTS} subagents

---

## {DOC_PATH_1}
<!-- Source: {QUELLDATEI_1} -->

{STRUCTURE_1}

---

## {DOC_PATH_2}
<!-- Source: {QUELLDATEI_2} -->

{STRUCTURE_2}

---

[... weitere Sektionen ...]

---

## Metadata

| Property | Value |
|----------|-------|
| Generated | {Timestamp} |
| Source files | {Anzahl} |
| Subagents used | {Anzahl} |
| Processing order | Alphabetically sorted |

---

## Conflicts

{Falls Konflikte erkannt wurden:}

- **[CONFLICT]** Duplicate DOC_PATH detected
  - Source 1: {Pfad} → {Was dort steht}
  - Source 2: {Pfad} → {Was dort steht}
  - Resolution: Manual review required

{Falls keine Konflikte:}

No conflicts detected.
```

---

## STEP 9: Output schreiben

Schreibe die konsolidierte CLAUDE.md in das aktuelle Verzeichnis:

```
./CLAUDE.md
```

Verwende das Write-Tool.

---

## STEP 10: Abschlussbericht

Zeige dem User einen Abschlussbericht:

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    CLAUDE.md KONSOLIDIERUNG ABGESCHLOSSEN                    ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ Verzeichnis: {pwd}                                                           ║
║ Verarbeitete Dateien: {N}                                                    ║
║ Verwendete Subagents: {Anzahl}                                               ║
║ Erkannte Konflikte: {Anzahl}                                                 ║
╠══════════════════════════════════════════════════════════════════════════════╣
║ Output: ./CLAUDE.md                                                          ║
╚══════════════════════════════════════════════════════════════════════════════╝

{Falls Konflikte vorhanden:}
⚠️  Es wurden Konflikte erkannt. Bitte prüfe den "Conflicts" Abschnitt.
```

---

## Architektur-Diagramm

```
/instruction
    │
    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Hauptagent (Orchestrator)                                          │
│  ═════════════════════════                                          │
│  1. rm -f ./CLAUDE.md                                               │
│  2. find . -name "CLAUDE.md" | sort                                 │
│  3. Partitioniere in 10er-Gruppen                                   │
│  4. Für jede Gruppe: Task(haiku, two-part-prompt)                   │
│  5. Sammle alle Ergebnisse                                          │
│  6. Erkenne Konflikte                                               │
│  7. Konsolidiere → ./CLAUDE.md                                      │
│  8. Zeige Abschlussbericht                                          │
└─────────────────────────────────────────────────────────────────────┘
         │
         │  Parallele Task-Aufrufe (eine Nachricht)
         │
         ├── Task(haiku) → Gruppe 1: Dateien 1-10
         ├── Task(haiku) → Gruppe 2: Dateien 11-20
         ├── Task(haiku) → Gruppe 3: Dateien 21-30
         └── Task(haiku) → Gruppe 4: Dateien 31-37
                     │
                     │ Jeder Sub-Agent:
                     │ 1. Liest Skill-Datei
                     │ 2. Liest zugewiesene CLAUDE.md-Dateien
                     │ 3. Extrahiert nach Skill-Regeln
                     │ 4. Gibt strukturierte Ergebnisse zurück
                     │
                     ▼
              Ergebnisse sammeln
                     │
                     ▼
         Konflikte erkennen + Konsolidieren
                     │
                     ▼
              ./CLAUDE.md erstellen
```

---

## Fehlerbehandlung

### Sub-Agent-Fehler

Falls ein Sub-Agent fehlschlägt:
- Dokumentiere den Fehler in der Output-Datei
- Fahre mit den anderen Sub-Agents fort
- Markiere fehlende Extraktionen

### Schreibfehler

Falls das Schreiben fehlschlägt:
- Klare Fehlermeldung ausgeben
- Keine partielle Datei hinterlassen

---

## Vorteile des Two-Part-Prompts

1. **Wartbarkeit**: Skill-Datei einmal aktualisieren, alle Sub-Agents profitieren
2. **Konsistenz**: Alle Sub-Agents befolgen identische Regeln
3. **Effizienz**: Kürzere Prompts, Skill wird nur bei Bedarf geladen
4. **Klarheit**: Trennung zwischen "Wie" (Skill) und "Was" (Dateipfade)
