# Quick Start Guide - DigiStars Scrape & Crawl

Schnelleinstieg in 15 Minuten! ⚡

## 🎯 Was du brauchst

- [ ] Docker installiert
- [ ] n8n (lokal oder Cloud)
- [ ] OpenAI API Key ($10 Guthaben)
- [ ] EXA AI Account (Free Tier)
- [ ] Google Account

## ⚡ 5-Minuten Setup

### 1. Docker Container starten

```bash
# crawl4ai
docker run -d --name crawl4ai -p 11235:11235 crawl4ai/server:latest

# n8n (optional, falls nicht installiert)
docker run -d --name n8n -p 5678:5678 n8nio/n8n
```

### 2. Workflow importieren

1. Öffne n8n: http://localhost:5678
2. **Workflows** → **Import from File**
3. Wähle `251016_Digistars_-_Scrape___Crawl.json`
4. Klicke **Import**

### 3. Minimal-Setup

#### OpenAI API Key

1. Besuche https://platform.openai.com/api-keys
2. Erstelle neuen Key
3. In n8n: Klicke auf Node "o4-mini"
4. **Credentials** → **Create New**
5. Füge API Key ein → **Save**

#### Google Sheet

1. Erstelle neues Google Sheet
2. Benenne Tab: "Google-Suche"
3. Füge Header ein (siehe unten)
4. In n8n: Credentials für Google Sheets einrichten

**Header-Zeile:**
```
Name | URL | Status | Score | Art der Webseite | Kurzbeschreibung | Zielgruppe | Plattform | Altersgruppe | Verfügbare Sprachen | Kosten | Kategorie | Organisation | Ersteller | Crawl-Datum | Letztes Update
```

### 4. Test-Run

1. Füge Test-URL in Google Sheet ein:
   ```
   URL: https://fideo.de
   Status: Neu erfasst
   ```

2. In n8n: **Execute Workflow**

3. Prüfe Ergebnis in Google Sheet 🎉

## 🚀 Erste Schritte

### Option A: Sheets-Modus (Empfohlen für Start)

**Workflow:**
```
Google Sheet → Crawl → Analyse → Update Sheet
```

**Schritte:**
1. Füge URLs manuell in Sheet ein (Status: "Neu erfasst")
2. Execute Workflow
3. URLs werden automatisch gecrawlt und kategorisiert

### Option B: EXA Search

**Workflow:**
```
EXA Search → Deduplizierung → Crawl → Analyse → Neue Einträge
```

**Schritte:**
1. Hole EXA API Key: https://exa.ai/
2. Öffne Node "EXA Suche"
3. Füge API Key in Header ein
4. Passe Suchanfrage an:
   ```json
   {
     "Suche": "Deutschland Jugend digitale Hilfe",
     "num_results": "10"
   }
   ```
5. Execute Workflow

## 🔧 Minimale Konfiguration

### Welche Nodes müssen konfiguriert werden?

✅ **Pflicht:**
- OpenAI Credentials (o4-mini, gpt-4.1-nano)
- Google Sheets Credentials + Sheet ID

⚠️ **Optional aber empfohlen:**
- EXA API Key (für Suche)

❌ **Nicht sofort notwendig:**
- Google Custom Search API
- FireCrawl API

### Node-Konfiguration vereinfachen

**Deduplizierung deaktivieren:**
1. Node "Call Digistars - deduplication" → **Disable**
2. Verbinde "URL Variable" direkt mit "Google-Input"

**Google Search deaktivieren:**
1. Node "Google Suche" → **Disable**
2. Nutze nur EXA oder Sheets-Modus

## 📊 Was passiert im Workflow?

```
┌─────────────────┐
│  URL Eingabe    │  ← Manuell (Sheet) oder EXA Search
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   crawl4ai      │  ← Lädt Webseite, konvertiert zu Markdown
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  OpenAI o4-mini │  ← Kategorisiert: Tool/Liste/Irrelevant
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Google Sheets   │  ← Speichert strukturierte Daten
└─────────────────┘
```

## 🎓 Nächste Schritte

### Nach dem ersten erfolgreichen Run:

1. **Mehr URLs testen**
   ```
   - https://jugendnotmail.de
   - https://www.nummergegenkummer.de
   - https://www.u25-deutschland.de
   ```

2. **EXA Search ausprobieren**
   - API Key holen
   - Erste Suche starten

3. **Batch-Processing verstehen**
   - Öffne Node "Loop Over Items"
   - Siehe `batchSize: 10` (10 URLs gleichzeitig)

4. **Automatisierung einrichten**
   - Schedule Trigger für tägliche Suche
   - Webhook für externe Integration

## 🐛 Häufige Anfänger-Fehler

### ❌ "crawl4ai connection refused"

**Lösung:**
```bash
docker ps | grep crawl4ai
# Wenn nicht läuft:
docker start crawl4ai
```

### ❌ "Google Sheets permission denied"

**Lösung:**
1. Google Cloud Console öffnen
2. APIs aktivieren: Sheets API, Drive API
3. OAuth2 Client erstellen
4. In n8n neu verbinden

### ❌ "OpenAI rate limit"

**Lösung:**
1. Batch Size reduzieren: `batchSize: 5`
2. Mehr Guthaben aufladen
3. Tier upgraden

### ❌ "Execution timeout"

**Lösung:**
```
n8n Settings → Executions → Timeout: 3600
```

## 📚 Weiterführende Docs

Nach dem Quick Start:

1. **[SETUP.md](SETUP.md)** - Vollständige Einrichtung
2. **[WORKFLOW-GUIDE.md](WORKFLOW-GUIDE.md)** - Detaillierte Workflow-Erklärung
3. **[README.md](README.md)** - Projekt-Übersicht
4. **[FAQ.md](FAQ.md)** - Häufige Fragen

## 💡 Pro-Tipps

### Kosten sparen

**Batch Size:**
```javascript
batchSize: 5  // Statt 10, langsamer aber stabiler
```

**Token Limit:**
```javascript
maxTokens: 10000  // Statt 20000, weniger Kosten
```

**Nur wichtige Felder extrahieren:**
- Deaktiviere "Irrelevant Details"
- Verwende nur "Tool | Details"

### Performance verbessern

**Parallele Crawls:**
```javascript
batchSize: 20  // Mehr gleichzeitig (braucht mehr RAM)
```

**crawl4ai statt FireCrawl:**
- Viel schneller
- Keine API-Kosten
- Funktioniert für 90% der Seiten

### Qualität verbessern

**Besseres AI-Model:**
```
o4-mini → o4 (teurer, besser)
```

**Längere Prompts:**
- Füge mehr Beispiele hinzu
- Spezifiziere Edge-Cases

## ✅ Checkliste: Setup abgeschlossen

- [ ] Docker läuft (crawl4ai)
- [ ] n8n läuft
- [ ] Workflow importiert
- [ ] OpenAI Credentials eingerichtet
- [ ] Google Sheets eingerichtet
- [ ] Erster Test-Run erfolgreich
- [ ] Ergebnis in Sheet sichtbar

**Fertig?** Glückwunsch! 🎉

**Probleme?** Check [Troubleshooting](README.md#troubleshooting)

---

**Geschätzte Setup-Zeit:** 15 Minuten  
**Schwierigkeit:** Anfänger  
**Kosten:** ~$0.10 pro 10 URLs

Viel Erfolg! 🚀
