# Workflow Guide - DigiStars Scrape & Crawl

Dieses Dokument erklärt im Detail, wie der Workflow funktioniert und welche Nodes welche Aufgaben haben.

## 📊 Workflow-Übersicht

Der Workflow besteht aus 5 Hauptphasen:

```
Input → Deduplizierung → Scraping → AI-Analyse → Speicherung
```

## 🔍 Phase 1: Input

### Eingabe-Quellen

Der Workflow kann aus 3 verschiedenen Quellen gestartet werden:

#### 1.1 Google Custom Search

**Node:** `Google Suche`

Führt eine Google-Suche aus und liefert bis zu 10 Ergebnisse.

```json
{
  "url": "https://www.googleapis.com/customsearch/v1",
  "parameters": {
    "key": "YOUR_API_KEY",
    "q": "Cyber-Mobbing App Jugendliche Deutschland",
    "cx": "c6dc86504a592425a",
    "num": "5",
    "gl": "de",
    "lr": "lang_de",
    "start": "1"
  }
}
```

**Output Format:**
```json
{
  "items": [
    {
      "title": "Beispiel Tool",
      "link": "https://example.com",
      "snippet": "Kurzbeschreibung..."
    }
  ]
}
```

**Node:** `Split Out - Google`  
Teilt das Array `items` in einzelne Elemente auf.

#### 1.2 EXA AI Search

**Node:** `EXA Suchanfrage`

Semantische Suche mit bis zu 100 Ergebnissen.

```json
{
  "query": "Deutschland Jugend digitale Hilfe neu",
  "num_results": 50,
  "excludeDomains": ["bereits-erfasste-domains.de"]
}
```

**Vorverarbeitung:**
1. **Node:** `Existierende URLs` - Lädt alle URLs aus Google Sheets
2. **Node:** `zu Base-URLs für EXA` - Konvertiert zu Base-URLs mit www
3. **Node:** `Aggregate` - Sammelt URLs in ein Array
4. **Node:** `EXA Suche` - Führt Suche aus mit excludeDomains

**Output Format:**
```json
{
  "results": [
    {
      "id": "https://example.com",
      "title": "Beispiel Tool",
      "text": "Volltext der Seite...",
      "url": "https://example.com"
    }
  ]
}
```

**Node:** `Split Out - EXA`  
Teilt das Array `results` in einzelne Elemente auf und benennt Feld zu `link` um.

#### 1.3 Google Sheets

**Node:** `Neu erfasste abrufen`

Liest URLs mit Status "Neu erfasst" aus Google Sheets.

```json
{
  "filters": [
    {
      "column": "Status",
      "value": "Neu erfasst"
    }
  ]
}
```

**Output:**
```json
{
  "Name": "Beispiel Tool",
  "URL": "https://example.com",
  "Status": "Neu erfasst",
  "row_number": 42
}
```

### Input-Routing

**Node:** `SetInput`

Setzt eine Input-Variable für späteres Routing:
- `Recherche` = Von Google/EXA
- `Sheets` = Von Google Sheets

**Node:** `Google-Input` / `Sheets-Input`

Bereitet Daten für die jeweilige Quelle vor.

## 🔄 Phase 2: Deduplizierung

### URL-Normalisierung

**Node:** `URL Variable`

Extrahiert URL aus dem `link` Objekt.

```javascript
{
  "URL": $json.link.id
}
```

**Node:** `Call Digistars - deduplication`

Ruft einen separaten Deduplizierungs-Workflow auf.

**Zweck:** Verhindert, dass bereits verarbeitete URLs erneut gecrawlt werden.

## 🛠 Phase 3: Web-Scraping

### Batch-Processing

**Node:** `Loop Over Items`

Verarbeitet URLs in Batches von 10 (konfigurierbar).

```javascript
{
  "batchSize": 10
}
```

**Vorteil:** Verhindert Überlastung und ermöglicht Fortschrittsanzeige.

### PDF-Erkennung

**Node:** `If | kein PDF`

Prüft ob URL auf `.pdf` endet.

```javascript
// Condition
$json.URL NOT endsWith ".pdf"
```

**Routing:**
- `true` → Web-Scraping (crawl4ai/FireCrawl)
- `false` → PDF-Verarbeitung

### Web-Scraping

#### Primär: crawl4ai

**Node:** `crawl4ai`

Lokaler Docker-Service für schnelles Scraping.

```json
{
  "url": "http://host.docker.internal:11235/md",
  "method": "POST",
  "body": {
    "url": "https://example.com",
    "f": "fit",
    "q": null,
    "c": "0"
  }
}
```

**Output:**
```json
{
  "url": "https://example.com",
  "success": true,
  "markdown": "# Überschrift\n\nInhalt...",
  "html": "<html>..."
}
```

**Vorteile:**
- Sehr schnell (lokal)
- Keine API-Kosten
- Keine Rate Limits

**Node:** `output einsortieren`

Standardisiert Output-Format für beide Scraper.

```javascript
{
  "success": $json.error ? false : $json.success,
  "data": {
    "markdown": $json.markdown || "FEHLER: " + $json.error.message
  },
  "debug": {
    "url": $json.url
  }
}
```

#### Backup: FireCrawl

**Node:** `FireCrawl Scraper`

Cloud-Service als Fallback für schwierige Seiten.

```json
{
  "url": "https://example.com",
  "enableDebugLogs": true,
  "onError": "continueRegularOutput"
}
```

**Output:**
```json
{
  "success": true,
  "data": {
    "markdown": "# Überschrift\n\nInhalt...",
    "metadata": {
      "title": "Seitentitel",
      "og:title": "OG Title"
    }
  }
}
```

### PDF-Verarbeitung

**Nodes:** `PDF lesen` → `Code` → `Extract from File`

1. **PDF lesen:** Lädt PDF via crawl4ai
2. **Code:** Konvertiert Base64 zu Binary
3. **Extract from File:** Extrahiert Text aus PDF

**Bei Fehler:**

**Node:** `Fehler Sheets PDF`

Schreibt "Fehler" Status zurück zu Google Sheets.

### Merge & Fehlerbehandlung

**Node:** `Merge3`

Kombiniert Input-Daten mit Scraping-Ergebnis.

```javascript
// Input 1: SetInput
// Input 2: crawl4ai/FireCrawl
```

**Node:** `Firecrawl Fehlerbehandlung`

Prüft `success` Field und routet:
- `success: true` → Weiter zu AI-Analyse
- `success: false` → Fehlerbehandlung

### Token-Limitierung

**Node:** `Auf 20k Tokens begrenzen`

Schneidet Markdown auf max. 20.000 Tokens (≈80.000 Zeichen).

```javascript
const maxTokens = 20000;
const maxChars = maxTokens * 4;

if (markdown.length > maxChars) {
  markdown = markdown.substring(0, maxChars) + 
    "\n\n... [Content wurde nach 100k Tokens abgeschnitten]";
}
```

**Grund:** OpenAI Token Limits und Kostenoptimierung.

## 🤖 Phase 4: AI-Analyse

### Kategorisierung

**Node:** `Kategorisierung`

Hauptnode für KI-gestützte Analyse mit OpenAI o4-mini.

**Prompt-Struktur:**

```
System: [Detaillierte Anweisungen zur Kategorisierung]

User: 
# Quelle
{{ url }}

# Google Snippet (falls vorhanden)
{{ snippet }}

# Webseite als Markdown
{{ markdown }}
```

**Kategorien:**

1. **Tool** - Offizielle Seite eines digitalen Tools
2. **Liste** - Beschreibung/Empfehlung von Tools
3. **Irrelevant** - Nicht relevant für DigiStars

**Output Format:**

```json
{
  "reasoning": "Diese Seite ist...",
  "tools": [
    {"name": "Tool-Name", "url": "URL"}
  ],
  "category": "Tool|Liste|Irrelevant"
}
```

**Node:** `Structured Output Parser`

Erzwingt strukturiertes JSON-Output.

**Node:** `Sammeln`

Aggregiert alle Ergebnisse für weitere Verarbeitung.

### Routing nach Kategorie

**Node:** `Tool/Liste/Irrelevant`

Leitet Items zu den entsprechenden Detail-Extraktoren.

```
Tool → Tool | Details
Liste → Liste Details + Links
Irrelevant → Irrelevant Details
```

### Detail-Extraktion

Alle drei Nodes verwenden den **Information Extractor** mit identischem Schema:

**Schema:**
```json
{
  "Art der Webseite": "Informationsportal",
  "Kurzbeschreibung": "Eine Webseite, die...",
  "Zielgruppe": "Jugendliche und junge Erwachsene",
  "Plattform": "Web, Mobile App",
  "Altersgruppe": "14-25 Jahre",
  "Verfügbare Sprachen": ["Deutsch", "Englisch"],
  "Kosten": "Kostenlos",
  "Kategorie": "Suizidprävention",
  "Organisation": "Diskussionsforum Depression e.V."
}
```

**Unterschiede:**

1. **Tool | Details** (o4-mini)
   - Detaillierteste Extraktion
   - Für manuelle Prüfung vorgesehen

2. **Liste Details** (gpt-4.1-nano)
   - Schneller, günstiger
   - Automatisch als "Geprüft-KI" markiert

3. **Irrelevant Details** (gpt-4.1-nano)
   - Minimale Extraktion
   - Automatisch als "Geprüft-KI" markiert

### Link-Extraktion (nur Liste)

**Node:** `Links`

Extrahiert URLs aus dem `tools` Array.

```javascript
// Split Out auf: output.tools
```

**Node:** `URLs normalisieren`

Standardisiert URLs:
```javascript
// 1. http → https
// 2. www entfernen
// 3. Trailing slash hinzufügen
```

**Node:** `Remove Duplicates`

Entfernt Duplikate basierend auf URL-Field.

### Duplikatsprüfung für Listen-Links

**Zweistufiger Prozess:**

#### Stufe 1: Exakte URL-Prüfung

**Node:** `Exakte Matches`

Sucht nach exakter URL in Google Sheets.

```json
{
  "filter": {
    "column": "URL",
    "value": "{{ $json.URL }}"
  }
}
```

**Node:** `Exakte Matches entfernen`

Merge mit `joinMode: keepNonMatches` entfernt gefundene URLs.

#### Stufe 2: Base-URL Tool-Check

**Node:** `Tools Base-URL Matches`

Sucht nach Base-URL die bereits als Tool erfasst ist.

```json
{
  "filters": [
    {
      "column": "URL",
      "value": "{{ $json.URL.split('/').slice(0,3).join('/') + '/' }}"
    },
    {
      "column": "Score",
      "value": "Tool"
    }
  ]
}
```

**Grund:** Subpages bereits erfasster Tools sollen nicht neu aufgenommen werden.

**Node:** `Original-URL wiederherstellen`

Setzt URL zurück auf Original (nicht Base-URL).

**Node:** `Tools entfernen`

Finale Merge entfernt alle gefundenen Tool-Matches.

### Merge Detail-Extractions

**Node:** `Merge2`

Kombiniert die 3 Detail-Extraktoren wieder zusammen.

```
Input 1: Tool | Details
Input 2: Irrelevant Details
Input 3: Liste Details
```

## 💾 Phase 5: Speicherung

### Routing: Recherche vs. Sheets

**Node:** `Code1`

Prüft ob `row_number` vorhanden ist.

```javascript
try {
  const targetNode = $('Neu erfasste abrufen');
  if (targetNode.item.json.row_number) {
    return { route: "has_data" };
  }
} catch {
  return { route: "no_data" };
}
```

**Node:** `Switch2`

Leitet zu:
- `Recherche` → Neue Einträge erstellen
- `Sheets` → Bestehende Einträge updaten

### Speicherung: Recherche-Modus

**Node:** `Neu`

Erstellt neue Zeile in Google Sheets.

```json
{
  "columns": {
    "Name": "{{ $json.name }}",
    "URL": "{{ $json.URL }}",
    "Status": "{{ category === 'Tool' ? 'Prüfen' : 'Geprüft-KI' }}",
    "Score": "{{ $json.output.category }}",
    "Art der Webseite": "{{ $json.output['Art der Webseite'] }}",
    // ... weitere Felder
    "Crawl-Datum": "{{ $now.format('yyyy-MM-dd tt') }}",
    "Ersteller": "{{ $json.Input }}"
  }
}
```

### Speicherung: Sheets-Modus

**Node:** `Update`

Aktualisiert bestehende Zeile via `row_number`.

```json
{
  "matchingColumn": "row_number",
  "columns": {
    "Status": "{{ category === 'Tool' ? 'Prüfen' : 'Geprüft-KI' }}",
    // ... wie oben
    "Letztes Update": "{{ $now.format('yyyy-MM-dd tt') }}"
  }
}
```

**Unterschied zu Neu:**
- Verwendet `row_number` zum Matchen
- Setzt `Letztes Update` statt `Crawl-Datum`
- Überschreibt `Ersteller` NICHT

### Speicherung: Listen-Links

**Node:** `Anfügen aus Liste`

Fügt neue URLs aus Listen hinzu.

```json
{
  "columns": {
    "Name": "{{ $json.name }}",
    "URL": "{{ $json.URL }}",
    "Status": "Neu erfasst",
    "Crawl-Datum": "{{ $now.format('yyyy-MM-dd tt') }}",
    "Ersteller": "Liste auf {{ $json.parent_url }}"
  }
}
```

**Ersteller-Format:** `"Liste auf https://example.com"` zeigt Ursprung.

### Fehlerbehandlung

#### Fehler bei Recherche

**Node:** `Fehler Google`

Erstellt neue Zeile mit Status "Fehler".

```json
{
  "columns": {
    "URL": "{{ $json.URL }}",
    "Status": "Fehler",
    "Kurzbeschreibung": "{{ $json.error }}",
    "Letztes Update": "{{ $now }}"
  }
}
```

#### Fehler bei Sheets

**Node:** `Fehler Sheets`

Updated bestehende Zeile mit Status "Fehler".

```json
{
  "matchingColumn": "row_number",
  "columns": {
    "Status": "Fehler",
    "Letztes Update": "{{ $now }}"
  }
}
```

## 🔁 Loop-Mechanismus

**Node:** `Replace Me`

Verbindet zurück zu `Loop Over Items` für nächsten Batch.

```
Loop Over Items → ... → Replace Me → Loop Over Items
```

**Abbruch:** Nach Verarbeitung aller Batches endet Workflow automatisch.

## 🎨 Best Practices

### Batch Size anpassen

**Klein (5):**
- Stabiler bei vielen Fehlern
- Langsamer
- Weniger Speicher

**Groß (20):**
- Schneller
- Höherer Speicherverbrauch
- Mehr parallele API-Calls

### Token-Optimierung

**20k Tokens:**
- Standard für die meisten Seiten
- Gute Balance Qualität/Kosten

**10k Tokens:**
- Für sehr lange Seiten
- Reduziert Timeouts
- Kann Qualität beeinträchtigen

### Fehler-Monitoring

**Error Workflow erstellen:**

1. Error Trigger
2. Format Error Message
3. Send Notification (E-Mail/Slack)
4. Optional: Retry fehlgeschlagene URLs

### Rate Limits

**OpenAI:**
- o4-mini: 10.000 RPM (Tier 1)
- Bei Überschreitung: Batch Size reduzieren

**EXA:**
- Free Tier: 1.000 Searches/Monat
- Paid: Unlimited

**Google Search:**
- 100/Tag kostenlos
- Bei mehr: Paid Plan

## 🔧 Erweiterte Konfiguration

### Webhook-Integration

**Node hinzufügen:** Webhook Trigger

```json
{
  "path": "digistars-search",
  "method": "POST",
  "body": {
    "query": "Suchbegriff",
    "num_results": 50
  }
}
```

**Aufruf:**
```bash
curl -X POST https://your-n8n.com/webhook/digistars-search \
  -H "Content-Type: application/json" \
  -d '{"query": "Jugend Mental Health Apps", "num_results": 20}'
```

### Schedule Trigger

**Node hinzufügen:** Schedule Trigger

```json
{
  "rule": {
    "interval": [
      {
        "field": "cronExpression",
        "expression": "0 2 * * *"
      }
    ]
  }
}
```

**Cron Examples:**
- `0 2 * * *` - Täglich um 2 Uhr
- `0 */6 * * *` - Alle 6 Stunden
- `0 9 * * 1` - Jeden Montag um 9 Uhr

### Custom Prompts

**Kategorisierung anpassen:**

Öffne Node "Kategorisierung" → Options → System Message

**Beispiel: Strengere Tool-Definition**

```
Kategoriere als "Tool" nur wenn:
1. Offizielle Produktseite
2. Download/App Store Link vorhanden
3. Direkte Nutzung möglich
4. Keine reine Informationsseite
```

### Zusätzliche Felder extrahieren

**Information Extractor Schema erweitern:**

```json
{
  // Bestehende Felder ...
  "Datenschutz": "DSGVO-konform",
  "Verfügbarkeit": "24/7",
  "Support": "E-Mail, Chat",
  "Zertifizierungen": ["CE-Kennzeichnung"],
  "Wissenschaftliche Basis": "Peer-reviewed"
}
```

## 📚 Weitere Ressourcen

- [README.md](README.md) - Projekt-Übersicht
- [SETUP.md](SETUP.md) - Einrichtungsanleitung
- [FAQ.md](FAQ.md) - Häufige Fragen
- [CONFIGURATION.md](CONFIGURATION.md) - Detaillierte Konfiguration

---

**Fragen zum Workflow?** Erstelle ein [GitHub Issue](https://github.com/your-repo/issues)
