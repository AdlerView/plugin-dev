# Instruction Plugin

Deterministische Konsolidierung aller CLAUDE.md-Dateien in eine kanonische Root-CLAUDE.md.

## Features

- **Rekursive Suche**: Findet alle CLAUDE.md-Dateien im Projektverzeichnis
- **Deterministische Verarbeitung**: Stabile, reproduzierbare alphabetische Reihenfolge
- **Partitionierung**: Verarbeitung in 10er-Gruppen durch dedizierte Subagents
- **Two-Part-Prompt**: Universeller Skill + spezifische Dateipfade pro Subagent
- **Konflikt-Erkennung**: Markiert widersprüchliche Informationen
- **Quellenangaben**: Jede Information mit Herkunftspfad annotiert

## Installation

Das Plugin ist in `~/.claude/plugins/instruction` installiert.

## Verwendung

```
/instruction
```

## Komponenten

### Skills

| Skill | Zweck |
|-------|-------|
| `instruction-orchestrator` | Workflow für Hauptagent: Orchestrierung, Partitionierung, Konsolidierung |
| `claude-extractor` | Regeln für Subagents: Extraktion, Output-Format, Quellenangaben |

### Command

| Command | Zweck |
|---------|-------|
| `/instruction` | Startet den Konsolidierungs-Workflow |

## Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│  Hauptagent (liest instruction-orchestrator Skill)                  │
│  ══════════════════════════════════════════════════                 │
│  1. rm -f ./CLAUDE.md                                               │
│  2. find . -name "CLAUDE.md" | sort                                 │
│  3. Partitioniere in 10er-Gruppen                                   │
│  4. Spawne Subagents (haiku) mit Two-Part-Prompt                    │
│  5. Sammle Ergebnisse                                               │
│  6. Erkenne Konflikte                                               │
│  7. Konsolidiere → ./CLAUDE.md                                      │
└─────────────────────────────────────────────────────────────────────┘
                              │
      ┌───────────────────────┼───────────────────────┐
      ▼                       ▼                       ▼
┌───────────┐           ┌───────────┐           ┌───────────┐
│ Subagent 1│           │ Subagent 2│           │ Subagent N│
│ (haiku)   │           │ (haiku)   │           │ (haiku)   │
├───────────┤           ├───────────┤           ├───────────┤
│ Teil 1:   │           │ Teil 1:   │           │ Teil 1:   │
│ Lies Skill│           │ Lies Skill│           │ Lies Skill│
│ (gleich)  │           │ (gleich)  │           │ (gleich)  │
├───────────┤           ├───────────┤           ├───────────┤
│ Teil 2:   │           │ Teil 2:   │           │ Teil 2:   │
│ Pfade 1-10│           │ Pfade11-20│           │ Pfade rest│
│ (variabel)│           │ (variabel)│           │ (variabel)│
└───────────┘           └───────────┘           └───────────┘
      │                       │                       │
      └───────────────────────┼───────────────────────┘
                              ▼
                    Ergebnisse sammeln
                              │
                              ▼
                  Konflikte + Konsolidierung
                              │
                              ▼
                       ./CLAUDE.md
```

## Two-Part-Prompt Struktur

### Teil 1 (Universell - für alle Subagents gleich)

```
Lies den Extraktions-Skill:
/Users/home/.claude/plugins/instruction/skills/claude-extractor/SKILL.md
```

### Teil 2 (Variabel - spezifische Dateipfade)

```
Deine zugewiesenen Dateien (Gruppe X von Y):
1. ./path/to/file1/CLAUDE.md
2. ./path/to/file2/CLAUDE.md
...
```

## Output-Format

```markdown
# Consolidated Documentation Index

Generated: 2025-12-28T15:00:00Z
Source files: 37
Processed by: 4 subagents

---

## /path/to/docs
<!-- Source: ./subdir/CLAUDE.md -->

├── file.md → Description
└── other.md → Another description

---

## Conflicts

No conflicts detected.
```

## Vorteile der Architektur

1. **Wartbarkeit**: Skill-Datei einmal aktualisieren, alle Subagents profitieren
2. **Konsistenz**: Alle Subagents befolgen identische Extraktionsregeln
3. **Determinismus**: Stabile Reihenfolge, reproduzierbare Ergebnisse
4. **Skalierbarkeit**: Parallele Verarbeitung in 10er-Gruppen
5. **Transparenz**: Quellenangaben für jede konsolidierte Information
