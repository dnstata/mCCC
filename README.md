# Multivariate Control Chart via Copula (mCCC)
The time series from weather station 270009, located in the Atacama Desert (Chile), in which raw values per minute were collected from Jan/2019 to Oct/2026. Sources: Datos Estación Automática – Datos minutarios Mensuales (RE3017) | Ficha de la Estación 270009 – Copiapó Universidad de Atacama.

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
