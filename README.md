# kommundatakarta

An interactive Flask web app for exploring Kolada KPI data across Swedish
municipalities. The app renders a Leaflet map of Sweden, lets you search for a
KPI, choose a year, and colors each municipality by its value. Hovering a
municipality shows its current value and a small historical trend chart.

## What It Does

- Displays Swedish municipality boundaries from a local GeoJSON file.
- Searches Kolada KPI groups and KPI titles through the Kolada API.
- Loads KPI values for all municipalities for a selected year.
- Colors municipalities by percentile buckets, from red to green.
- Lets you toggle whether high values or low values should be treated as good.
- Shows a hover tooltip and a historical line chart for each municipality.
- Caches the Kolada KPI catalog in memory after the first search request.

## Tech Stack

- Python with Flask
- Requests for server-side Kolada API calls
- Flask-CORS
- Leaflet for the map UI
- Chart.js for historical trend charts
- Local JSON/GeoJSON files for municipality IDs and map boundaries

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       └── main_kommundata.yml
├── app.py
├── requirements.txt
├── templates/
│   └── index.html
└── static/
    ├── css/
    │   └── styles.css
    ├── data/
    │   ├── municipality_id.json
    │   └── sweden_municipalities.geojson
    └── js/
        └── main.js
```

## Setup

Create and activate a virtual environment, then install the dependencies:

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Run the app:

```sh
python app.py
```

Open the local Flask URL shown in the terminal, usually:

```text
http://127.0.0.1:5000
```

## Usage

1. Click the KPI search field to load the first page of available KPIs, or type
   to filter by KPI title or KPI group.
2. Select a KPI from the dropdown.
3. Pick a year. The app defaults to the previous year when available.
4. Use the "High is Good" / "Low is Good" toggle to reverse the map colors.
5. Hover over a municipality to see its value and historical trend.

## Data Sources

The app combines local geography with live Kolada API responses:

- `static/data/sweden_municipalities.geojson` contains municipality map
  boundaries used by Leaflet.
- `static/data/municipality_id.json` contains municipality IDs and filters the
  app to entries with type `K`.
- `http://api.kolada.se/v2/kpi_groups` is used to discover KPI groups and their
  member KPIs.
- `http://api.kolada.se/v2/data/kpi/{kpi_id}/municipality/{municipality_ids}/year/{year}`
  is used to fetch values for all municipalities for a selected KPI and year.
- `http://api.kolada.se/v2/data/kpi/{kpi_id}/municipality/{municipality_id}/year/{years}`
  is used to fetch historical values for one municipality.

The browser also loads Leaflet and Chart.js from public CDNs, so both the server
and client need network access for the full app to work.

## Flask Routes

| Route | Purpose |
| --- | --- |
| `/` | Renders the map page. |
| `/kpi_data` | Returns the current year and five previous years. |
| `/municipality_ids` | Returns local municipality records filtered to type `K`. |
| `/municipality_data/<kpi_id>?year=<year>` | Returns Kolada KPI values for all municipalities for one year. |
| `/search_kpis?term=<term>&page=<page>` | Searches cached Kolada KPI metadata with 50 results per page. |
| `/historical_data/<kpi_id>/<municipality_id>` | Returns total-value history from 2017 through the current year. |

## Notes

- The KPI catalog cache is in memory only. Restarting Flask clears it.
- Values are taken from Kolada entries where `gender` is `T`, meaning total.
- The map color buckets are percentile based, so colors show relative position
  among municipalities for the selected KPI and year.
- The app is intended for local exploration and currently runs with
  `debug=True` when started through `python app.py`.

## Deployment

The repository includes a GitHub Actions workflow at
`.github/workflows/main_kommundata.yml`. It builds the Python app on Ubuntu with
Python 3.12, packages the repository into `release.zip`, and deploys it to the
Azure Web App named `kommundata` when changes are pushed to `main` or when the
workflow is triggered manually.
