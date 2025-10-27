# Setup Anleitung - DigiStars Scrape & Crawl

Diese Anleitung führt dich Schritt für Schritt durch die komplette Einrichtung des Workflows.

## 🎯 Vorbereitungsphase

### 1. API-Keys beschaffen

#### Google Custom Search API

1. Besuche [Google Cloud Console](https://console.cloud.google.com/)
2. Erstelle ein neues Projekt oder wähle ein bestehendes aus
3. Aktiviere die **Custom Search API**:
   - Navigation → APIs & Services → Library
   - Suche nach "Custom Search API"
   - Klicke "Enable"
4. Erstelle API Credentials:
   - APIs & Services → Credentials
   - Create Credentials → API Key
   - Kopiere den API Key
5. Erstelle eine Custom Search Engine:
   - Besuche [Programmable Search Engine](https://programmablesearchengine.google.com/)
   - Klicke "Add" / "Neu"
   - Konfiguration:
     - Name: "DigiStars Search"
     - Sites to search: "Search the entire web"
     - Language: German
   - Kopiere die Search Engine ID (cx)

**Kosten:** 100 Suchen/Tag kostenlos, danach $5 pro 1.000 Abfragen

#### EXA AI API

1. Besuche [EXA AI](https://exa.ai/)
2. Registriere dich für einen Account
3. Navigiere zu API Settings
4. Erstelle einen neuen API Key
5. Kopiere den Key

**Kosten:** Free Tier verfügbar, danach Pay-as-you-go

#### OpenAI API

1. Besuche [OpenAI Platform](https://platform.openai.com/)
2. Erstelle einen Account (falls noch nicht vorhanden)
3. Navigiere zu API Keys
4. Erstelle einen neuen API Key
5. Lade Guthaben auf (mindestens $10 empfohlen)
6. Kopiere den Key

**Kosten:**
- gpt-4.1-nano: ~$0.15 pro 1M Input Tokens
- o4-mini: ~$1.50 pro 1M Input Tokens

#### FireCrawl (Optional)

1. Besuche [FireCrawl](https://firecrawl.dev/)
2. Registriere dich
3. Erstelle einen API Key
4. Kopiere den Key

**Kosten:** Free Tier: 500 Credits/Monat

### 2. Google Sheets vorbereiten

#### Sheet erstellen

1. Öffne [Google Sheets](https://sheets.google.com/)
2. Erstelle ein neues Sheet: "DigiStars Datenbank"
3. Benenne das erste Tab: "Google-Suche"

#### Spalten einrichten

Kopiere folgende Header-Zeile in Zeile 1:

```
Name | URL | Status | Score | Fachteam-Bewertung | Kommentar | Art der Webseite | Kurzbeschreibung | Zielgruppe | Plattform | Altersgruppe | Verfügbare Sprachen | Kosten | Kategorie | Organisation | Ersteller | Crawl-Datum | Letztes Update
```

#### OAuth2 Credentials

1. Besuche [Google Cloud Console](https://console.cloud.google.com/)
2. Wähle dein Projekt
3. Navigation → APIs & Services → Credentials
4. Create Credentials → OAuth 2.0 Client ID
5. Application type: Web application
6. Authorized redirect URIs:
   - `http://localhost:5678/rest/oauth2-credential/callback`
   - `https://deine-n8n-domain.de/rest/oauth2-credential/callback`
7. Erstelle und kopiere Client ID + Secret
8. Aktiviere folgende APIs:
   - Google Sheets API
   - Google Drive API

### 3. Docker Setup

#### crawl4ai installieren

```bash
# Docker Hub Image ziehen
docker pull crawl4ai/server:latest

# Container starten
docker run -d \
  --name crawl4ai \
  -p 11235:11235 \
  --restart unless-stopped \
  crawl4ai/server:latest

# Verfügbarkeit testen
curl http://localhost:11235/health
```

**Expected Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0"
}
```

### 4. n8n Installation

#### Lokale Installation (empfohlen für Entwicklung)

```bash
# Via npm
npm install -g n8n

# Starten
n8n start

# Browser öffnen
open http://localhost:5678
```

#### Docker Installation

```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  --restart unless-stopped \
  n8nio/n8n
```

#### Cloud (n8n.cloud)

1. Registriere dich auf [n8n.cloud](https://n8n.cloud/)
2. Erstelle eine neue Instanz
3. Wähle einen Plan (Starter $20/Monat)

## 🔧 Workflow-Konfiguration

### 1. Workflow importieren

1. Öffne n8n im Browser
2. Klicke links auf **Workflows**
3. Rechts oben: **Import from File**
4. Wähle `251016_Digistars_-_Scrape___Crawl.json`
5. Klicke **Import**

### 2. Credentials einrichten

#### Google Sheets OAuth2

1. In n8n: Klicke auf einen beliebigen Google Sheets Node
2. Credentials → Create New
3. Name: "Google Sheets | DigiStars"
4. OAuth2:
   - Client ID: [Deine Client ID]
   - Client Secret: [Dein Client Secret]
   - Authorization URL: `https://accounts.google.com/o/oauth2/v2/auth`
   - Access Token URL: `https://oauth2.googleapis.com/token`
   - Scope: `https://www.googleapis.com/auth/spreadsheets https://www.googleapis.com/auth/drive.file`
5. Klicke **Connect my account**
6. Durchlaufe den OAuth2 Flow
7. Save

#### OpenAI

1. Klicke auf einen OpenAI Node (z.B. "o4-mini")
2. Credentials → Create New
3. Name: "OpenAI DigiStars"
4. API Key: [Dein OpenAI API Key]
5. Save

#### EXA AI

1. Öffne Node "EXA Suche"
2. In Header Parameters:
   - Name: `x-api-key`
   - Value: [Dein EXA API Key]
3. Save

#### Google Custom Search

1. Öffne Node "Google Suche"
2. In Query Parameters:
   - `key`: [Dein Google API Key]
   - `cx`: [Deine Search Engine ID]
3. Save

#### FireCrawl (Optional)

1. Öffne Node "FireCrawl Scraper"
2. Credentials → Create New
3. API Key: [Dein FireCrawl API Key]
4. Save

### 3. Google Sheet IDs aktualisieren

In jedem Google Sheets Node:

1. Öffne den Node
2. Document ID → List Mode
3. Wähle "DigiStars Datenbank"
4. Sheet Name → "Google-Suche"
5. Save

**Betroffene Nodes:**
- Neu erfasste abrufen
- Update
- Neu
- Anfügen aus Liste
- Fehler Google
- Fehler Sheets
- Fehler Sheets PDF
- Exakte Matches
- Tools Base-URL Matches

### 4. Deduplizierungs-Workflow einrichten

Der Workflow "Digistars - deduplication" wird referenziert. Dieser muss separat erstellt werden oder du deaktivierst den Node "Call Digistars - deduplication".

**Option 1: Node deaktivieren**
```
1. Klicke auf Node "Call Digistars - deduplication"
2. Rechtsklick → Disable
3. Verbinde "URL Variable" direkt mit "Google-Input"
```

**Option 2: Deduplizierungs-Workflow erstellen**
```
Erstelle einen einfachen Workflow der:
- Input: URL
- Output: {URL: "url", isDuplicate: false}
- Workflow ID in Node eintragen
```

### 5. Test-Ausführung

#### Manuelle Test-Suche

1. Öffne Node "EXA Suchanfrage"
2. Passe die Suche an:
   ```json
   {
     "Suche": "Test Jugend Digital",
     "num_results": "5"
   }
   ```
3. Deaktiviere "Neu erfasste abrufen" Node
4. Klicke "Execute Workflow"
5. Prüfe die Ergebnisse in Google Sheets

#### Sheets-Modus testen

1. Füge manuell eine Test-URL in Google Sheets ein:
   - URL: `https://fideo.de`
   - Status: `Neu erfasst`
2. Aktiviere "When clicking 'Execute workflow'"
3. Deaktiviere "EXA Suchanfrage"
4. Klicke "Execute Workflow"
5. Prüfe ob URL gecrawlt und kategorisiert wurde

## 🔍 Validierung

### Checklist

- [ ] crawl4ai antwortet auf http://localhost:11235/health
- [ ] n8n läuft und ist erreichbar
- [ ] Alle Credentials sind korrekt eingerichtet
- [ ] Google Sheet ist erstellt mit korrekten Spalten
- [ ] Workflow wurde importiert
- [ ] Test-Ausführung war erfolgreich
- [ ] Ergebnisse erscheinen in Google Sheets

### Häufige Fehler beim Setup

#### "crawl4ai connection refused"
```bash
# Prüfe Docker Container
docker ps | grep crawl4ai

# Prüfe Logs
docker logs crawl4ai

# Prüfe Port
netstat -an | grep 11235
```

#### "Google Sheets insufficient permissions"
```
Fehler: OAuth2 Scopes fehlen
Lösung:
1. Google Cloud Console → Credentials
2. OAuth2 Client bearbeiten
3. Scopes hinzufügen:
   - https://www.googleapis.com/auth/spreadsheets
   - https://www.googleapis.com/auth/drive.file
4. In n8n: Reconnect account
```

#### "OpenAI rate limit"
```
Fehler: Zu viele Requests
Lösung:
1. Batch Size reduzieren (Loop Over Items: batchSize = 5)
2. OpenAI Tier prüfen
3. Usage Limits in OpenAI Dashboard erhöhen
```

#### "Node execution timeout"
```
Fehler: Execution dauert zu lange
Lösung:
1. n8n Settings → Executions
2. Timeout erhöhen: 3600 Sekunden
3. Batch Size reduzieren
```

## 🚀 Produktiv-Deployment

### n8n Cloud

1. Exportiere den Workflow mit Credentials (verschlüsselt)
2. Importiere in n8n Cloud Instanz
3. Teste alle Nodes
4. Aktiviere Workflow

### Self-Hosted mit Docker Compose

Erstelle `docker-compose.yml`:

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=changeme
      - N8N_HOST=https://deine-domain.de
      - WEBHOOK_URL=https://deine-domain.de/
    volumes:
      - ~/.n8n:/home/node/.n8n
    depends_on:
      - crawl4ai

  crawl4ai:
    image: crawl4ai/server:latest
    restart: always
    ports:
      - "11235:11235"
```

Starten:
```bash
docker-compose up -d
```

### Monitoring einrichten

#### Error Workflow

1. In n8n: Settings → Error Workflow
2. Erstelle einen neuen Workflow:
   - Trigger: Error Trigger
   - Action: Sende E-Mail/Slack Benachrichtigung
3. Aktiviere Error Workflow

#### Logging

```bash
# n8n Logs verfolgen
docker logs -f n8n

# crawl4ai Logs
docker logs -f crawl4ai
```

## 📊 Nächste Schritte

Nach erfolgreichem Setup:

1. **Erste echte Suche durchführen**
   - EXA Suche mit echtem Suchbegriff
   - Ergebnisse in Sheets prüfen

2. **Automatisierung einrichten**
   - Schedule Trigger für tägliche Searches
   - Webhook für externe Integration

3. **Monitoring aufsetzen**
   - Error Workflow aktivieren
   - Benachrichtigungen konfigurieren

4. **Dokumentation lesen**
   - [README.md](README.md) für Workflow-Details
   - [FAQ.md](FAQ.md) für häufige Fragen

## 🆘 Support

Bei Problemen:
1. Prüfe die [Troubleshooting-Sektion](README.md#troubleshooting)
2. Suche in n8n Community Forum
3. Erstelle ein GitHub Issue

---

**Einrichtung abgeschlossen?** Herzlichen Glückwunsch! 🎉

Nächster Schritt: [Verwendung](README.md#verwendung)
