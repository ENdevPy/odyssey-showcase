# Odyssey — live-data exemplar for SotonShip

**One sentence:** *We help someone walking home late in Southampton choose
streets the data says are safer, using an AI model trained on real Police UK
street crime and OpenStreetMap lighting, with live Environment Agency flood
and tide data — and we show every number's source, date, and what we don't
know.*

Odyssey is a night-safety route planner for Southampton. Its navigation
engine **Athena** scores every walking segment with a trained risk model,
offers Safest / Shortest / Balanced routes with plain-English explanations,
hands off to Google Maps, and reports live flood conditions. It is a static
web app: no backend, no accounts, no API keys — every data source is public
and credited.

## Repository layout

| Folder | What |
|---|---|
| `odyssey-app/` | The web app (Leaflet vendored, runs from any folder) |
| `odyssey-city-data/` | `build_odyssey_data.py`: fetches all live data, trains the Athena model, emits everything the app loads |
| `odyssey-city-data/entities/` | The generated data files (also copied into `odyssey-app/data/`) |

Run the app:
```
cd odyssey-app
python3 -m http.server 8000     # then open http://localhost:8000
```

Refresh all data + retrain (about 4 minutes, internet needed):
```
cd odyssey-city-data
python3 build_odyssey_data.py
cp entities/*.json ../odyssey-app/data/
```

## Data sources — every number's provenance

| Source | What we use | Licence | Checked |
|---|---|---|---|
| OpenStreetMap via Overpass API | 23,144 walking ways (geometry, names, `lit` tags, types) | ODbL 1.0, © OSM contributors | build date (see `data_manifest.json`) |
| Police UK street-level crime API | 15,258 real recorded street crimes, Feb–Aug 2026, snapped to segments | OGL v3 (data.police.uk) | build date |
| Environment Agency flood-monitoring API | **Live** tide gauge (Woolston E12660) + active flood warnings, fetched live in the browser during use | OGL v3 | every app load |
| OSM nightlife venues | pubs/bars/nightclubs as model features | ODbL | build date |
| Curated safe havens | 15 real staffed places; hours hand-checked Oct 2026 | our curation | Oct 2026 |
| GeoJSON basemap (water/green/rail) | map background, self-contained | © OSM contributors (ODbL) | built by pipeline (step 2/8) |
| (optional) UKHO Admiralty Tidal API | tide predictions; set `TIDAL_API_KEY` to enable | UKHO free tier | build date |

The app itself shows this table live: **Data sources & freshness** panel,
generated from `data_manifest.json`.

## The AI at the core (Athena risk model)

Remove the model and Odyssey has no product: no risk bands, no safest
routing, no explanations. The model is trained by the pipeline on real data:

- **Type:** logistic regression (L2-regularised, 19 standardised features)
- **Labels:** real Police UK street crimes snapped to walking segments
- **Features:** lighting (OSM lit tag / documented assumption / unknown),
  highway type, isolation, tide exposure, nightlife density, distance to
  24/7 venues, and lagged crime counts
- **Honest split:** trained on months up to 2026-07, evaluated on the
  held-out month **2026-08**: **AUC 0.982** vs **0.866** for a baseline that
  predicts from last month's crime alone — the comparison the brief asks for
- **Output:** each segment's predicted probability of a reported street crime
  next month. Bands are **evidence-gated**: high needs BOTH a top-5% model score AND ≥2 street crimes in the last 90 days; medium needs the 75th percentile AND ≥1 recent crime, so an old incident cluster with clean recent months can never be rated high. Incident recency decays exponentially (half-life 1.5 months) via the log1p_decay feature.
- **In the router:** Athena's Dijkstra consumes the continuous probability
  (not just the band) as extra cost per metre, with a 12-minute detour bound

Model card with coefficients, train/test months and honesty notes:
`entities/model.json` (also rendered in the app's coverage panel).

## What's honestly real, and the failure cases we show

- **45% of the walking network has no lighting data.** OSM has no lighting
  survey for most footpaths. We render those grey-dashed and label them
  "risk unknown" — the model never scores them. That's the demo's opening.
- **Police UK blurs crime locations** to a nearby street point. We snap each
  crime to the nearest walking segment (≤120 m, snap distance recorded) and
  say so in the popup.
- **No time-of-day in Police UK data.** The model predicts reported street
  crime generally; lighting and isolation features carry the night weighting.
- **When "safest" can't help, it says so.** In uniformly high-risk areas the
  app states that reported crime is similar on all options.
- **Out of coverage / offline:** outside the Southampton bbox the router says
  so; offline the live EA panel falls back to the dated build snapshot,
  labelled "offline, not live".

## Business case

- **User:** people walking home late at night — Southampton has ~40,000
  students and a night-time economy along Bevois Valley and the waterfront.
- **Who pays:** university student-safety budgets and Students' Union
  (orientation weeks), the council's community-safety team (public-health
  duty), and the Business Improvement District for the night-time economy.
  Universities already fund safety-escort and bus services; Odyssey is a
  per-term licence on top of data they already publish.
- **Running cost:** a static web app + one 4-minute monthly retrain
  (public data, no paid APIs). Effectively £0 infrastructure; the cost is
  the monthly retrain and support.
- **Second city:** the pipeline is already city-parameterised — swap the OSM
  area id, the bbox, and the Police UK sample points and it rebuilds for any
  English/Welsh city. Lighting data quality varies by city, and the unknown
  rendering is honest about that.

## Disclosure (track rules)

Built as an organiser's exemplar ahead of the event with AI assistance
(Base44 Superagent). Everything above was generated by the scripts in this
repo; no proprietary data or restricted systems were used. Teams entering
the track should build their own during the event — this repo shows one way
to hit the bar: real data, AI doing the load-bearing work, an honest model
card, and a live demo.

## Demo script (3 minutes)

1. **Open on the honesty stat.** Grey dashed segments everywhere:
   "45% of Southampton's paths have no lighting data. We show what we don't
   know — every grey line is unscorable, by design."
2. **Shirley → city centre.** Shortest passes **265 reported street crimes**
   (real Police UK data); Athena's Safest route passes **13** for +7 min.
   Read the model card line in the panel: AUC 0.982 vs 0.866 baseline.
3. **Click a red segment** near the centre: real crime categories with the
   Police UK credit and month. Click a grey one: "risk unknown."
4. **Live data moment:** the panel shows the real-time Woolston tide gauge
   reading fetched from the EA seconds ago, and active flood warnings.
5. **St Denys honesty case:** all options are high-risk here; the app says so
   instead of inventing a fake "safe" route.
6. Hand off to Google Maps, report a route, show SOS (999/101 + official
   guidance links).

## Licences & attribution

OpenStreetMap data © OSM contributors, ODbL 1.0. Police UK and Environment
Agency data under OGL v3. Basemap GeoJSON © OSM contributors (ODbL). Leaflet BSD-2-Clause. App code:
MIT. Crime data contains public-sector information licensed under OGL v3.
