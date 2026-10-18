# Odyssey — night-safety route planner for Southampton

**Run it:** `python3 -m http.server 8000` in this folder, then open
http://localhost:8000. No backend, no accounts, no keys.

- **Map:** every walking segment coloured by the Athena risk model
  (green low / amber medium / red high / grey-dashed = no lighting data, never
  scored). Click a segment for lighting, real Police UK crime history
  (categories + months), isolation, tide exposure, and sources.
- **Routing:** Safest / Shortest / Balanced. Athena's Dijkstra consumes the
  model's continuous risk probability per segment, with a 12-minute detour
  bound — and says so when a corridor offers no real safety gain.
- **Live:** the panel fetches the real-time Environment Agency tide gauge at
  Woolston and active flood warnings directly in your browser.
- **Turn-by-turn:** shown in-app on Athena's exact path (OSM street names) — no rerouting, no stop markers. The Google Maps handoff pins the route with a single `via:` passthrough street — Google's own dragged-route mechanism, so no stop markers there either. Verified live (Oct 2026, google.com/maps/dir): exactly one via survives Google's URL parsing (any second waypoint turns both into stops; `via:` with coordinates or plus codes is rejected; the legacy `daddr +to:via:` chain is no longer parsed), so the one pin goes on the street where Athena's path most departs from the obvious walk. A separate "Pin exact route" link offers the 8-waypoint version: follows Athena turn-for-turn, at the cost of stop markers.
- **SOS:** 999 / 101 dial buttons, copy-my-location, official guidance links.
- **Honesty:** `node test_core.js` verifies routes on the real data;
  `data/data_manifest.json` lists every source with its licence and fetch
  date; `data/model.json` is the full model card (AUC 0.982 vs 0.866
  recency baseline on held-out Aug 2026).

Regenerate all data + retrain the model: see `../odyssey-city-data`.
Full write-up and demo script: `../GUIDE.md` and `../odyssey-city-data/README.md`.

Licences: OSM ODbL (basemap, network) · Police UK & EA OGL v3 · Leaflet BSD · code MIT.
