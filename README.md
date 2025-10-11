# GeoServer test task

Spin up a tiny GIS stack, publish the provided dataset in GeoServer (backed by PostGIS), expose a small FastAPI API, and render the layer with a React + OpenLayers viewer (legend, scale bar, identify).

---

## ✅ What’s provided (do not modify)

Use these files for seeding PostGIS and styling the GeoServer layer.

../data/regions.geojson #sample dataset (EPSG:4326)

../data/styles/regions.sld #simple SLD for the layer

---

## 🧩 What you need to build

### 1) Dockerized stack

Create `docker-compose.yml` with services:

- `db` — PostgreSQL + PostGIS
- `geoserver` — GeoServer (persistent `data_dir`)
- `api` — Python FastAPI
- `web` — React + TypeScript (Vite) + OpenLayers

**Requirements**

- Expose only what’s needed (GeoServer UI may be exposed during review).
- Provide a `.env.example` with ports, credentials, workspace, etc.

---

### 2) Seed PostGIS (scripted)

Add `scripts/seed.sh` that:

- Loads `/data/regions.geojson` into PostGIS as `public.regions` (`geom` column, **EPSG:4326**).  
  _Tip: use `ogr2ogr` from a GDAL container._
- Ensures SRID is 4326 and creates a GIST index on `geom`.

**Acceptance**

- `SELECT COUNT(*) FROM public.regions;` returns > 0.

---

### 3) Publish layer & style in GeoServer (scripted)

Add `scripts/publish_layer.sh` that (idempotent):

- Creates a **workspace** (e.g., `gis_test`).
- Creates a **PostGIS datastore** (JDBC params).
- Publishes a **layer** for table `public.regions`.
- Uploads **/data/styles/regions.sld** and sets it as **default style**.
- Prints a **WMS GetCapabilities** URL on success.

**Acceptance**

- Capabilities lists the layer.
- `GetMap` returns an image (PNG).
- Default style is applied to the layer.

---

### 4) FastAPI backend

Implement:

- `GET /layers` → returns JSON describing the published layer, including **browser-reachable** WMS URL and dataset bbox:
  ```json
  {
    "name": "regions",
    "title": "Regions",
    "srs": "EPSG:4326",
    "bbox": [minx, miny, maxx, maxy],
    "wms": {
      "url": "http://localhost:8080/geoserver/orangegis_test/wms",
      "layer": "orangegis_test:regions"
    }
  }

### 5) Output

![Screenshot 2025-10-11 at 10.58.47.png](public/Screenshot%202025-10-11%20at%2010.58.47.png)