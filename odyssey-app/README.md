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
- **Handoff:** one tap opens Google Maps with the route pinned to Athena's path via up to 8 waypoints (walking) — the only way to make Google follow our route; stops are shown, unavoidable.
- **SOS:** 999 / 101 dial buttons, copy-my-location, official guidance links.
- **Honesty:** `node test_core.js` verifies routes on the real data;
  `data/data_manifest.json` lists every source with its licence and fetch
  date; `data/model.json` is the full model card (AUC 0.971 vs 0.866
  recency baseline on held-out Aug 2026).

Regenerate all data + retrain the model: see `../odyssey-city-data`.
Full write-up and demo script: `../GUIDE.md` and `../odyssey-city-data/README.md`.

Licences: OSM ODbL (basemap, network) · Police UK & EA OGL v3 · Leaflet BSD · code MIT.
