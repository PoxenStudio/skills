---
name: openstreet-map
version: 1.0.0
description: "Use when you need OpenStreetMap geocoding or annotated map generation for a set of places."
---

# OpenStreet Map Skill (for OpenClaw)

This skill provides two capabilities:

1. Query one place's location information.
2. Render an annotated map image for multiple places with numbered markers and a legend.

## Prerequisites

- Python 3.10+
- Internet access (Nominatim + OSM tile endpoints)

Optional environment variable:

- `OPENSTRET_HOST`: Override OpenStreet service host (for private/self-host proxy).
- `OPENSTREET_HOST`: Compatibility alias; lower priority than `OPENSTRET_HOST`.

When set, the skill uses:

- Geocode endpoint: `${OPENSTRET_HOST}/search`
- Tile endpoint: `${OPENSTRET_HOST}/{z}/{x}/{y}.png`

## Install

```bash
pip install -r requirements.txt
```

## Capability 1: Query location info

```bash
python tools/openstreet_skill.py locate --query "The Bund Shanghai" --limit 1
```

Output is JSON including `display_name`, `lat`, `lon`, and type metadata.

## Capability 2: Render annotated map

Input JSON file format (array):

```json
[
  {"name": "Point A", "lat": 31.2304, "lon": 121.4737},
  {"name": "Point B", "query": "The Bund Shanghai"}
]
```

Render command:

```bash
python tools/openstreet_skill.py render \
  --points-file examples/points.json \
  --output output/annotated_map.png \
  --width 1200 \
  --height 800
```

Render command with base64 JSON input:

```bash
python tools/openstreet_skill.py render \
  --points-base64 "W3sibmFtZSI6IlBvaW50IEEiLCJsYXQiOjMxLjIzMDQsImxvbiI6MTIxLjQ3Mzd9LHsibmFtZSI6IlBvaW50IEIiLCJxdWVyeSI6IlRoZSBCdW5kIFNoYW5naGFpIn1d" \
  --output output/annotated_map.png
```

`render` now supports either:

- `--points-file` for file input.
- `--points-base64` for direct base64 JSON string input.

Behavior:

- Selects a suitable zoom level to fit all points.
- Downloads OSM tiles and stitches them into one map image.
- Draws numbered markers on each place.
- Appends a legend under the map with `index -> place name (lat, lon)`.
- Saves final image to the specified output path.

## Notes for OpenClaw integration

- Expose these two CLI actions as callable tools in OpenClaw:
  - `locate`: maps to `python tools/openstreet_skill.py locate ...`
  - `render`: maps to `python tools/openstreet_skill.py render ...`
- Return JSON stdout to the caller and treat non-zero exit code as failure.
- Respect OSM usage policy by not sending high-frequency requests.
