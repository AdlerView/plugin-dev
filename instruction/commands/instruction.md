---
description: Konsolidiere alle CLAUDE.md-Dateien deterministisch in eine kanonische Root-CLAUDE.md
allowed-tools: [Task, Bash, Read, Write]
---

# CLAUDE.md Konsolidierung

Konsolidiere alle CLAUDE.md-Dateien im aktuellen Verzeichnis in eine einzige kanonische Root-CLAUDE.md.

## Ausführung

**SCHRITT 0**: Lies zuerst den Orchestrator-Skill:

```
${CLAUDE_PLUGIN_ROOT}/skills/instruction-orchestrator/SKILL.md
```

Befolge dann den Workflow im Skill exakt:

1. Führe jeden Schritt wie im Skill beschrieben aus
2. Verwende den Two-Part-Prompt für Sub-Agents wie im Skill definiert
3. Gib den Abschlussbericht wie im Skill spezifiziert aus

## Workflow-Übersicht

```
1. rm -f ./CLAUDE.md                    → Bestehende Datei löschen
2. find . -name "CLAUDE.md" | sort      → Alle CLAUDE.md-Dateien finden
3. Partitioniere in 10er-Gruppen        → Max. 10 Dateien pro Sub-Agent
4. Task(haiku) pro Gruppe               → Parallele Verarbeitung
5. Ergebnisse sammeln                   → Warte auf alle Sub-Agents
6. Konflikte erkennen                   → Duplikate/Inkonsistenzen markieren
7. Konsolidieren → ./CLAUDE.md          → Finale Datei erstellen
8. Abschlussbericht anzeigen            → Statistiken + Konflikte
```

## Kritische Regeln

- **KEINE CLAUDE.md-Dateien selbst lesen** - nur Sub-Agents lesen
- **Stabile Reihenfolge** - alphabetisch sortiert, deterministisch
- **10 Dateien pro Gruppe** - nur letzte Gruppe darf weniger haben
- **Parallele Ausführung** - alle Task-Aufrufe in einer Nachricht
