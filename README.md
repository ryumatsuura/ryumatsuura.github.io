# Destination Selector ✈️

A self-hosted web app that randomly selects a trip destination within a country of your choice, converted from Streamlit to FastAPI.

---

## 🚀 Live Demo

👉 **[destination-selector-ryu.fly.dev](https://destination-selector-ryu.fly.dev)**

---

## 📊 What this does

- Allows you to choose a country and administrative boundary level (e.g. Prefecture, Town)
- Optionally narrows the selection to within a specific higher-level division (e.g. towns within Tokyo)
- Animates through random candidates before revealing a final destination on an interactive map

---

## 🧩 Project structure

```
destination/
  config.yaml    # Country/boundary metadata
  utils.py       # load_map, animate_selection, extract_higher_divisions, etc.

main.py          # FastAPI app + single-page HTML/JS frontend
Dockerfile       # Container config for deployment
fly.toml         # Fly.io deployment config
pyproject.toml   # Python dependencies
```

---

## 🗺️ Data sources

Administrative boundary maps are provided by [GADM (The Database of Global Administrative Areas)](https://gadm.org/), version 4.1.

This project fetches GADM data directly from GADM servers at runtime — raw data files are not hosted or redistributed within this repository. Use is for non-commercial/academic purposes under the [GADM License](https://gadm.org/license.html).

---

## 🛠️ Local development

### 1. Install dependencies

```bash
uv sync
```

### 2. Run

```bash
uv run uvicorn main:app --reload
```

Open [http://localhost:8000](http://localhost:8000)

---

## 🚢 Deployment (Fly.io)

```bash
fly launch --no-deploy
fly deploy
```

The app will be live at `https://<your-app-name>.fly.dev`.

---

## ⚙️ How it works

The animation logic is split between server and browser:

- **Server** (`POST /api/animate`): loads the GADM GeoDataFrame, optionally filters to a sub-region, picks 10 random destinations, renders each as a matplotlib PNG, and returns them all as base64-encoded images in a single JSON response.
- **Browser**: plays back the frames with a delay that speeds up toward the final reveal, then displays the result.

Additional endpoints:

| Endpoint | Description |
|---|---|
| `GET /api/countries` | List of all available countries |
| `GET /api/boundaries/{country}` | Available boundary levels for a country |
| `GET /api/higher-divisions/{country}/{boundary}` | Higher divisions available for filtering |
| `GET /api/division-items/{country}/{division}` | Named areas within a division (e.g. all prefectures) |

### Why FastAPI and not pure JavaScript?

This app requires a backend because:
- It downloads GADM zip files (~10–50 MB each) via `requests`
- It processes geospatial data with `geopandas` (Python-only)
- It renders maps with `matplotlib`
