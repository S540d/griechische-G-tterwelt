# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt

Interaktive Lernseite zur griechischen Götterwelt. Einzige Quelldatei: `start.html` (~700 Zeilen). Keine Build-Tools, kein Server nötig – Datei direkt im Browser öffnen.

## Architektur

Alles in einer HTML-Datei in drei Blöcken:

1. **`<style>`** – CSS-Variablen (`--gold`, `--obsidian` usw.) und alle Styles für Cards, Modal, Stammbaum-SVG
2. **`<script id="gdata" type="application/json">`** – Götterdaten als JSON-Array (20 Einträge). Diesen Block bearbeiten, um Götter hinzuzufügen oder Inhalte zu ändern.
3. **`<script>`** – Gesamte Logik als IIFE, aufgeteilt in Abschnitte:
   - **STARS** – Canvas-Sternhimmel-Animation
   - **DATA** – JSON parsen, `godMap` und Kategorie-Konstanten (`DC`, `DL`, `DO`) aufbauen
   - **TABS** – Pantheon / Stammbaum umschalten
   - **CARDS** – Karten rendern, Domänen-Filter
   - **MODAL** – Detailansicht öffnen/schließen (`openM`, `closeM`)
   - **TREE** – SVG-Stammbaum mit Pan/Zoom (`buildTree`, wird lazy beim ersten Tab-Wechsel aufgebaut)

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
  "parents": ["kronos","rhea"],      // IDs der Elternteile (müssen im Array existieren)
  "pairs": ["hera"]                  // IDs von Partnern (werden als gestrichelte Linie gezeigt)
}
```

Gültige `cats`-Werte: `olympier`, `himmel`, `meer`, `unterwelt`, `krieg`, `weisheit`, `liebe`, `titan`

## Stammbaum-Layout

Götter werden manuell in `LAYERS`-Array (in `buildTree()`) auf Generationsebenen verteilt. Neue Götter müssen dort eingetragen werden, sonst erscheinen sie nicht im Stammbaum.

```js
var LAYERS = [
  ['rhea', 'kronos', 'prometheus', 'nike'],   // Gen 0
  ['zeus', 'hera', ...],                       // Gen 1
  ['athene', 'apollon', ...],                  // Gen 2
  ['eros']                                     // Gen 3
];
```

## Entwicklung

Keine Build-Pipeline. Direkt `start.html` im Browser öffnen und nach Änderungen neu laden (F5).

Für DevTools-Debugging: alle Variablen leben in einer IIFE, sind also nicht global erreichbar. Temporär `window.gods = gods` einfügen zum Inspizieren.
