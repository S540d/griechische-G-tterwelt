# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt

Interaktive Lernseite zur griechischen Götterwelt. Einzige Quelldatei: `start.html` (~1000 Zeilen). Keine Build-Tools, kein Server nötig – Datei direkt im Browser öffnen (Doppelklick, Drag & Drop).

## Architektur

Alles in einer HTML-Datei in drei Blöcken:

1. **`<style>`** – CSS-Variablen (`--gold`, `--obsidian` usw.) und alle Styles für Cards, Modal, Stammbaum-SVG, Quiz
2. **`<script id="gdata" type="application/json">`** – Götterdaten als JSON-Array (26 Einträge). Diesen Block bearbeiten, um Götter hinzuzufügen oder Inhalte zu ändern.
3. **`<script>`** – Gesamte Logik als IIFE, aufgeteilt in Abschnitte:
   - **STARS** – Canvas-Sternhimmel-Animation
   - **DATA** – JSON parsen, `godMap` und Kategorie-Konstanten (`DC`, `DL`, `DO`) aufbauen
   - **TABS** – Pantheon / Stammbaum / Quiz umschalten
   - **CARDS** – Karten rendern, Domänen-Filter + Live-Suchfeld
   - **MODAL** – Detailansicht öffnen/schließen (`openM`, `closeM`) mit Fokus-Management
   - **TREE** – SVG-Stammbaum mit Pan/Zoom (`buildTree`, wird lazy beim ersten Tab-Wechsel aufgebaut)
   - **QUIZ** – Eigene IIFE: Multiple-Choice aus Götterdaten generieren, Punktzahl, Ergebnis

## Götterdaten-Schema

Jeder Eintrag im JSON-Array hat diese Felder:

```json
{
  "id": "zeus",           // eindeutiger Bezeichner, referenziert in parents[] und pairs[]
  "name": "Zeus",
  "roman": "Iuppiter / Jupiter",
  "epithet": "Vater der Goetter und Menschen",
  "sym": "&#9889;",       // HTML-Entity für das Symbol-Emoji
  "color": "#5b8ec4",     // Akzentfarbe der Karte
  "cats": ["olympier","himmel"],  // Domänen-Filter-Kategorien
  "ol": true,             // true = Olympier (zeigt goldenen Punkt im Stammbaum)
  "tag": "...",           // Kurzbeschreibung unter dem Namen
  "attrs": ["Blitz","Adler","..."],
  "desc": "...",          // Langer Beschreibungstext
  "myths": ["...","..."], // Liste bekannter Mythen
  "facts": [["Label","Wert"], ...],  // Steckbrief-Tabelle (je 2-Felder-Array)
  "sources": ["Hesiod: Theogonie", "..."],  // Primärquellen (optional, erscheint im Modal)
  "parents": ["kronos","rhea"],      // IDs der Elternteile (müssen im Array existieren)
  "pairs": ["hera"]                  // IDs von Partnern (werden als gestrichelte Linie gezeigt)
}
```

Gültige `cats`-Werte: `olympier`, `himmel`, `meer`, `unterwelt`, `krieg`, `weisheit`, `liebe`, `titan`, `held`

## Stammbaum-Layout

Götter werden manuell in `LAYERS`-Array (in `buildTree()`) auf Generationsebenen verteilt. Neue Götter müssen dort eingetragen werden, sonst erscheinen sie nicht im Stammbaum.

```js
var LAYERS = [
  ['gaia', 'uranos'],                                              // Gen -1 (Urgottheiten)
  ['rhea', 'kronos', 'prometheus', 'nike', 'hypnos', 'charon'],  // Gen 0
  ['zeus', 'hera', 'poseidon', 'hades', 'demeter', 'hestia'],    // Gen 1
  ['athene', 'apollon', 'artemis', 'ares', ...],                  // Gen 2
  ['eros', 'asklepios', 'herakles']                               // Gen 3
];
```

## Features (aktueller Stand)

- **Pantheon** – 26 Götter/Titanen/Helden mit Filterkategorien und Live-Suchfeld
- **Stammbaum** – SVG mit Pan/Zoom, Tastaturnavigation, Mobile-optimiert
- **Modal** – Beschreibung, Attribute, Mythen, Steckbrief, Quellenangaben; vollständig barrierefrei (ARIA, Fokus-Management)
- **Quiz** – 10 zufällige Multiple-Choice-Fragen aus 4 Typen (Symbol, Attribut, Familie, Domäne)
- **Footer** – Quellenübersicht mit Links zu theoi.com und Projekt Gutenberg

## Barrierefreiheit (Conventions)

- Karten: `role="button"`, `tabindex="0"`, `aria-label`, Enter/Space aktivieren Modal
- Filter-Buttons: `aria-pressed` aktuell halten
- Modal: `role="dialog"`, `aria-modal`, `aria-labelledby`; Fokus auf `.close-btn` beim Öffnen, Rückgabe an auslösende Karte beim Schließen
- Trefferzähler: `aria-live="polite"` für Screenreader
- SVG-Baumknoten: `role="button"`, `tabindex="0"`, Enter/Space öffnen Modal

## Entwicklung

Keine Build-Pipeline. Direkt `start.html` im Browser öffnen und nach Änderungen neu laden (F5).

Für DevTools-Debugging: alle Variablen leben in einer IIFE, sind also nicht global erreichbar. Temporär `window.gods = gods` einfügen zum Inspizieren.
