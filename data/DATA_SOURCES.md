# Data Sources

This project does **not** store data locally. All records are fetched at runtime from the
City of Chicago Open Data Portal via the Socrata API. No download or manual setup is required.

---

## Primary Dataset: Chicago Crimes (2001 to Present)

| Property | Value |
|---|---|
| Source | City of Chicago Open Data Portal |
| Dataset ID | `ijzp-q8t2` |
| API endpoint | `https://data.cityofchicago.org/resource/ijzp-q8t2.csv` |
| Format | CSV via Socrata REST API |
| License | Public Domain (City of Chicago) |
| Update frequency | Daily |
| Full dataset size | 8M+ records (2001–present) |
| Rows fetched by notebook | 100,000 (most recent, via `$limit=100000`) |
| Rows after 2023-01-01 filter | 99,924 |
| Working sample | 10,000 (random, `random_state=42`) |

### Schema (22 columns)

| Column | Type | Description |
|---|---|---|
| `id` | int | Unique record identifier |
| `case_number` | str | CPD records division number |
| `date` | datetime | Date and time of incident |
| `block` | str | Partially redacted address |
| `iucr` | str | Illinois Uniform Crime Reporting code |
| `primary_type` | str | Top-level crime category (e.g., THEFT, BATTERY) |
| `description` | str | Sub-category description |
| `location_description` | str | Type of location (STREET, RESIDENCE, etc.) |
| `arrest` | bool | Whether an arrest was made |
| `domestic` | bool | Whether incident was domestic |
| `ward` | float | City council ward number |
| `community_area` | float | Chicago community area number |
| `fbi_code` | str | FBI NIBRS classification code |
| `x_coordinate` | float | Illinois State Plane East projection (feet) |
| `y_coordinate` | float | Illinois State Plane East projection (feet) |
| `year` | int | Year of incident |
| `updated_on` | datetime | Record last updated timestamp |
| `latitude` | float | WGS84 latitude |
| `longitude` | float | WGS84 longitude |
| `location` | str | Lat/lon as formatted string |

### Sample Row

```
id: 13999246
case_number: JJ453453
date: 2025-10-11
block: 043XX N ELSTON AVE
iucr: 1320
primary_type: CRIMINAL DAMAGE
description: TO VEHICLE
location_description: VACANT LOT / LAND
arrest: False
domestic: False
ward: 39.0
community_area: 16.0
fbi_code: 14
year: 2025
latitude: 41.958991
longitude: -87.726887
```

---

## How the Data Is Loaded

```python
import pandas as pd

url = "https://data.cityofchicago.org/resource/ijzp-q8t2.csv?$limit=100000"
df = pd.read_csv(url)
df = df.dropna(subset=['latitude', 'longitude', 'date'])
df['date'] = pd.to_datetime(df['date'])
recent_df = df[df['date'] >= '2023-01-01']
```

No API key is required for read-only access up to the Socrata rate limits.

---

## Quick Setup Verification

After running the first notebook cell, confirm:

```python
print("Records loaded:", len(recent_df))   # expect: ~99,924
print(recent_df.columns.tolist())           # expect: 22 columns including latitude, longitude, date
print(recent_df['date'].min())              # expect: 2023-01-01 or later
```

---

## Reproducibility Notes

- The API returns the most recent 100,000 records at query time. As new crimes are added
  daily, exact record counts will differ from those reported in the notebook outputs.
- The working sample uses `random_state=42` for reproducibility, but row order from the
  API is not guaranteed, so the exact 10k sample may vary across runs.
- Validation window (2025-09-11 → 2025-10-11) and the PAI@5% = 16.23% result reflect
  the state of the dataset as of October 2025.

---

## Privacy

All data is anonymized at the block level (e.g., "043XX N ELSTON AVE"). No individual
identifying information is present. See the
[City of Chicago Data Portal terms](https://www.chicago.gov/city/en/narr/foia/data_disclaimer.html)
for usage guidelines.

> Do not commit any locally cached CSV exports of this dataset to version control.
> Files matching `*.csv` are excluded via `.gitignore`.
