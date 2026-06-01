# InSAR DEM Generation with Sentinel-1 IW

Automated InSAR processing pipeline for Digital Elevation 
Model (DEM) generation using Sentinel-1 IW products, 
built entirely in Google Colab.

## What this project does
Replicates the full ESA SNAP interferometric processing 
workflow programmatically via Python and bash, eliminating 
the need for manual interaction with the SNAP GUI. The 
pipeline processes two Sentinel-1 acquisitions over the 
same area and generates a terrain-corrected DEM exported 
as GeoTIFF.

## Processing pipeline
1. Data loading from Google Drive (S1A + S1B, July 2019)
2. ESA SNAP 9.0 installation in Colab environment
3. TOPSAR Split — burst selection
4. Apply Orbit File — orbital state vector correction
5. Back Geocoding — coregistration of image pair
6. Interferogram formation + flat-earth phase removal (ESD)
7. Goldstein Phase Filtering — phase noise reduction
8. Phase Unwrapping — SNAPHU external tool
9. Phase to Elevation — phase-to-height conversion
10. Terrain Correction + GeoTIFF export

## Output
- Terrain-corrected DEM in GeoTIFF format
- Intermediate products saved at each processing step

## Tools & Software
- Python (os, subprocess, zipfile, shutil)
- ESA SNAP 9.0 (GPT command-line processor)
- SNAPHU (phase unwrapping)
- Google Colab + Google Drive
- Sentinel-1 IW SLC products (ESA Copernicus)

## Data
Sentinel-1A (2019-07-08) and Sentinel-1B (2019-07-02) 
acquisitions over Erzincan, Turkey. Data available via 
ESA Copernicus Open Access Hub.

## Author
Tania Gianturco
[LinkedIn](https://www.linkedin.com/in/tania-gianturco/) | 
[Tableau Public](https://public.tableau.com/app/profile/tania.gianturco/vizzes) | 
[Kaggle](https://www.kaggle.com/taniabee/discussion)
