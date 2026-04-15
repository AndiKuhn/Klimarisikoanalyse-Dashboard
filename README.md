# Klimarisikoanalyse-Dashboard
Eine einfache Schnittstelle um Daten aus gängigen Klimadatenbanken abzurufen. 


# ClimateRisk Data Hub — Klimarisikoanalyse

> Ein browser-basiertes Open-Source-Tool für den Zugriff auf Klimadaten via API.  
> Kein Backend. Kein Build-Schritt. Eine HTML-Datei.

![License: MIT](https://img.shields.io/badge/License-MIT-cyan)
![Status: Beta](https://img.shields.io/badge/Status-Beta-yellow)
![Zero Dependencies](https://img.shields.io/badge/Dependencies-0-brightgreen)

---

## Inhalt

- [Überblick](#überblick)
- [Für wen ist dieses Tool?](#für-wen-ist-dieses-tool)
- [Datenquellen im Detail](#datenquellen-im-detail)
- [Was geht ohne API-Key?](#was-geht-ohne-api-key)
- [Schnellstart](#schnellstart)
- [API-Keys einrichten](#api-keys-einrichten)
- [Features](#features)
- [Architektur](#architektur)
- [CORS — was funktioniert im Browser, was nicht?](#cors--was-funktioniert-im-browser-was-nicht)
- [Klimaprojektionen (CMIP6 / CORDEX / IPCC Atlas)](#klimaprojektionen-cmip6--cordex--ipcc-atlas)
- [Skalierung & Weiterentwicklung](#skalierung--weiterentwicklung)
- [Datenquellen-Lizenzen](#datenquellen-lizenzen)
- [Mitwirken (Contributing)](#mitwirken-contributing)
- [Lizenz](#lizenz)

---

## Überblick

**ClimateRisk Data Hub** ist ein standalone HTML-Interface für die Abfrage, den Export und die Weiterverarbeitung von Klimadaten aus sieben Datenquellen. Es richtet sich an alle, die im Kontext Klimarisikoanalyse, ESG-Reporting oder Nachhaltigkeitsberatung auf offene Klimadaten zugreifen müssen — ohne Python-Kenntnisse, ohne Server-Setup, ohne Installation.

Das Tool bündelt die Zugänge zu folgenden Quellen in einer Oberfläche:

| Quelle | Dienst | API-Typ | Auth erforderlich |
|--------|--------|---------|-------------------|
| **Copernicus CDS** | ERA5 Reanalysis, CMIP6 Projektionen, CORDEX, IPCC Atlas | REST | ✅ Personal Access Token |
| **Copernicus ADS** | CAMS Luftqualität, Treibhausgase, Aerosole | REST | ✅ Personal Access Token (identisch mit CDS) |
| **Copernicus Marine** | Ozeanographie, Meeresspiegel, SST, Salinität | Python CLI | ✅ Eigener CMEMS-Account |
| **Copernicus Land** | CORINE, Bodenversiegelung, Baumkronenbedeckung, Urban Atlas | WMS/WCS | ❌ Kein Key nötig |
| **DWD / Bright Sky** | Wetter historisch + aktuell, Regenradar, Warnungen | REST (JSON) | ❌ Kein Key nötig |
| **OpenStreetMap / Overpass** | Geodaten: Infrastruktur, Schutzgebiete, Gewässer | REST (JSON) | ❌ Kein Key nötig |
| **EU Open Data Portal** | Katalogsuche + SPARQL über data.europa.eu | REST + SPARQL | ❌ Kein Key nötig |

---

## Für wen ist dieses Tool?

- **Nachhaltigkeitsmanager:innen & ESG-Berater:innen**, die Klimadaten für Risikoanalysen, CSRD-Reporting oder Standortbewertungen benötigen
- **KMU**, die ohne Data-Engineering-Team auf offene Klimadaten zugreifen wollen
- **Forscher:innen & Studierende**, die einen schnellen Einstieg in Copernicus, DWD und OSM-Daten suchen
- **Entwickler:innen**, die ein Frontend als Ausgangspunkt für eigene Klima-Tools nutzen wollen

---

## Datenquellen im Detail

### 🌡️ Copernicus Climate Data Store (CDS)

Der CDS ist die zentrale Plattform des Copernicus Climate Change Service (C3S) und bietet Zugang zu über 140 Datensätzen mit 3,8 Petabyte Klimadaten.

**Im Tool integriert:**

- **ERA5 Reanalysis** (`reanalysis-era5-single-levels`) — globale Klimadaten seit 1940, stündlich. Variablen: Temperatur 2m, Niederschlag, Wind 10m, Oberflächendruck, Meeresoberflächentemperatur, Bodentemperatur, Schneehöhe, Abfluss, Verdunstung. Produkt-Typen: Reanalysis, Ensemble Members/Mean/Spread, Monatsmittel.
- **CMIP6 Global** (`projections-cmip6`) — 51 Variablen, 40+ Klimamodelle, 9 SSP-Szenarien, 1850–2100. Siehe [Klimaprojektionen](#klimaprojektionen-cmip6--cordex--ipcc-atlas).
- **CORDEX Regional** (`projections-cordex-domains-single-levels`) — regionale Downscaling-Projektionen mit höherer Auflösung als CMIP6 global.
- **IPCC AR6 Atlas** (`projections-climate-atlas`) — qualitätsgeprüfte Daten aus dem IPCC Interactive Atlas, 22 Variablen/Indizes.
- **Eigene Abfrage** — beliebiges CDS-Dataset mit frei editierbarem JSON-Request.

API-Dokumentation: [cds.climate.copernicus.eu/how-to-api](https://cds.climate.copernicus.eu/how-to-api)

### 🌫️ Atmosphere Data Store (ADS / CAMS)

Copernicus Atmosphere Monitoring Service — Luftqualität, Treibhausgase, Aerosole.

**Datasets:** GHG Reanalysis (monatlich), EAC4 Global Reanalysis, EU Luftqualitäts-Prognose, Radiative Forcings.  
**Variablen:** CO₂, Methan (CH₄), NO₂, Ozon, PM2.5, PM10, SO₂.

Nutzt denselben Personal Access Token wie CDS, aber eine andere Basis-URL (`ads.atmosphere.copernicus.eu`).

### 🌊 Copernicus Marine Service (CMEMS)

Ozeanographische Daten: Meeresspiegel, Wassertemperatur, Strömungen, Salinität.

**Produkte:** Global Ocean Physics (täglich/monatlich), Sea Level (L4), Baltic Sea Physics, NW Shelf SST.  
**Variablen:** Wassertemperatur (thetao), Salinität (so), Meeresspiegel/SSH (zos), Strömung U/V (uo/vo).

⚠️ CMEMS bietet keinen CORS-fähigen Browser-Zugang. Das Tool generiert die passenden Python-CLI-Befehle für das `copernicusmarine`-Paket.

API-Dokumentation: [data.marine.copernicus.eu](https://data.marine.copernicus.eu)

### 🛰️ Copernicus Land Monitoring (CLMS)

Landbedeckung, Vegetation und Bodenversiegelung über WMS/WCS-Dienste.

**Datasets:** CORINE Land Cover, Bodenversiegelung (Imperviousness), Baumkronenbedeckung, Wasser & Feuchtgebiete, Kleine Gehölzstrukturen, Urban Atlas, EU-DEM Höhenmodell.

Das Tool unterstützt die Eingabe benutzerdefinierter WMS/WCS-Endpoints und liefert GetCapabilities-Abfragen.

Produktkatalog: [land.copernicus.eu/en/products](https://land.copernicus.eu/en/products)

### ⛅ Deutscher Wetterdienst (DWD) via Bright Sky

Kostenloser JSON-API-Zugang zu DWD-Wetterdaten über die Open-Source-Schnittstelle Bright Sky. **Kein API-Key erforderlich.**

**Endpunkte:**
- `/weather` — historische + aktuelle Wetterdaten (Temperatur, Niederschlag, Wind, Bewölkung, Sonnenschein, Luftdruck)
- `/current_weather` — aktuelles Wetter an einer Position
- `/radar` — Regenradar-Daten
- `/alerts` — aktive Wetterwarnungen des DWD

**Ergebnis:** JSON-Daten werden automatisch als Tabelle dargestellt (bis 100 Datensätze).

Bright Sky API-Doku: [brightsky.dev/docs](https://brightsky.dev/docs/)  
DWD Open Data: [opendata.dwd.de](https://opendata.dwd.de/)

### 🗺️ OpenStreetMap / Overpass API

Geodaten-Abfragen über die Overpass-API. **Kein API-Key erforderlich.**

**Vordefinierte Abfragen:**
- Überschwemmungsgebiete (`natural=floodplain`)
- Flüsse & Wasserläufe (`waterway=river|stream`)
- Naturschutzgebiete (`boundary=protected_area`)
- Industriegebiete (`landuse=industrial`)
- Krankenhäuser (`amenity=hospital`)
- Stromtrassen (`power=line`)

Zusätzlich kann jede beliebige Overpass-QL-Abfrage eingegeben werden. Link zu Overpass Turbo zum visuellen Testen integriert.

Overpass API Doku: [wiki.openstreetmap.org/wiki/Overpass_API](https://wiki.openstreetmap.org/wiki/Overpass_API)

### 🇪🇺 EU Open Data Portal

Zugang zum offiziellen Datenportal der Europäischen Union mit zwei Modi:

- **Katalog-Suche (REST)** — Freitextsuche über `data.europa.eu/api/hub/search/datasets`. Ergebnisse als Tabelle mit Titel, Publisher, verfügbare Formate.
- **SPARQL** — direkte Abfrage des Linked-Data-Endpoints für komplexe Suchen.

Doku: [data.europa.eu](https://data.europa.eu/en)

### Umweltbundesamt (UBA)

Das UBA bietet keinen offiziellen REST-API-Endpunkt. Datenexport ist über CSV/Excel auf den Themen-Seiten möglich: [umweltbundesamt.de/daten](https://www.umweltbundesamt.de/daten). Der Verweis ist in der Dokumentations-Seite des Tools enthalten.

---

## Was geht ohne API-Key?

| Funktion | Ohne Key | Mit CDS-Token |
|----------|----------|---------------|
| DWD Wetterdaten abrufen | ✅ | — |
| DWD Wetterwarnungen | ✅ | — |
| OSM Geodaten (Schutzgebiete, Flüsse etc.) | ✅ | — |
| EU Open Data Suche + SPARQL | ✅ | — |
| Copernicus Land (WMS/WCS) | ✅ | — |
| ERA5 Reanalysis | ❌ | ✅ |
| CMIP6 Klimaprojektionen (2015–2100) | ❌ | ✅ |
| CORDEX regionale Projektionen | ❌ | ✅ |
| IPCC AR6 Atlas-Daten | ❌ | ✅ |
| CAMS Luftqualitätsdaten | ❌ | ✅ |
| Marine/Ozean-Daten | ❌ | ✅ (eigener CMEMS-Account) |

**Ein einziger Copernicus-Token reicht für CDS + ADS + alle Projektions-Datasets.** DWD, OSM und EU Open Data laufen komplett ohne Registrierung.

---

## Schnellstart

### Variante 1: Direkt nutzen
1. `index.html` herunterladen (oder Repo klonen)
2. Datei im Browser öffnen (Doppelklick)
3. DWD, OSM und EU Open Data funktionieren sofort

### Variante 2: Via GitHub Pages
1. Repo forken
2. Unter Settings → Pages → Source: `main` branch, Ordner `/ (root)`
3. Tool ist unter `https://<user>.github.io/<repo>/` erreichbar

### Variante 3: Lokaler Webserver
```bash
# Python
python -m http.server 8000

# Node.js
npx serve .
```
Dann `http://localhost:8000` im Browser öffnen.

---

## API-Keys einrichten

### Copernicus CDS / ADS (ein Token für beides)

1. Account erstellen auf [cds.climate.copernicus.eu](https://cds.climate.copernicus.eu)
2. Einloggen und **Personal Access Token** unter [Profil](https://cds.climate.copernicus.eu/profile) kopieren
3. Im Tool in der Sidebar unter 🔑 „API-Schlüssel verwalten" eintragen
4. **Wichtig:** Für jedes Dataset müssen die Lizenzbedingungen auf der CDS-Website separat akzeptiert werden (z.B. für `projections-cmip6` auf der [Dataset-Seite](https://cds.climate.copernicus.eu/datasets/projections-cmip6))

Der Token wird ausschließlich im `localStorage` des Browsers gespeichert und verlässt den Rechner nie (außer als `PRIVATE-TOKEN`-Header bei API-Anfragen an Copernicus).

### Copernicus Marine (CMEMS)

1. Account erstellen auf [data.marine.copernicus.eu](https://data.marine.copernicus.eu)
2. Python-Toolbox installieren: `pip install copernicusmarine`
3. Login: `copernicusmarine login`
4. Das Tool generiert die CLI-Befehle — Browser-Direktzugriff ist wegen fehlender CORS-Header nicht möglich

### EU Open Data Portal

Für die Katalogsuche wird kein Key benötigt. Ein optionaler API-Key kann im Tool hinterlegt werden, ist aber für die Grundfunktionen nicht erforderlich.

---

## Features

### Datenabfrage
- **7 Datenquellen** in einer Oberfläche (CDS, ADS, Marine, Land, DWD, OSM, EU Data)
- **3 Klimaprojektions-Datasets** (CMIP6 Global, CORDEX Regional, IPCC AR6 Atlas)
- **9 SSP-Szenarien** für CMIP6 (SSP1-1.9 bis SSP5-8.5 inkl. Overshoot)
- **40+ Klimamodelle** auswählbar, gruppiert nach Region (EU-relevant, empfohlen, weltweit)
- **~50 Klima-Variablen** über alle Datasets (Temperatur, Niederschlag, Wind, Strahlung, Ozean, Eis)
- **Vordefinierte Overpass-Abfragen** für klimarelevante Geodaten
- **EU-Datenkatalog mit SPARQL-Zugang**
- **Region voreingestellt** auf Deutschland (BBOX 55°N, 5°W, 47°S, 16°E) — frei anpassbar

### Werkzeuge
- **Batch-Abfrage** — mehrere Quellen gleichzeitig mit Fortschrittsanzeige
- **Python-Code-Generator** — für jede CDS/ADS/CMIP6/CORDEX/Atlas-Abfrage per Klick kopierbarer Python-Code
- **Request-Preview** — Voransicht des API-Requests vor dem Absenden
- **Abfrage-Historie** — die letzten 50 Abfragen (Zeitstempel, Quelle, Parameter, Status), lokal gespeichert
- **Export** — JSON, CSV, Zwischenablage für alle Ergebnisse
- **DWD-Tabelle** — automatische Tabellendarstellung der Wetterdaten

### Technisch
- **Zero Dependencies** — eine HTML-Datei, kein Build, kein Framework
- **Responsive Design** — Sidebar-Layout auf Desktop, gestapelt auf Mobile
- **API-Key-Management** — localStorage-basiert, kein Server-Kontakt
- **Job-Polling** — CDS-Jobs werden automatisch alle 5 Sek. auf Status geprüft
- **Inline-Dokumentation** — API-Referenz mit Endpoints, URLs und Links zu offiziellen Dokumentationen

---

## Architektur

```
climate-risk-data-hub/
├── index.html          ← Gesamte Applikation (HTML + CSS + JS)
├── README.md           ← Diese Datei
├── LICENSE             ← MIT-Lizenz
└── (optional)
    ├── CONTRIBUTING.md
    └── .github/
        └── FUNDING.yml
```

### Aufbau der `index.html`

```
index.html (~2.500 Zeilen)
│
├── <style>  ─── CSS Design-System
│   ├── CSS Custom Properties (Farben, Fonts, Spacing)
│   ├── Layout (Sidebar + Main mit sticky Header)
│   ├── Komponenten (Cards, Formulare, Tabs, Tabellen, Log)
│   └── Responsive Breakpoints
│
├── <body>  ─── HTML-Struktur
│   ├── Sidebar
│   │   ├── Logo + Versionsinfo
│   │   ├── Navigation (7 Datenquellen + 3 Werkzeuge)
│   │   ├── API-Key-Manager (collapsible)
│   │   └── Footer
│   │
│   └── Main Content (10 Panels, per Navigation umschaltbar)
│       ├── CDS Panel (3 Tabs: ERA5 / CMIP6 / Custom)
│       │   └── CMIP6 (3 Sub-Tabs: Global / CORDEX / IPCC Atlas)
│       ├── ADS Panel
│       ├── Marine Panel
│       ├── Land Panel
│       ├── DWD Panel (mit Tabellenansicht)
│       ├── OSM Panel (mit Overpass-Editor)
│       ├── EU Data Panel (REST + SPARQL)
│       ├── Batch Panel
│       ├── Historie Panel
│       └── Dokumentation Panel
│
└── <script>  ─── JavaScript
    ├── State Management (results, history)
    ├── Navigation (Panel-Switching, Tabs, Sub-Tabs)
    ├── API-Key-Management (localStorage)
    ├── Logging (timestamped, farbcodiert nach Level)
    ├── API-Clients
    │   ├── CDS (ERA5) — REST + Job-Polling
    │   ├── CMIP6 — REST + Job-Polling + Python-Generator
    │   ├── CORDEX — REST + Python-Generator
    │   ├── IPCC Atlas — REST + Python-Generator
    │   ├── ADS (CAMS) — REST
    │   ├── Marine — Python-CLI-Generator
    │   ├── Land — WMS GetCapabilities
    │   ├── DWD — Bright Sky REST + Tabelle
    │   ├── OSM — Overpass POST
    │   └── EU Data — REST-Suche + SPARQL
    ├── Batch-Engine (sequenzielle Multi-Source-Abfrage)
    ├── Export (JSON, CSV, Clipboard)
    └── Historie (localStorage, max 50 Einträge)
```

---

## CORS — was funktioniert im Browser, was nicht?

### ✅ Direkt im Browser nutzbar

| API | Endpunkt | CORS |
|-----|----------|------|
| DWD / Bright Sky | `api.brightsky.dev` | ✅ Erlaubt |
| Overpass (OSM) | `overpass-api.de/api/interpreter` | ✅ Erlaubt |
| EU Open Data | `data.europa.eu/api/hub/search/datasets` | ✅ Erlaubt |
| EU SPARQL | `data.europa.eu/sparql` | ✅ Erlaubt |

### ⚠️ Eingeschränkt / blockiert

| API | Endpunkt | Problem | Lösung im Tool |
|-----|----------|---------|----------------|
| CDS | `cds.climate.copernicus.eu/api/...` | Kein CORS-Header | 🐍 Python-Code-Generator |
| ADS | `ads.atmosphere.copernicus.eu/api/...` | Kein CORS-Header | 🐍 Python-Code-Generator |
| CMEMS | `data.marine.copernicus.eu` | Kein Browser-API | 🐍 CLI-Befehle generiert |
| CLMS | Variiert je WMS/WCS-Endpoint | Teilweise blockiert | Endpoint-Eingabe + GetCapabilities |

### Lösungen für CORS-geschützte APIs

1. **Python-Code kopieren** (im Tool integriert, empfohlen für Einzelabfragen)
2. **Lokaler CORS-Proxy** (z.B. [cors-anywhere](https://github.com/Rob--W/cors-anywhere))
3. **Backend-Proxy** für Produktionsumgebungen (z.B. auf Hetzner mit Coolify/Node.js)
4. **Browser-Extension** (nur für Entwicklung, nicht für Produktion)

---

## Klimaprojektionen (CMIP6 / CORDEX / IPCC Atlas)

Das Tool bietet Zugriff auf drei CDS-Datasets für Zukunftsprojektionen:

### CMIP6 Global (`projections-cmip6`)

Globale Klimaprojektionen aus dem Coupled Model Intercomparison Project Phase 6.

- **Zeitraum:** 1850–2100 (Historical + Szenarien)
- **Experimente:** Historical, SSP1-1.9, SSP1-2.6, SSP2-4.5, SSP3-7.0, SSP4-3.4, SSP4-6.0, SSP5-3.4-OS, SSP5-8.5
- **Modelle:** 40+, darunter MPI-ESM1-2-LR (DE), EC-Earth3 (EU), IPSL-CM6A-LR (FR), HadGEM3 (UK), GFDL-ESM4 (US) und viele weitere
- **Variablen:** ~50 (Temperatur, Niederschlag, Wind, Strahlung, Wolken, Ozean, Meereis)
- **Temporale Auflösung:** monatlich, täglich, fest
- **Level:** Single Levels (Oberfläche) oder Pressure Levels
- **Datenformat:** NetCDF (legacy/4) oder ZIP

### CORDEX Regional (`projections-cordex-domains-single-levels`)

Regionale Downscaling-Projektionen mit höherer Auflösung.

- **Domains:** Europa (EUR-11 / EUR-22), Afrika, Nordamerika, Südamerika, Ostasien, Arktis
- **Experimente:** Historical, RCP 2.6, RCP 4.5, RCP 8.5
- **Variablen:** Temperatur (Mittel/Max/Min), Niederschlag, Wind, Solarstrahlung, Luftdruck, Feuchte, Evapotranspiration
- **Zeitraum:** 1950–2100

### IPCC AR6 Interactive Atlas (`projections-climate-atlas`)

Qualitätsgeprüfte Daten des IPCC Working Group I Atlas.

- **Datenquellen:** CMIP5, CMIP6 und CORDEX
- **22 Variablen und Klimaindizes** inkl. Frosttage, Sommertage, Tropennächte, max. Trockentage, max. 5-Tage-Niederschlag
- **Zeiträume:** 2021–2040, 2041–2060, 2061–2080, 2081–2100

### SSP-Szenarien erklärt

| Szenario | Beschreibung | Erwärmung ~2100 |
|----------|-------------|-----------------|
| **SSP1-1.9** | Sehr niedrige Emissionen, 1.5°C-Pfad | ~1.0–1.8°C |
| **SSP1-2.6** | Nachhaltige Entwicklung | ~1.3–2.4°C |
| **SSP2-4.5** | Mittlerer Weg | ~2.1–3.5°C |
| **SSP3-7.0** | Regionale Rivalität, hohe Emissionen | ~2.8–4.6°C |
| **SSP5-8.5** | Fossil getriebene Entwicklung | ~3.3–5.7°C |

---

## Skalierung & Weiterentwicklung

Das Tool ist bewusst als Single-File-App gebaut, um die Einstiegshürde maximal niedrig zu halten. Für den Ausbau gibt es verschiedene Pfade:

### Kurzfristig (ohne Server)
- GitHub Pages für das Hosting
- Weitere Overpass-Presets (z.B. Deiche, Kläranlagen, Solarparks)
- Kartenvisualisierung für OSM-Ergebnisse (z.B. Leaflet)
- Diagramme für DWD-Zeitreihen (z.B. Chart.js)

### Mittelfristig (mit leichtem Backend)
- **CORS-Proxy** (Node.js/Python) für CDS/ADS-Direktzugriff im Browser
- **Caching-Layer** für häufige Abfragen (Redis/SQLite)
- **Scheduled Downloads** via GitHub Actions (z.B. täglich DWD-Daten für einen Standort)

### Langfristig (Plattform)
- **Datenbank** (z.B. Supabase, PostgreSQL) für Ergebnis-Persistenz
- **Multi-User** mit Login und gespeicherten Abfragen
- **CLIMADA-Integration** für physische Klimarisikomodellierung
- **Kommerzielle APIs** (Jupiter Intelligence, Climate X) als weitere Datenquellen
- **Report-Generator** — automatische Klimarisiko-Reports aus den Abfrage-Ergebnissen

---

## Datenquellen-Lizenzen

| Quelle | Lizenz | Link |
|--------|--------|------|
| Copernicus CDS / ADS / Marine / Land | Copernicus Licence (frei, Namensnennung) | [Lizenz](https://cds.climate.copernicus.eu/datasets) |
| DWD Open Data | GeoNutzV (Verordnung zur Festlegung der Nutzungsbestimmungen für die Bereitstellung von Geodaten des Bundes) | [GeoNutzV](https://www.dwd.de/DE/service/copyright/copyright_node.html) |
| OpenStreetMap | ODbL (Open Database License) | [ODbL](https://www.openstreetmap.org/copyright) |
| EU Open Data Portal | CC BY 4.0 | [Lizenz](https://data.europa.eu/en) |
| Bright Sky | MIT (Wrapper); Daten unterliegen DWD GeoNutzV | [GitHub](https://github.com/jdemaeyer/brightsky) |
| CMIP6 | Modellspezifisch; CDS-Subset unter Copernicus Terms of Use | [Terms](https://cds.climate.copernicus.eu/datasets/projections-cmip6) |

---

## Mitwirken (Contributing)

Beiträge sind willkommen! Das Projekt ist eine einzelne HTML-Datei — Pull Requests sind entsprechend einfach.

### Mögliche Beiträge

- **Neue Datenquellen** — weitere offene APIs integrieren (z.B. NASA POWER, World Bank CCKP, Sentinel Hub)
- **Overpass-Presets** — weitere klimarelevante OSM-Abfragen
- **Bugfixes** — insbesondere bei API-Änderungen der Quellen
- **Übersetzungen** — Interface-Texte in weitere Sprachen
- **Visualisierungen** — Karten, Diagramme, Zeitreihen
- **Dokumentation** — Tutorials, Use Cases, Beispiel-Workflows

### Ablauf

1. Repo forken
2. Feature-Branch erstellen (`git checkout -b feature/neue-datenquelle`)
3. Änderungen committen
4. Pull Request öffnen

### Code-Stil

- Vanilla HTML/CSS/JS — keine Frameworks, keine Build-Tools
- CSS Custom Properties für Theming
- Funktionsnamen auf Englisch, UI-Texte auf Deutsch
- Kommentare im Code auf Englisch

---

## FAQ

**Werden meine API-Keys irgendwo hochgeladen?**  
Nein. Keys werden ausschließlich im `localStorage` des Browsers gespeichert und nur als HTTP-Header an die jeweilige API gesendet. Es gibt keinen Server, der Daten empfängt.

**Warum funktioniert die CDS-Abfrage nicht im Browser?**  
Die Copernicus-APIs setzen keine CORS-Header. Browser blockieren deshalb die Anfragen. Nutze den 🐍 Python-Code-Button, um den Request als Python-Script zu kopieren und lokal auszuführen.

**Kann ich mehrere CDS-Datasets in einer Abfrage kombinieren?**  
Nicht in einem einzelnen API-Call. Nutze die Batch-Funktion, um mehrere Quellen sequenziell abzufragen.

**Wie groß sind die CMIP6-Downloads?**  
Das hängt stark von Variablen, Modell, Zeitraum und Auflösung ab. Ein einzelnes Modell mit einer Variable über 2015–2100 monatlich liegt typischerweise bei 50–500 MB. Niedrig aufgelöste Modelle (INM-CM4-8, MIROC6) sind deutlich kleiner.

**Kann ich das Tool offline nutzen?**  
Die HTML-Datei selbst funktioniert offline (öffnet sich im Browser). API-Abfragen benötigen natürlich eine Internetverbindung.

---

## Lizenz

MIT — frei nutzbar für alle Zwecke, auch kommerziell.

Siehe [LICENSE](LICENSE) für den vollständigen Lizenztext.

---

Erstellt für Klimarisikoanalysen im DACH-Raum.
