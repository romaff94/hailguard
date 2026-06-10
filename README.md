# HailGuard — DIY Hail Nowcasting Station
### Seveso/Brianza Corridor · Lombardy, Italy

> **Independent research project** — Roberto Maffio  
> Contact: [GitHub Issues](https://github.com/rmaffio/hailguard/issues) or via email on request

---

## Overview

HailGuard is a self-contained hail prediction and monitoring station designed for the **Seveso/Brianza corridor** in Lombardy (northern Italy), one of the most hail-prone sub-regions of the Po Valley.

The project combines real-time sensor data, numerical weather model output, and historical ground-truth events to derive **statistically weighted hail risk scores** — without machine learning, by design (see [Methodology](#methodology)).

The station runs autonomously, sends Telegram alerts on elevated hail risk, and logs all data locally to SD card for post-event validation.

---

## Hardware

| Component | Role |
|---|---|
| ESP32 WROOM-32 (38-pin, CP2102) | Main MCU — real-time logic, WiFi, alerts |
| BME280 (I2C, external shield) | Barometric pressure, temperature, humidity |
| AS3935 (CJMCU, SPI) | Lightning distance estimation |
| MicroSD module (SPI) | Local event + sensor log (~10-year capacity) |
| Raspberry Pi 5 (4 GB) | Async analysis node — bulletin parsing, anomaly detection |

**Enclosure:** IP65 box (~150×110×70mm), balcony-mounted, shaded.  
BME280 external via IP68 cable gland with radiation shield.  
AS3935 internal (RF penetrates plastic enclosure, confirmed).

---

## Software Stack

- **Firmware:** PlatformIO / Arduino framework, ~1700 lines across 12 modules
- **Web UI:** LittleFS-hosted, served directly from ESP32
- **Alerts:** Telegram Bot API
- **Edge AI:** ollama + Qwen2.5 1.5B on RPi5 (Italian-language bulletin parsing)
- **APIs:** Open-Meteo Archive/Forecast, ARPA Lombardia, Meteoalarm RSS, Protezione Civile

**Key firmware modules:**

```
main.cpp            — scheduler, WiFi, boot sequence
hail_risk.h         — multi-factor risk scoring (CAPE, K-Index, Zambretti, AS3935)
pressure_history.h  — 24h pressure trend + second derivative
zambretti.h         — Zambretti forecaster adapted for Po Valley
open_meteo.h        — NWP data fetch + parse
auto_verify.h       — post-event validation vs ERA5-seamless archive
feedback_store.h    — local correction log
sd_logger.h         — structured CSV event logging
telegram_alert.h    — alert composition + delivery
```

---

## Methodology

### Why no ML?

- ESP32 has ~320 KB usable RAM — no inference headroom
- Target area produces **< 30 verifiable hail events/year** in available ground-truth data
- Statistical weights derived from historical datasets are **fully auditable and reversible**

### Risk scoring approach

`hail_risk.h` combines:

1. **Thermodynamic indices** — CAPE, K-Index, lifted index (from Open-Meteo)
2. **Dynamic parameters** — CAPE×Shear product, 850–500 hPa wind shear
3. **Local pressure signal** — Zambretti tendency + second derivative (BME280)
4. **Lightning proximity** — AS3935 distance + strike rate
5. **Climatological multipliers** — hour-of-day, month-of-year (Po Valley patterns)
6. **Pattern B override** — nocturnal Swiss supercell tracks (Locarnese/VCO → Malpensa → Brianza) require ×2.5 correction; systematically underscored by standard indices

### Statistical weight derivation

Weights are derived offline via `bootstrap_weights.py` from five layered datasets:

| Dataset | Points | Period | Purpose |
|---|---|---|---|
| `seveso_core` | 9 | 2010–2026 | Primary calibration |
| `lombardia` | 20 | 2010–2026 | Regional validation |
| `nord_italia` | 465 | 2010–2026 | Background climatology |
| `long_period` | 465 | 2000–2026 | Trend baseline |
| `eswd_matched` | TBD | 1994–2026 | Ground-truth event alignment |

The `eswd_matched` dataset requires **ESWD QC1 hail events** for the study area — currently the primary data gap.

---

## Study Area

**Target:** Seveso/Brianza corridor, Lombardy  
**Bounding box for ESWD request:** 43.5–47°N / 6.5–14°E  
**Period of interest:** 1994–2026  
**Quality level:** QC1 minimum  
**Event types:** Hail (large hail preferred), no avalanche

The corridor sits at the convergence of three documented hail tracks:
- **Alpine foehn descent** — Ticino valley outflow
- **Po Valley convective initiation** — afternoon CAPE buildup
- **Pattern B (nocturnal)** — Swiss supercell propagation from Locarnese/Verbano-Cusio-Ossola toward Malpensa and Brianza

---

## Data Sources

| Source | Status | Notes |
|---|---|---|
| Open-Meteo ERA5-seamless | ✅ Downloading | 465 grid points, 2010–2026 |
| ARPA Lombardia (Socrata) | ✅ Acquired | ~22.6M rows, 2023–2025 |
| ERA5 via CDS Copernicus | 🔄 Configured | Script ready, not yet launched |
| ESWD QC1 hail events | ⏳ Requested | Formal request submitted to ESSL |
| CGP Varese PDF bollettini | 🔄 Planned | astrogeo.va.it |
| Linate/LIML hourly soundings | 🔄 Planned | paolociraci.it, back to 1980 |

---

## Project Status

- [x] Hardware assembled and field-tested
- [x] Firmware core complete (pressure, Zambretti, Open-Meteo, alerts)
- [x] SD logging operational
- [x] ERA5-seamless download pipeline running
- [x] ARPA Lombardia dataset acquired
- [ ] `hail_risk.h` — statistical weight derivation (pending ESWD data)
- [ ] `bootstrap_weights.py` — offline calibration script
- [ ] YL-83 rain sensor integration (auto-verification feedback loop)
- [ ] INA219 battery monitoring
- [ ] RPi5 Qwen2.5 bulletin parser

---

## License

[Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)](LICENSE)

Data and derived weights are for **non-commercial research use only**.  
Third-party datasets (ESWD, ERA5, ARPA) retain their original licenses.

---

## Acknowledgements

- [ESSL — European Severe Storms Laboratory](https://www.essl.org) — ESWD database
- [Open-Meteo](https://open-meteo.com) — ERA5-seamless archive API
- [ARPA Lombardia](https://www.arpalombardia.it) — regional meteorological data
- [Copernicus Climate Change Service](https://cds.climate.copernicus.eu) — ERA5 reanalysis
