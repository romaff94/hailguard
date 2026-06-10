# Study Area — Seveso/Brianza Corridor

## Geographic Context

The **Seveso/Brianza corridor** is a sub-region of Lombardy (northern Italy)
situated between the foothills of the Prealpi Lombarde and the northern Po Valley.

**Station coordinates:** approx. 45.6°N / 9.2°E (Brianza plateau)  
**Elevation:** ~250–300 m a.s.l.

---

## Bounding Box (ESWD request)

| Parameter | Value |
|---|---|
| Latitude | 43.5°N – 47.0°N |
| Longitude | 6.5°E – 14.0°E |
| Period | 1994–2026 |
| QC level | QC1 minimum |
| Event types | Hail (large hail priority), no avalanche |

This box covers the full Alpine–Po Valley system to capture upstream
initiation regions, not just the immediate target area.

---

## Known Hail Tracks Affecting the Station

### Track A — Po Valley convective initiation
- **Trigger:** afternoon CAPE buildup (typically 14:00–18:00 local)
- **Favoured months:** June–August
- **Characteristics:** multicell clusters, moderate hail, short duration

### Track B — Nocturnal Swiss supercell propagation *(Pattern B)*
- **Origin:** Locarnese / Verbano-Cusio-Ossola (Switzerland/Piedmont border)
- **Propagation:** ENE — Malpensa → Brianza → Bergamo
- **Timing:** typically 21:00–03:00 local
- **Characteristics:** organized supercells, large hail (>3 cm documented),
  long track, systematically underscored by standard thermodynamic indices
- **Correction applied in HailGuard:** ×2.5 multiplier on Pattern B signature

### Track C — Alpine foehn descent
- **Trigger:** Ticino valley pressure gradient
- **Characteristics:** rapid onset, mixed precipitation type

---

## Why ESWD Data Is Critical

Standard NWP indices (CAPE, K-Index, shear) capture Track A well.  
Pattern B events are the **primary false-negative failure mode** of
current `hail_risk.h` implementation.

ESWD QC1 events for 1994–2026 in the study bounding box will be used to:

1. Quantify Pattern B event frequency and intensity distribution
2. Derive statistical weights for each risk factor via `bootstrap_weights.py`
3. Validate the ×2.5 Pattern B multiplier against ground truth
4. Build the `eswd_matched` dataset (events ±3h from model output)

**No ML inference.** All processing runs on ESP32 (320 KB usable RAM).
Weights are compile-time constants derived offline.

---

## Data Handling Commitments

- ESWD data will **not** be redistributed or published
- Used exclusively for offline weight derivation; no raw events in repo
- All derived weights published under CC BY-NC 4.0
- Full methodology documented and reproducible
