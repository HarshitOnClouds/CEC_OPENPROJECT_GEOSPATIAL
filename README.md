# CEC_OPENPROJECT_GEOSPATIAL
# Project Report: Oceanic SAR Wind Field Estimator
<img width="1914" height="1029" alt="image" src="https://github.com/user-attachments/assets/d6fd33ac-d721-4a13-bcba-50b73be44d41" />

## Overview
A full-stack web application designed to estimate and visualize ocean wind speeds and directions using satellite radar data. By combining Synthetic Aperture Radar (SAR) imagery with meteorological forecast data, the application calculates high-resolution wind fields over user-selected coastal and oceanic regions.

## Architecture

The system is built on a decoupled frontend-backend architecture:

### Frontend (Next.js & React)
- **Framework**: Next.js (App Router)
- **Mapping**: Leaflet and React-Leaflet
- **Styling**: Tailwind CSS
- **Functionality**: Provides a dark-themed, interactive dashboard. Users can pan across a global map, draw an Area of Interest (AOI) bounding box, and select specific dates. The frontend communicates with the backend via API proxy routes to avoid CORS issues and dynamically overlays the resulting wind field images directly onto the map.

### Backend (FastAPI & Python)
- **Framework**: FastAPI
- **Data Pipeline**: Google Earth Engine (GEE) Python API
- **Processing**: NumPy, SciPy, Matplotlib
- **Functionality**: Receives bounding box coordinates, fetches relevant satellite and weather data, applies geophysical models to compute wind vectors, and returns base64-encoded visualizations to the frontend.

## Data Sources

1. **Sentinel-1 SAR (Copernicus)**: Used to retrieve C-band Ground Range Detected (GRD) imagery. We specifically extract the VV polarization backscatter (`sigma0`) and the radar incidence angle. Radar backscatter correlates directly with ocean surface roughness, which is driven by wind speed.
2. **NOAA GFS (Global Forecast System)**: Because radar backscatter cannot definitively determine wind direction, we fetch the 10-meter U and V wind components from the GFS model (`NOAA/GFS0P25`) to serve as our prior wind direction.
3. **MOD44W (Global Water Mask)**: Used to accurately identify and isolate ocean pixels from landmasses.

## Core Algorithm: CMOD5.n

The calculation relies on the CMOD5.n Geophysical Model Function (GMF). The model mathematically relates radar backscatter (`sigma0`), incidence angle, and relative wind direction to the neutral wind speed at 10 meters above the sea surface. 

Because the standard CMOD5.n function is a forward model (calculating expected backscatter from wind speed), we implemented a numerical inversion algorithm. The inversion iteratively adjusts wind speed estimates until the calculated backscatter matches the observed Sentinel-1 backscatter, resolving the true wind speed for each pixel.

## Key Engineering Decisions & Optimizations

- **Composite Wind Fields**: Sentinel-1 swaths often do not cover an entire user-selected bounding box. To prevent visual clipping, the backend generates a composite field: it prioritizes the high-resolution SAR wind speeds where available, and falls back to the GFS wind speeds for the remainder of the bounding box.
- **Land Masking & Erosion**: Coastal boundaries often create "mixed pixels" containing both land and water, leading to artificially high backscatter and wildly inaccurate coastal wind estimates. We utilized SciPy's `binary_erosion` on the Earth Engine water mask to buffer the coastlines by 2 pixels, effectively eliminating these artifacts.
- **Transparent Matplotlib Overlays**: Standard plotting libraries generate figures with margins, axes, and solid backgrounds, which misalign when overlaid onto web maps. The backend is configured to generate a strictly bounded, 100% transparent PNG containing only the data heatmap and direction vectors, ensuring perfect alignment when stretched across the Leaflet bounding box.

## Setup & Execution

**Backend Initialization**
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```

**Frontend Initialization**
```bash
cd frontend
npm install
npm run dev
```



<img width="1885" height="1031" alt="image" src="https://github.com/user-attachments/assets/0357b19b-9418-45a9-94ec-8e91fcc3f63b" />
<img width="1902" height="1027" alt="image" src="https://github.com/user-attachments/assets/0adb2c21-acd5-4b56-872f-f297c5449f9e" />

<img width="1911" height="1025" alt="image" src="https://github.com/user-attachments/assets/6d853bee-9c5b-4944-9860-76b58e263887" />
<img width="1909" height="1001" alt="image" src="https://github.com/user-attachments/assets/a92026fb-80e1-4d23-ba93-d4c6af577fd7" />
