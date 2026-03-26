# Spreekbuis AI-inzichten — Implementatieplan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Voeg een wit AI-paneel toe aan `spreekbuis.html` dat elke 30 seconden het volledige transcript naar de Claude API stuurt en actiepunten, wensen en oplossingen extraheert.

**Architecture:** Eén enkel HTML-bestand (`spreekbuis.html`). De bestaande wordcloud-logica en spraakherkenning blijven volledig ongewijzigd. Er worden zes toevoegingen gedaan: CSS, HTML, state, render, API-aanroep, en interval-koppeling. Geen nieuwe bestanden.

**Tech Stack:** Vanilla HTML/CSS/JS, Anthropic Messages API (`claude-haiku-4-5-20251001`), Web Speech API (bestaand)

---

## Bestandsoverzicht

| Bestand | Actie | Wat verandert |
|---------|-------|---------------|
| `spreekbuis.html` | Wijzigen | CSS, HTML-structuur, JS state, render, API-aanroep, interval |

---

### Taak 1: CSS — AI-paneel stijlen

**Bestand:** `spreekbuis.html:40-52` (CSS-blok na `#cloud`)

- [ ] **Stap 1: Pas `#cloud` aan van `flex: 1` naar `flex: 2`**

Zoek:
```css
#cloud {
  flex: 1;
```
Vervang door:
```css
#cloud {
  flex: 2;
```

- [ ] **Stap 2: Voeg CSS toe voor het AI-paneel direct ná de `.word.pulsing`-regel (na regel 71)**

```css
#ai-panel {
  flex: 1;
  background: #ffffff;
  border-top: 2px solid #e84300;
  padding: 8px 10px;
  overflow-y: auto;
  flex-shrink: 0;
}

#ai-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 7px;
}

#ai-title {
  color: #e84300;
  font-family: 'Nunito', sans-serif;
  font-size: 0.6rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

#ai-updated {
  color: #aaa;
  font-family: 'Nunito', sans-serif;
  font-size: 0.55rem;
}

#ai-columns {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 6px;
}

.ai-col-title {
  color: #e84300;
  font-family: 'Nunito', sans-serif;
  font-size: 0.58rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  margin-bottom: 4px;
}

.ai-item {
  background: #f5f5f5;
  border-radius: 4px;
  padding: 5px;
  margin-bottom: 3px;
  font-family: 'Nunito', sans-serif;
  font-size: 0.65rem;
  color: #111;
  line-height: 1.3;
}
```

- [ ] **Stap 3: Open `spreekbuis.html` in de browser — wordcloud moet nu de bovenste 2/3 vullen, onderste 1/3 leeg wit**

- [ ] **Stap 4: Commit**

```bash
cd /Users/koosris/SimpleVibeCheck
git add spreekbuis.html
git commit -m "feat: voeg CSS toe voor AI-paneel (lay-out)"
```

---

### Taak 2: HTML — AI-paneel structuur

**Bestand:** `spreekbuis.html:110-111` (tussen `#cloud` en `#warning`)

- [ ] **Stap 1: Voeg het AI-paneel HTML-element toe direct ná `<div id="cloud"></div>`**

Zoek:
```html
  <div id="cloud"></div>
  <div id="warning"></div>
```
Vervang door:
```html
  <div id="cloud"></div>

  <div id="ai-panel">
    <div id="ai-header">
      <span id="ai-title">AI Inzichten</span>
      <span id="ai-updated"></span>
    </div>
    <div id="ai-columns">
      <div>
        <div class="ai-col-title">✅ Actiepunten</div>
        <div id="col-actiepunten"></div>
      </div>
      <div>
        <div class="ai-col-title">💬 Wensen</div>
        <div id="col-wensen"></div>
      </div>
      <div>
        <div class="ai-col-title">💡 Oplossingen</div>
        <div id="col-oplossingen"></div>
      </div>
    </div>
  </div>

  <div id="warning"></div>
```

- [ ] **Stap 2: Verifieer in browser — wit paneel met drie lege kolommen zichtbaar onder de wordcloud**

- [ ] **Stap 3: Commit**

```bash
git add spreekbuis.html
git commit -m "feat: voeg HTML-structuur toe voor AI-paneel"
```

---

### Taak 3: JS — State variabelen en transcript-accumulator

**Bestand:** `spreekbuis.html` — script-blok

- [ ] **Stap 1: Voeg nieuwe state variabelen toe direct ná de bestaande variabelen (na `var recognition = null;`, regel ~148)**

Zoek:
```javascript
    var wordCounts  = {};
    var wordOrder   = []; // insertion order for tie-breaking
    var listening   = false;
    var recognition = null;
```
Vervang door:
```javascript
    var wordCounts  = {};
    var wordOrder   = []; // insertion order for tie-breaking
    var listening   = false;
    var recognition = null;

    // AI-inzichten state
    var CLAUDE_API_KEY      = 'JOUW_API_SLEUTEL_HIER';
    var fullTranscript      = '';
    var lastAnalyzedLength  = 0;
    var insights            = { actiepunten: [], wensen: [], oplossingen: [] };
    var aiIntervalId        = null;
    var lastUpdated         = null;
    var updatedTimerId      = null;
```

- [ ] **Stap 2: Voeg transcript-accumulator toe in `processText()` als eerste regel van de functie**

Zoek:
```javascript
    function processText(text) {
      var words = text.toLowerCase()
```
Vervang door:
```javascript
    function processText(text) {
      fullTranscript += ' ' + text;
      var words = text.toLowerCase()
```

- [ ] **Stap 3: Reset AI-state in `startListening()` — voeg toe direct ná `listening = true;`**

Zoek:
```javascript
    function startListening() {
      listening = true;
      warning.textContent = '';
```
Vervang door:
```javascript
    function startListening() {
      listening = true;
      warning.textContent = '';
      fullTranscript = '';
      lastAnalyzedLength = 0;
      lastUpdated = null;
      insights = { actiepunten: [], wensen: [], oplossingen: [] };
```

- [ ] **Stap 4: Commit**

```bash
git add spreekbuis.html
git commit -m "feat: voeg state en transcript-accumulator toe"
```

---

### Taak 4: JS — Render functies

**Bestand:** `spreekbuis.html` — voeg toe ná de `stopListening()`-functie

- [ ] **Stap 1: Voeg `renderInsights()` en `renderCol()` toe ná de sluitende accolade van `stopListening()`**

Zoek:
```javascript
    function stopListening() {
      listening = false;
      if (recognition) { recognition.stop(); recognition = null; }
    }
```
Vervang door:
```javascript
    function stopListening() {
      listening = false;
      if (recognition) { recognition.stop(); recognition = null; }
      if (aiIntervalId)  { clearInterval(aiIntervalId);  aiIntervalId  = null; }
      if (updatedTimerId){ clearInterval(updatedTimerId); updatedTimerId = null; }
    }

    function renderInsights() {
      renderCol(document.getElementById('col-actiepunten'), insights.actiepunten);
      renderCol(document.getElementById('col-wensen'),      insights.wensen);
      renderCol(document.getElementById('col-oplossingen'), insights.oplossingen);
    }

    function renderCol(el, items) {
      el.innerHTML = '';
      items.forEach(function(item) {
        var div = document.createElement('div');
        div.className = 'ai-item';
        div.textContent = item;
        el.appendChild(div);
      });
    }
```

- [ ] **Stap 2: Verifieer — open browser console, plak dit in:**
```javascript
insights = { actiepunten: ['Speeltuin renoveren'], wensen: ['Meer groen'], oplossingen: ['Buurtapp'] };
renderInsights();
```
Verwacht: drie items verschijnen in het witte paneel.

- [ ] **Stap 3: Commit**

```bash
git add spreekbuis.html
git commit -m "feat: voeg renderInsights en renderCol toe"
```

---

### Taak 5: JS — Claude API-aanroep

**Bestand:** `spreekbuis.html` — voeg toe ná `renderCol()`

- [ ] **Stap 1: Voeg `analyzeTranscript()` toe direct ná `renderCol()`**

```javascript
    function analyzeTranscript() {
      var text = fullTranscript.trim();
      if (text.length < 30 || text.length === lastAnalyzedLength) return;
      lastAnalyzedLength = text.length;

      var prompt = 'Je analyseert een transcriptie van een buurtgesprek.\n\n'
        + 'Huidig transcript:\n' + text + '\n\n'
        + 'Bestaande inzichten (voorkom duplicaten):\n' + JSON.stringify(insights) + '\n\n'
        + 'Geef een VOLLEDIGE bijgewerkte JSON terug met alle inzichten (oud + nieuw, duplicaten samengevoegd):\n'
        + '{"actiepunten":["..."],"wensen":["..."],"oplossingen":["..."]}\n\n'
        + 'Regels:\n'
        + '- Alleen wat écht gezegd is in het transcript\n'
        + '- Maximaal 8 items per categorie\n'
        + '- Nederlands, beknopt (max 6 woorden per item)\n'
        + '- Geen duplicaten of herformuleringen van hetzelfde punt\n'
        + '- Geef alleen de JSON terug, geen uitleg';

      fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key': CLAUDE_API_KEY,
          'anthropic-version': '2023-06-01',
          'anthropic-dangerous-direct-browser-client': 'true'
        },
        body: JSON.stringify({
          model: 'claude-haiku-4-5-20251001',
          max_tokens: 512,
          messages: [{ role: 'user', content: prompt }]
        })
      })
      .then(function(res) { return res.json(); })
      .then(function(data) {
        var responseText = data.content && data.content[0] && data.content[0].text;
        if (!responseText) return;
        var match = responseText.match(/\{[\s\S]*\}/);
        if (!match) return;
        var parsed = JSON.parse(match[0]);
        if (Array.isArray(parsed.actiepunten)) insights.actiepunten = parsed.actiepunten;
        if (Array.isArray(parsed.wensen))      insights.wensen      = parsed.wensen;
        if (Array.isArray(parsed.oplossingen)) insights.oplossingen = parsed.oplossingen;
        lastUpdated = Date.now();
        renderInsights();
      })
      .catch(function() { /* stille fallback — vorige inzichten blijven staan */ });
    }
```

- [ ] **Stap 2: Vervang `'JOUW_API_SLEUTEL_HIER'` met een echte Anthropic API-sleutel**

Haal een sleutel op via console.anthropic.com. Vervang de placeholder in de `CLAUDE_API_KEY`-variabele.

> ⚠️ **Let op:** De repo is publiek op GitHub. Commit de API-sleutel NIET naar git. Werk lokaal en push `spreekbuis.html` pas nadat je de sleutel weer hebt vervangen door de placeholder, of voeg `spreekbuis.html` tijdelijk toe aan `.gitignore` tijdens het testen.

- [ ] **Stap 3: Verifieer via browser console — plak dit in:**
```javascript
fullTranscript = 'We moeten de speeltuin op de Kerkstraat echt renoveren. Er is geen verlichting en de kinderen kunnen er niet veilig spelen. Ik zou graag meer groen in de straat zien.';
analyzeTranscript();
```
Wacht 3-5 seconden. Verwacht: kolommen vullen zich met inzichten.

- [ ] **Stap 4: Commit**

```bash
git add spreekbuis.html
git commit -m "feat: voeg Claude API-aanroep toe voor transcript-analyse"
```

---

### Taak 6: JS — Tijdindicator en interval koppelen

**Bestand:** `spreekbuis.html`

- [ ] **Stap 1: Voeg `startUpdatedTimer()` toe direct ná `analyzeTranscript()`**

```javascript
    function startUpdatedTimer() {
      updatedTimerId = setInterval(function() {
        var el = document.getElementById('ai-updated');
        if (!el) return;
        if (lastUpdated === null) { el.textContent = ''; return; }
        var secs = Math.round((Date.now() - lastUpdated) / 1000);
        el.textContent = 'bijgewerkt ' + secs + 's geleden';
      }, 1000);
    }
```

- [ ] **Stap 2: Start de AI-interval in `startListening()` — voeg toe direct ná `recognition.start();`**

Zoek:
```javascript
      recognition.start();
    }
```
Vervang door:
```javascript
      recognition.start();
      aiIntervalId = setInterval(analyzeTranscript, 30000);
      startUpdatedTimer();
    }
```

- [ ] **Stap 3: Verifieer end-to-end in browser**

1. Open `spreekbuis.html` in Chrome of Vivaldi
2. Tik op het scherm om te starten
3. Spreek ~30 seconden over een buurtonderwerp
4. Na 30 seconden: inzichten verschijnen in het witte paneel
5. Tijdindicator tikt mee ("bijgewerkt Xs geleden")
6. Spreek nog 30 seconden: nieuwe inzichten worden toegevoegd aan bestaande
7. Tik opnieuw: luisteren stopt, paneel blijft staan

- [ ] **Stap 4: Commit**

```bash
git add spreekbuis.html
git commit -m "feat: koppel AI-interval en tijdindicator aan start/stop"
```

---

### Taak 7: Publiceren

- [ ] **Stap 1: Push naar GitHub Pages**

```bash
cd /Users/koosris/SimpleVibeCheck
git push origin main
```

- [ ] **Stap 2: Verifieer live op GitHub Pages URL na ~60 seconden**
