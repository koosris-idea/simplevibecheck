# Spreekbuis AI-inzichten — Ontwerpdocument

**Datum:** 2026-03-25
**Project:** SimpleVibeCheck / Het Goede Gesprek
**Bestand:** `spreekbuis.html`

---

## Doel

De Spreekbuis luistert real-time naar een gesprek en visualiseert de gespreksinhoud als wordcloud. Dit ontwerp voegt een AI-laag toe die het transcript analyseert en actiepunten, wensen en oplossingen extraheert — zichtbaar terwijl het gesprek loopt.

---

## Lay-out

Het scherm wordt permanent in tweeën gedeeld:

- **Boven (2/3):** bestaande wordcloud — ongewijzigd
- **Onder (1/3):** wit AI-paneel, gescheiden van de wordcloud door een oranje lijn (`#e84300`)

Het AI-paneel bevat drie gelijke kolommen:

| Kolom | Label | Emoji |
|-------|-------|-------|
| 1 | Actiepunten | ✅ |
| 2 | Wensen | 💬 |
| 3 | Oplossingen | 💡 |

Elke kolom toont items als lichtgrijze kaartjes (`#f5f5f5`) met zwarte tekst. Kolomtitels zijn oranje en in hoofdletters. Rechtsboven in het paneel staat een tijdindicator: "bijgewerkt Xs geleden".

---

## Werking

### Trigger
Elke 30 seconden wordt gecontroleerd of er nieuw transcript is (transcript is bijgekomen sinds de vorige Claude-aanroep). Zo ja, dan volgt een API-aanroep. Zo nee, wordt de aanroep overgeslagen.

### API-aanroep
- **Model:** `claude-haiku-4-5-20251001` (snel, lage kosten)
- **Aanroep:** direct vanuit de browser via de Anthropic Messages API
- **API-sleutel:** hardcoded in de frontend (prototype-aanname, bewust gekozen)

### Prompt
```
Je analyseert een transcriptie van een buurtgesprek.

Huidig transcript:
<transcript>
[volledig transcript]
</transcript>

Bestaande inzichten (voorkom duplicaten):
<bestaand>
[JSON met huidige lijsten]
</bestaand>

Geef een VOLLEDIGE bijgewerkte JSON terug met alle inzichten (oud + nieuw, duplicaten samengevoegd):
{
  "actiepunten": ["...", "..."],
  "wensen": ["...", "..."],
  "oplossingen": ["...", "..."]
}

Regels:
- Alleen wat écht gezegd is in het transcript
- Maximaal 8 items per categorie
- Nederlands, beknopt (max 6 woorden per item)
- Geen duplicaten of herformuleringen van hetzelfde punt
- Geef alleen de JSON terug, geen uitleg
```

### Respons verwerking
Claude retourneert JSON. De client parseert dit en vervangt de drie lijsten volledig met de nieuwe gecombineerde set. Bij een parse-fout blijven de vorige inzichten staan.

### Accumulatie
Claude ontvangt altijd de volledige bestaande lijsten en het volledige transcript. De deduplicatie en samenvoeging worden volledig door Claude afgehandeld.

---

## Technische details

### Staat
Twee nieuwe variabelen naast de bestaande:
- `lastAnalyzedLength` — lengte van het transcript bij de vorige aanroep (voor delta-detectie)
- `insights` — object `{ actiepunten: [], wensen: [], oplossingen: [] }`

### Interval
Een tweede `setInterval` van 30 seconden, parallel aan de bestaande spraakherkenning. Gestart bij `startListening()`, gestopt bij `stopListening()`.

### DOM-aanpassingen
- `#cloud` krijgt `flex: 2` (was: `flex: 1`)
- Nieuw element `#ai-panel` met `flex: 1`, witte achtergrond, oranje bovenlijn
- Drie `div.ai-col` elementen daarbinnen
- Tijdindicator `#ai-updated` rechtsboven in het paneel

### Wat niet verandert
- Spraakherkenning (`SpeechRecognition`) — ongewijzigd
- Wordcloud-logica — ongewijzigd
- Stopwoordfilter — ongewijzigd
- Navigatiebalk — ongewijzigd

---

## Beperkingen (by design)

- API-sleutel zit in de frontend (prototype)
- Geen foutmelding bij API-limiet — stille fallback naar vorige inzichten
- Geen persistentie na pagina-refresh
- Werkt alleen zolang spraakherkenning actief is

---

## Niet in scope

- Exportfunctie voor inzichten
- Backend/proxy voor API-sleutel
- Inzichten op andere pagina's (VibeCheck, Overzicht)
