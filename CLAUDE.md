# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Livestream Command Center is a single-file HTML dashboard used during live severe weather streams (primarily on TikTok/YouTube). It provides chat message templates, real-time NWS weather warnings, SPC convective outlooks, weather lookups, a Weather Intensity Score (WIS) widget, and a storm chaser logging system. The entire app lives in `livestreamcommandcenter.html` — no build system, bundler, or package manager.

## Development

Open `livestreamcommandcenter.html` directly in a browser. No server required, though some features (clipboard API, fetch) need `file://` or `localhost`. All state persists via `localStorage`.

## Architecture

The app is a single HTML file (~3000 lines) with three sections: `<style>`, HTML markup, and `<script>`. There is no framework — it's vanilla JS with DOM manipulation.

### Tab System
The UI is tab-based, controlled by `setFilter(filter, btn)` and `setChatFilter(value)`:
- **Chat Messages** (dropdown): Filterable pre-written messages stored in the `messages[]` array (~90 entries, each with `tag`, `label`, `text`). Click-to-copy to clipboard. Tags: safety, chat, rules, info, general, submit, weatherwise, yallcall, tempest, coverage, engagement.
- **Live Monitor** (`warnings`): Fetches active NWS alerts (tornado/SVR/flood warnings & watches) from `api.weather.gov/alerts/active`, auto-refreshes every 10s. Left sidebar shows the WIS widget and an inline weather lookup. Each alert card has Chat copy, Social copy, and a Leaflet map preview.
- **Chasers** (`chasers`): Storm chaser attendance tracker with start/end stream, per-chaser toggle logging, CSV export, and stream history. Default chaser list in `SC_DEFAULT_CHASERS[]`.
- **Weather Lookup** (`weather`): NWS forecast + SPC risk lookup by city/state or regional query (e.g. "Southeast Missouri"). Uses Nominatim geocoding -> NWS points API -> forecast. Regional queries use `STATE_CENTROIDS` + directional offsets.
- **SPC Outlook** (`spc`): Renders Day 1-3 SPC convective outlook GeoJSON on Leaflet maps. Shows countdown to next SPC update based on `SPC_SCHEDULE_UTC`.

### External APIs (all unauthenticated)
- **NWS** (`api.weather.gov`): Alerts, forecasts, observation stations. User-Agent: `RHY-TikTok-Live-Tool`
- **SPC** (`spc.noaa.gov`): Convective outlook GeoJSON
- **Census TIGERweb** (`tigerweb.geo.census.gov`): City lookups within warning polygons for social copy
- **Overpass API** (`overpass-api.de`): City/town lookups for SPC outlook risk areas
- **Nominatim** (`nominatim.openstreetmap.org`): Geocoding for weather lookups
- **WIS** (`ryanhallyall.com/rhy/wis.json`): Weather Intensity Score feed, refreshes every 20s

### Key Patterns
- **Warning location resolution** (`buildWarningLoc`): Multi-source cascade to extract city names from NWS alert text — headline cities -> "Locations impacted include" -> "storm will be near" -> "Other locations" -> storm position -> county fallback. Each source is logged to console with `[WarningLoc]`.
- **Social copy generation** (`buildSocialCopy`): Formats warnings for social media posts, includes YouTube live link from localStorage, rotates tracking lines, uses polygon-based Census city lookup for NWS-tweet-style formatting.
- **Point-in-polygon** (`pointInPoly`): Ray-casting used for SPC risk lookups and Census city filtering.
- **Timezone handling**: Warning expiry times use the NWS headline's local time when available, otherwise estimate timezone from polygon centroid longitude.

### External Libraries (loaded via CDN)
- Leaflet 1.9.4 (maps)
- html2canvas (WIS screenshot export)
- Google Fonts: Bebas Neue, DM Sans

### CSS Design Tokens
All colors use CSS custom properties on `:root` — `--bg`, `--card`, `--accent` (#fe2c55 red), `--accent2` (#25f4ee teal), `--text`, `--muted`, `--border`. Per-tag colors use `.tag-{name}` classes.
