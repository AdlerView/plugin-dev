---
name: CLAUDE.md Extractor
description: This skill should be used by subagents when extracting and parsing CLAUDE.md documentation index files. Provides universal extraction rules, output format specification, and source attribution requirements.
version: 1.0.0
---

# CLAUDE.md Extractor Skill

Universelle Extraktionsregeln für die Verarbeitung von CLAUDE.md-Dokumentations-Index-Dateien.

## Aufgabe

Lese die zugewiesenen CLAUDE.md-Dateien, extrahiere strukturierte Informationen und gib eine kompakte Zusammenfassung mit Quellenangaben zurück.

## Input-Format (CLAUDE.md)

Jede CLAUDE.md-Datei folgt diesem Schema:

```markdown
# Documentation: /absolute/path/to/directory

Generated: YYYY-MM-DDTHH:MM:SSZ

## Directory Structure

├── filename.md               → Kurzbeschreibung des Inhalts
├── another-file.md           → Weitere Beschreibung
├── subdirectory/
│   ├── nested-file.md        → Beschreibung der verschachtelten Datei
│   └── another-nested.md     → Weitere verschachtelte Datei
└── last-file.md              → Letzte Datei
```

## Extraktionsregeln

### Pflichtfelder extrahieren

1. **Dokumentationspfad**: Aus der Zeile `# Documentation: <path>`
2. **Generierungszeitpunkt**: Aus der Zeile `Generated: <timestamp>`
3. **Verzeichnisstruktur**: Kompletter Inhalt unter `## Directory Structure`

### Verarbeitungsregeln

1. **Keine Erfindungen**: Extrahiere NUR was tatsächlich in der Datei steht
2. **Vollständigkeit**: Übernimm die GESAMTE Verzeichnisstruktur, kürze nicht
3. **Formaterhaltung**: Behalte die Baumstruktur (├── │ └──) exakt bei
4. **Quellenangabe**: Gib IMMER den Dateipfad an, aus dem extrahiert wurde

## Output-Format

Gib für JEDE verarbeitete Datei ein strukturiertes Ergebnis zurück:

```
=== EXTRACTION: [Dateipfad] ===
DOC_PATH: [Extrahierter Dokumentationspfad]
GENERATED: [Extrahierter Timestamp]
STRUCTURE:
[Komplette Verzeichnisstruktur]
=== END EXTRACTION ===
```

### Beispiel-Output

```
=== EXTRACTION: ./backend/api/CLAUDE.md ===
DOC_PATH: /Users/home/project/backend/api
GENERATED: 2025-12-28T14:30:00Z
STRUCTURE:
├── endpoints.md              → REST API endpoint documentation
├── authentication.md         → OAuth2 and JWT authentication flows
├── rate-limiting.md          → Rate limiting configuration
└── errors/
    ├── error-codes.md        → Standard error code reference
    └── troubleshooting.md    → Common API error solutions
=== END EXTRACTION ===

=== EXTRACTION: ./frontend/CLAUDE.md ===
DOC_PATH: /Users/home/project/frontend
GENERATED: 2025-12-28T15:00:00Z
STRUCTURE:
├── components.md             → React component library
└── styling.md                → CSS and theming guide
=== END EXTRACTION ===
```

## Fehlerbehandlung

### Datei nicht lesbar

```
=== EXTRACTION: [Dateipfad] ===
ERROR: Datei konnte nicht gelesen werden
REASON: [Fehlerbeschreibung]
=== END EXTRACTION ===
```

### Ungültiges Format

```
=== EXTRACTION: [Dateipfad] ===
ERROR: Ungültiges CLAUDE.md Format
REASON: [Was fehlt oder falsch ist]
PARTIAL_CONTENT: [Was extrahiert werden konnte]
=== END EXTRACTION ===
```

## Wichtige Regeln

1. **Strikt nur Lesen**: Verwende ausschließlich das Read-Tool
2. **Keine Modifikationen**: Ändere keine Dateien
3. **Keine Annahmen**: Erfinde keine Inhalte die nicht existieren
4. **Vollständige Extraktion**: Extrahiere ALLES aus jeder zugewiesenen Datei
5. **Klare Trennung**: Trenne Extraktionen verschiedener Dateien deutlich
6. **Quellenangabe**: Jede Information muss ihrem Quelldateipfad zugeordnet sein
