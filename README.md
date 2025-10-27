# DigiStars
Ein intelligenter n8n-Workflow zur automatisierten Recherche, Kategorisierung und Verwaltung digitaler Tools für Kinder und Jugendliche in Deutschland.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![n8n](https://img.shields.io/badge/n8n-Workflow-ff6d5a)](https://n8n.io)

## 📖 Dokumentation

- **[Schnellstart](docs/QUICKSTART.md)** - In 15 Minuten loslegen
- **[Setup-Anleitung](docs/SETUP.md)** - Vollständige Einrichtung
- **[Workflow-Guide](docs/WORKFLOW-GUIDE.md)** - Detaillierte Funktionsweise

## 📋 Inhaltsverzeichnis

- [Überblick](#-überblick)
- [Features](#-features)
- [Systemarchitektur](#-systemarchitektur)
- [Voraussetzungen](#-voraussetzungen)
- [Installation](#-installation)
- [Konfiguration](#konfiguration)
- [Verwendung](#-verwendung)
- [Workflow-Details](#-workflow-details)
- [Troubleshooting](#-troubleshooting)
- [Lizenz](#-lizenz)

## 🎯 Überblick

DigiStars ist ein automatisierter Workflow zur Identifizierung, Bewertung und Katalogisierung digitaler Innovationen für Kinder und Jugendliche. Der Workflow kombiniert Web-Scraping, KI-gestützte Kategorisierung und strukturierte Datenspeicherung.

### Hauptziele

- **Automatische Recherche** digitaler Tools über Suchmaschinen
- **Intelligente Kategorisierung** durch KI (Tool, Liste, Irrelevant)
- **Duplikatsprüfung** auf mehreren Ebenen
- **Strukturierte Datenverwaltung** in Google Sheets
- **Skalierbare Verarbeitung** mit Batch-Processing

## Screenshot von v0.3
<img width="1936" height="1045" alt="image" src="https://github.com/user-attachments/assets/61409ec0-22e6-4135-a5cf-36a40ae1f0e1" />

## ✨ Features

### 🔍 Multi-Source Search
- **Google Custom Search API** - Präzise, deutschsprachige Suche
- **EXA AI Search** - Intelligente semantische Suche mit bis zu 100 Ergebnissen
- **Google Sheets Integration** - Verarbeitung bereits erfasster URLs

### 🤖 KI-gestützte Analyse
- **Automatische Kategorisierung** in Tool, Liste oder Irrelevant
- **Strukturierte Datenextraktion** (Organisation, Zielgruppe, Kosten, etc.)
- **Link-Extraktion** aus Listen-Webseiten
- **Token-optimiertes Processing** (max. 20k Tokens)

### 🔄 Intelligente Duplikatsprüfung
1. **Exakte URL-Prüfung** - Verhindert doppelte Einträge
2. **Base-URL Tool-Check** - Erkennt bereits erfasste Domains als Tools
3. **Deduplizierungs-Workflow** - Separate Workflow-Integration

### 📊 Datenmanagement
- **Google Sheets** als zentrale Datenbank
- **Statusverwaltung** (Neu erfasst, Prüfen, Geprüft-KI, Fehler)
- **Zeitstempel** für Crawl-Datum und Updates
- **Fehlerbehandlung** mit detaillierter Protokollierung

### 🛠 Web-Scraping
- **crawl4ai** - Lokaler Scraping-Service via Docker
- **FireCrawl** - Cloud-Backup für komplexe Seiten
- **PDF-Support** - Automatische PDF-Erkennung und -Verarbeitung
- **Markdown-Konvertierung** - Strukturierte Inhaltsverarbeitung

## 🏗 Systemarchitektur

```
┌─────────────────┐
│   Input Layer   │
├─────────────────┤
│ • Google Search │
│ • EXA AI        │
│ • Google Sheets │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Deduplication   │
├─────────────────┤
│ • URL Check     │
│ • Domain Check  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Web Scraping   │
├─────────────────┤
│ • crawl4ai      │
│ • FireCrawl     │
│ • PDF Handler   │
└────────┬────────┘
         │
         ▼
┌─────────────────-┐
│  AI Analysis     │
├─────────────────-┤
│ • Kategorisierung│
│ • Extraktion     │
│ • o4-mini        │
│ • gpt-4.1-nano   │
└────────┬───────-─┘
         │
         ▼
┌─────────────────┐
│ Data Storage    │
├─────────────────┤
│ • Google Sheets │
│ • Status Update │
│ • Validation    │
└─────────────────┘
```

## 📦 Voraussetzungen

### Software
- **n8n** (v1.0+)
- **Docker** (für crawl4ai)
- **Node.js** (v18+)

### Services & APIs
- Google Custom Search API Key + Search Engine ID
- EXA AI API Key
- OpenAI API Key (gpt-4.1-nano, o4-mini)
- FireCrawl API Key (optional)
- Google Sheets OAuth2 Credentials

### Docker Container
```bash
# crawl4ai Service
docker run -d \
  --name crawl4ai \
  -p 11235:11235 \
  crawl4ai/server:latest
```

## 🚀 Installation

### 1. Repository klonen
```bash
git clone https://github.com/ChildResourceRadar/DigiStars.git
cd DigiStars
```

### 2. n8n Workflow importieren
1. Öffne n8n im Browser
2. Navigiere zu **Workflows** → **Import from File**
3. Wähle `workflows/digistars-n8n-scraper-categorizer.json`
4. Bestätige den Import

### 3. Credentials einrichten

#### Google Custom Search
```
Node: "Google Suche"
- Parameter: query.key = YOUR_GOOGLE_API_KEY
- Parameter: query.cx = YOUR_SEARCH_ENGINE_ID
```

#### EXA AI
```
Node: "EXA Suche"
- Header: x-api-key = YOUR_EXA_API_KEY
```

#### OpenAI
```
Credentials: "OpenAI Account"
- API Key eingeben
```

#### Google Sheets
```
Credentials: "Google Sheets OAuth2"
- OAuth2 Flow durchlaufen
- Scopes: spreadsheets, drive.file
```

#### FireCrawl (optional)
```
Node: "FireCrawl Scraper"
- Credentials: API Key eingeben
```

### 4. Google Sheet vorbereiten

Erstelle ein Google Sheet mit folgenden Spalten:
```
Name | URL | Status | Score | Art der Webseite | Kurzbeschreibung | 
Zielgruppe | Plattform | Altersgruppe | Verfügbare Sprachen | 
Kosten | Kategorie | Organisation | Ersteller | Crawl-Datum | 
Letztes Update | row_number
```

Sheet-ID im Workflow aktualisieren:
```
Nodes: "Neu erfasste abrufen", "Update", "Neu", etc.
Document ID: YOUR_SHEET_ID
```

### 5. crawl4ai Docker Container starten
```bash
docker pull crawl4ai/server:latest
docker run -d --name crawl4ai -p 11235:11235 crawl4ai/server:latest
```

## Konfiguration

### Workflow-Modi

#### 1. **Recherche-Modus** (EXA/Google)
Neue Suche über Suchmaschinen:
```
Input: EXA Suchanfrage oder Google Suche Node
Output: Neue URLs in Google Sheets
```

#### 2. **Sheets-Modus**
Verarbeitung bereits erfasster URLs:
```
Input: Google Sheets Filter "Status = Neu erfasst"
Output: Aktualisierte Einträge mit KI-Analyse
```

### Wichtige Parameter

#### Batch Size
```javascript
Node: "Loop Over Items"
batchSize: 10  // Anzahl gleichzeitiger Crawls
```

#### Token Limit
```javascript
Node: "Auf 20k Tokens begrenzen"
maxTokens: 20000  // Maximale Token pro Analyse
```

#### EXA Search Results
```javascript
Node: "EXA Suchanfrage"
num_results: 50  // Max 100
```

### Status-Definitionen

| Status | Bedeutung |
|--------|-----------|
| `Neu erfasst` | URL wurde hinzugefügt, noch nicht verarbeitet |
| `Prüfen` | Als Tool kategorisiert, manuelle Prüfung erforderlich |
| `Geprüft-KI` | Als Liste/Irrelevant kategorisiert, automatisch verarbeitet |
| `Fehler` | Fehler beim Crawling oder Processing |

## 📖 Verwendung

### Manuelle Ausführung

1. **n8n Workflow öffnen**
2. **Modus wählen:**
   - Für neue Suche: EXA Suchanfrage Node konfigurieren
   - Für Sheet-Verarbeitung: "When clicking 'Execute workflow'" triggern
3. **Execute Workflow** klicken
4. **Ergebnisse** in Google Sheets prüfen

### Automatisierung

#### Scheduled Trigger hinzufügen
```
1. "When clicking 'Execute workflow'" Node ersetzen
2. Schedule Trigger Node hinzufügen
3. Cron Expression: z.B. "0 2 * * *" (täglich 2 Uhr)
```

#### Webhook Trigger
```
1. Webhook Trigger Node hinzufügen
2. Webhook URL generieren
3. Von externem Service aufrufen
```

## 🔧 Workflow-Details

### Eingabe-Nodes

#### Google Suche
```json
{
  "url": "https://www.googleapis.com/customsearch/v1",
  "parameters": {
    "key": "YOUR_GOOGLE_API_KEY",
    "q": "Cyber-Mobbing App Jugendliche Deutschland",
    "cx": "YOUR_SEARCH_ENGINE_ID",
    "num": "5",
    "gl": "de",
    "lr": "lang_de"
  }
}
```

#### EXA Suche
```json
{
  "query": "Deutschland Jugend digitale Hilfe neu",
  "num_results": 50,
  "excludeDomains": ["bereits-erfasste-domains.de"]
}
```

### Scraping-Nodes

#### crawl4ai
```json
{
  "url": "http://host.docker.internal:11235/md",
  "body": {
    "url": "target-url.de",
    "f": "fit",
    "q": null,
    "c": "0"
  }
}
```

#### FireCrawl (Backup)
```json
{
  "url": "target-url.de",
  "enableDebugLogs": true,
  "onError": "continueRegularOutput"
}
```

### KI-Kategorisierung

Das System verwendet einen mehrstufigen Prompt:

```
Kategorien:
- Tool: Offizielle Seite eines digitalen Tools
- Liste: Beschreibung/Empfehlung von Tools
- Irrelevant: Nicht relevant für DigiStars

Output Format:
{
  "reasoning": "Analyse-Text",
  "tools": [{"name": "Tool-Name", "url": "URL"}],
  "category": "Tool|Liste|Irrelevant"
}
```

### Duplikatsprüfung

#### Ebene 1: Exakte URL
```javascript
Filter: URL = eingehende_url
Ergebnis: Exakte Duplikate werden entfernt
```

#### Ebene 2: Base-URL Tool-Check
```javascript
Filter: URL = base_url + "/" AND Score = "Tool"
Ergebnis: Subpages bereits erfasster Tools werden entfernt
```

### Datenextraktion

Extrahierte Felder pro Tool:
- Art der Webseite
- Kurzbeschreibung
- Zielgruppe
- Plattform (Web, Mobile App, Desktop)
- Altersgruppe
- Verfügbare Sprachen
- Kosten (Kostenlos, Freemium, Kostenpflichtig)
- Kategorie (z.B. Suizidprävention, Cybermobbing)
- Organisation

## 🐛 Troubleshooting

### Häufige Probleme

#### crawl4ai nicht erreichbar
```bash
# Prüfe ob Container läuft
docker ps | grep crawl4ai

# Neu starten
docker restart crawl4ai

# Logs prüfen
docker logs crawl4ai
```

#### Google Sheets Berechtigungen
```
Fehler: "Insufficient permissions"
Lösung:
1. Google Cloud Console öffnen
2. Projekt auswählen
3. APIs & Services → Credentials
4. OAuth2 Client bearbeiten
5. Scopes prüfen: spreadsheets, drive.file
```

#### OpenAI Rate Limits
```
Fehler: "Rate limit exceeded"
Lösung:
1. Batch Size reduzieren (Loop Over Items: 5 statt 10)
2. Delay zwischen Requests hinzufügen
3. OpenAI Tier upgraden
```

#### EXA Search keine Ergebnisse
```
Problem: excludeDomains filtert zu viel
Lösung:
1. "Existierende URLs" Node deaktivieren
2. Manuelle Domain-Liste verwenden
3. Base-URL Normalisierung prüfen
```

### Logs & Debugging

#### n8n Execution Logs
```
1. Workflow öffnen
2. Rechts oben: "Executions"
3. Fehlgeschlagene Execution auswählen
4. Node-by-Node Daten prüfen
```

#### Enable Debug Modus
```
FireCrawl Node: "enableDebugLogs": true
HTTP Request Nodes: Options → Debug = true
```

### Performance-Optimierung

#### Große Datenmengen
```javascript
// Loop Over Items Batch Size anpassen
batchSize: 5  // Kleinere Batches = stabiler

// Token Limit für lange Seiten
maxTokens: 10000  // Reduzieren bei Timeouts
```

#### Parallele Verarbeitung
```
n8n Settings → Executions:
- Timeout: 3600 (1 Stunde)
- Max Payload: 16 MB
```

## 📊 Output-Beispiele

### Tool-Kategorisierung
```json
{
  "name": "FIDEO",
  "URL": "https://fideo.de/",
  "output": {
    "category": "Tool",
    "reasoning": "Offizielle Seite eines digitalen Tools für Jugendliche mit Depressionen",
    "Art der Webseite": "Informationsportal",
    "Kurzbeschreibung": "Online-Portal mit Hilfsangeboten bei Depressionen",
    "Zielgruppe": "Jugendliche und junge Erwachsene",
    "Plattform": "Web",
    "Altersgruppe": "14-25 Jahre",
    "Verfügbare Sprachen": "Deutsch",
    "Kosten": "Kostenlos",
    "Kategorie": "Mentale Gesundheit",
    "Organisation": "Diskussionsforum Depression e.V."
  }
}
```

### Listen-Kategorisierung
```json
{
  "name": "Hilfsangebote Übersicht",
  "URL": "https://example.de/artikel",
  "output": {
    "category": "Liste",
    "reasoning": "Artikel beschreibt mehrere Tools, ist aber nicht offizielle Tool-Seite",
    "tools": [
      {"name": "FIDEO", "url": "https://fideo.de"},
      {"name": "Nummer gegen Kummer", "url": null}
    ]
  }
}
```

## 🤝 Beitragen

Verbesserungsvorschläge und Pull Requests sind willkommen!

1. Fork das Repository
2. Feature Branch erstellen (`git checkout -b feature/AmazingFeature`)
3. Änderungen committen (`git commit -m 'Add AmazingFeature'`)
4. Branch pushen (`git push origin feature/AmazingFeature`)
5. Pull Request erstellen

## 📝 Lizenz

Dieses Projekt ist unter der MIT-Lizenz lizenziert - siehe [LICENSE](LICENSE) Datei für Details.

## 👥 Autoren & Danksagungen

**Entwickelt von:** Child Resource Radar e.V.  
**Projektleitung:** Dr. Juliane Petersen  
**Workflow-Entwicklung:** [Manuel Dingemann](captain-ai.de)  
**Website:** [childresourceradar.org](https://childresourceradar.org)

**Danke an:**
- DigiStars Projekt-Team
- n8n Community
- [crawl4ai](https://github.com/unclecode/crawl4ai) Contributors
- FireCrawl Team

## 📞 Support

Bei Fragen oder Problemen:
- **GitHub Issues:** [Issue erstellen](https://github.com/ChildResourceRadar/DigiStars/issues)
- **Workflow-Fragen:** manuel@captain-ai.de
- **Projekt-Kontakt:** info@childresourceradar.org
- **Website:** [childresourceradar.org](https://childresourceradar.org)

---

**Version:** 0.3  
**Letztes Update:** Oktober 2025

## 🌟 Projektkontext

DigiStars ist Teil des Child Resource Radar e.V. Projekts zur Förderung digitaler Innovationen für benachteiligte Kinder und Jugendliche in Deutschland. Das Projekt wird von der Deutschen Fernsehlotterie (Schwerpunkt Digitalisierung) gefördert.

Mehr Informationen: [Projektvorstellung](/mnt/project/Projektvorstellung__DigiStars_.pdf)
