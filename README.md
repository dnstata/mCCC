# Multivariate Control Chart via Copula (mCCC)
The time series from weather station 270009, located in the Atacama Desert (Chile), in which raw values per minute were collected from Jan/2019 to Oct/2026 (2,860 days and 2,646 complete cases). Sources: Datos Estación Automática – Datos minutarios Mensuales (RE3017) | Ficha de la Estación 270009 – Copiapó Universidad de Atacama.

| Column |	Source tab → field	| Units |
| ------------- | ------------- | ------------- |
| hr	| Presión y Humedad → hr	| % | 
| wind_speed_kt	| Viento → ff02Minutos (2-minute mean speed) |	knots |
| solar_radiation_wm2	| Radiación → radiacionGlobalInst (instantaneous global radiation)	| W/m² |
| temperature_c	| Temperatura → ts (dry-bulb air temperature) |	°C |

Longest gaps:\
-- 3–29 July 2021 (25 days)\
-- 20 May – 9 June 2021 (20 days)\
-- 15–28 October 2025 (12.6 days, the modem outage in the station log)\
-- About 8 days each in late December 2019, early January 2020, and June–July 2022\
-- In total, 23 gaps last a day or longer.

ENSO effect, Local Ocean and Marine Layer information was also analyzed.
| Group | New columns | Source |
|---|---|---|
| ENSO phase | `oni`, `enso_phase` (El Niño / Neutral / La Niña) | NOAA CPC Oceanic Niño Index, official ±0.5 °C definition; monthly, applied to every day of the month |
| ENSO strength | `nino34_anom_lag1`, `nino12_anom_lag1` | Daily SST anomaly, °C, averaged over the Niño 3.4 and Niño 1+2 regions, from NOAA OISST v2.1 (day *t*−1) |
| Local ocean | `sst_coast_lag1`, `sst_coast_anom_lag1` | Daily SST and anomaly, °C, over the coast off Copiapó–Caldera, same OISST product (day *t*−1) |
| Marine layer | `inv_base_m_lag1`, `inv_top_m_lag1`, `inv_strength_c_lag1`, `t850_c_lag1`, `lts_k_lag1` | Antofagasta radiosonde (Cerro Moreno, daily 12 UTC): inversion base and top heights, strength of the temperature jump, 850 hPa temperature, lower-tropospheric stability (day *t*−1) |

Additionally, daily flags marking camanchaca (marine low stratus or fog) as present (1) or absent (0) over Copiapó were obtained from the Advanced Baseline Imager (ABI) on the GOES-East geostationary satellite (75.2° W): GOES-16 until 7 April 2025 and GOES-19, which replaced it as GOES-East, from 8 April 2025. The unit of analysis is a ~6 × 6 km box centred on the city, observed once per hour on daylight mornings. All inputs come from NOAA's public archives on Amazon S3 (buckets `noaa-goes16` and `noaa-goes19`, anonymous read access). Both satellites deliver the same products on the same ABI fixed grid, so the box falls on the same pixels throughout.

| Product | Variable used | Resolution | Period used | Role |
| ------- | ------------- | ---------- | ----------- | ---- |
| ABI-L2-ACMF (Clear Sky Mask) | BCM: binary cloud mask, 0 clear / 1 cloudy | 2 km | 2 Dec 2019 – 30 Oct 2026 | Cloud presence for labels |
| ABI-L2-ACHAF (Cloud Top Height) | HT: cloud-top height, m | 10 km | 2 Dec 2019 – 30 Oct 2026 | Separates low cloud from cirrus |
| ABI-L2-CMIPF band C02 | CMI: reflectance factor, 0.64 µm | 0.5 km | 1 Jan 2019 – 30 Oct 2026 | Classifier feature (visible) |
| ABI-L2-CMIPF band C07 | CMI: brightness temperature, 3.9 µm, K | 2 km | 1 Jan 2019 – 30 Oct 2026 | Classifier feature (shortwave IR) |
| ABI-L2-CMIPF band C13 | CMI: brightness temperature, 10.3 µm, K | 2 km | 1 Jan 2019 – 30 Oct 2026 | Classifier feature (clean longwave IR) |
| ABI-L2-CMIPF band C15 | CMI: brightness temperature, 12.3 µm, K | 2 km | 1 Jan 2019 – 30 Oct 2026 | Classifier feature (dirty longwave IR) |

The archive start dates were checked by listing the bucket. The GOES-16 ACMF and ACHAF folders for 2019 contain only days 336–365, so the Level-2 labels exist from 2 December 2019; earlier days are flagged by the visible/IR classifier. The Cloud and Moisture Imagery (CMIPF) covers the whole study period. Scan cadence changed during the period: ABI Mode 3 (full disk every 15 min) until early April 2019, then Mode 6 (every 10 min). Taking the first scan of each hour makes the sampling identical under both modes. A flag could be assigned on 2,849 of 2,860 days; the other 11 had too few usable daylight scans. Camanchaca was present on 1,153 days (40.5%).

The satellite flags were joined by date to the daily weather summaries from the Dirección Meteorológica de Chile station Copiapó Universidad de Atacama (code 270009), computed from its 1-minute records. The merged dataset covers 2,860 days, 2019-01-01 to 2026-10-30, with every day present in both sources; 2,646 are complete cases. Its structure is:

| Feature | Unit | Description |
|---|---|---|
| `date` | YYYY-MM-DD | Local calendar day in Copiapó (America/Santiago time, daylight saving included). |
| `camanchaca` | 0 (NO) / 1 (YES) | Camanchaca occurrence on day *t* from GOES-16/GOES-19 (see `satellite`): 1 if at least 2 morning daylight scans show low cloud over a ~6 × 6 km box on the city (≥ 50% cloudy, median cloud-top height ≤ 4,000 m). |
| `hr_q10` | % | 10th percentile of 1-minute relative humidity on day *t*. |
| `hr_q90` | % | 90th percentile of 1-minute relative humidity on day *t*. |
| `hr_min_lag1` | % | Minimum 1-minute relative humidity on day *t*−1. |
| `hr_max_lag1` | % | Maximum 1-minute relative humidity on day *t*−1. |
| `hr_med_lag1` | % | Median 1-minute relative humidity on day *t*−1. |
| `wind_min_lag1` | kt | Minimum wind speed on day *t*−1 (2-minute mean, 10 m height). |
| `wind_max_lag1` | kt | Maximum wind speed on day *t*−1 (2-minute mean, 10 m height). |
| `wind_med_lag1` | kt | Median wind speed on day *t*−1 (2-minute mean, 10 m height). |
| `temp_min_lag1` | °C | Minimum 1-minute air (dry-bulb) temperature on day *t*−1. |
| `temp_max_lag1` | °C | Maximum 1-minute air (dry-bulb) temperature on day *t*−1. |
| `temp_med_lag1` | °C | Median 1-minute air (dry-bulb) temperature on day *t*−1. |
| `rad_min_lag1` | W/m² | Minimum 1-minute instantaneous global solar radiation on day *t*−1. |
| `rad_max_lag1` | W/m² | Maximum 1-minute instantaneous global solar radiation on day *t*−1. |
| `rad_med_lag1` | W/m² | Median 1-minute instantaneous global solar radiation on day *t*−1 (night minutes included). |
| `oni` | °C | NOAA CPC Oceanic Niño Index for the month of day *t* (3-month running mean of the Niño 3.4 SST anomaly, 1991–2020 base). |
| `enso_phase` | — | `El Nino`, `La Nina` or `Neutral` for the month of day *t*: ONI ≥ +0.5 °C (≤ −0.5 °C) for at least 5 consecutive overlapping seasons (CPC definition). |
| `nino34_anom_lag1` | °C | Niño 3.4 SST anomaly (5°N–5°S, 170°W–120°W) on day *t*−1, from NOAA OISST v2.1 daily, relative to a 1991–2020 climatology. |
| `nino12_anom_lag1` | °C | Niño 1+2 SST anomaly (0–10°S, 90°W–80°W) on day *t*−1, same source and base period. |
| `sst_coast_lag1` | °C | Sea surface temperature off Copiapó–Caldera (17 OISST cells, 26.5–28°S, 71.5–70.5°W) on day *t*−1. |
| `sst_coast_anom_lag1` | °C | Anomaly of `sst_coast_lag1` relative to its 1991–2020 climatology. |
| `inv_base_m_lag1` | m | Height above sea level of the base of the strongest temperature inversion below 3,000 m (base below 2,500 m), Antofagasta 12 UTC sounding (WMO 85442, NOAA IGRA v2) on day *t*−1; NA when there is no inversion. |
| `inv_top_m_lag1` | m | Height of the top of that inversion. |
| `inv_strength_c_lag1` | °C | Temperature increase across the inversion (0 when there is none). |
| `t850_c_lag1` | °C | Air temperature at 850 hPa, same sounding. |
| `lts_k_lag1` | K | Lower-tropospheric stability: potential temperature at 700 hPa minus at the surface, same sounding. |
| `n_hr_day` | minutes | Number of valid 1-minute humidity readings on day *t* (1,440 on a full day; 1,380 or 1,500 on daylight-saving change days). |
| `n_min_lag1` | minutes | Smallest number of valid 1-minute readings among the four variables on day *t*−1. |
| `complete_case` | TRUE/FALSE | TRUE when `camanchaca` and all 14 weather summaries are present. |
| `complete_case_ext` | TRUE/FALSE | TRUE when `complete_case` is TRUE and all the features above are present. |
| `source` | — | Method behind `camanchaca`: `L2 cloud mask + cloud-top height` (NOAA cloud products, from 2019-12-02), `visible/IR classifier` (before that date) or `missing`. |
| `satellite` | — | Satellite used: GOES-16 until 2025-04-07, GOES-19 from 2025-04-08. |
| `camanchaca_classifier` | 0/1 | Camanchaca flag from the visible/IR classifier for every day, giving a single method with no break at 2019-12-02. |
| `n_daylight_scans` | count | Usable hourly morning daylight scans on day *t* (11–16 UTC, sun more than 10° above the horizon). |
| `n_low_scans` | count | Number of those scans showing low cloud over the city. |
| `min_top_height_m` | m | Lowest median cloud-top height among day *t*'s daylight scans (cloud-product days only). |
