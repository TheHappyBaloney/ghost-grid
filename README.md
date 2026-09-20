# Ghost-Grid - The Jeddah that never was

## Simulating the Cancelled 2026 Saudi Arabian GP

## Project Overview

On 14 March 2026, Formula 1 cancelled the Saudi Arabian Grand Prix (17–19 April, Jeddah Corniche Circuit, Round 5) due to the outbreak of the 2026 Iran war and regional security concerns. No replacement race was held.

**ghost-grid** rebuilds that missing race weekend as a probabilistic simulation. Using real 2026-season data (Australia, China, Japan), historical Jeddah results (2021–2025), track-affinity signal from comparable high-speed street circuits, and Bayesian modeling of Jeddah's notoriously high safety-car rate, it runs a Monte Carlo race simulation to answer: *if this race had happened, what does the distribution of outcomes look like?*

The simulation follows the same two-stage structure as a real race weekend: a qualifying simulation first establishes the most likely starting grid per driver, then a race simulation runs from that grid to produce the most likely podiums and finishing positions — rather than assuming a fixed or historical grid going into the race.

The output is not a single predicted winner. It's a probability distribution over grid positions and over finishing positions per driver, a most-likely podium with its associated probability, and a sensitivity analysis showing which stochastic factors (a safety car arriving 10 laps later, a different DNF draw, a different qualifying outcome) would have moved the result.

## Architecture / Pipeline

Ingestion  →  Feature Engineering  →  Pace & Hazard Models  →  Monte Carlo Engine  →  Results
(FastF1,      (2026 pace baseline,     (XGBoost pace model,     (N simulated races,     (Win/podium
 OpenF1,       track similarity        DNF hazard model,         parallelized)           probabilities,
 Jolpica,      index, tyre deg,        PyMC safety-car                                    sensitivity
 Open-Meteo,   SC hazard rate)         posterior)                                          analysis)
 Pirelli)

## Data Sources

| Source | What it provides |
| --- | --- |
| [FastF1](https://docs.fastf1.dev/) | Lap times, sector times, telemetry, tyre stints, weather, race-control messages for 2026 completed rounds and 2021–2025 Jeddah/Baku/Melbourne sessions |
| [OpenF1](https://openf1.org/) | Cross-check source for stint/tyre and session metadata |
| [Jolpica-F1](https://github.com/jolpica/jolpica-f1) (Ergast-compatible successor — Ergast itself shut down in early 2025) | Historical results/standings, via `fastf1.ergast` |
| [Open-Meteo](https://open-meteo.com/) | Historical Jeddah race-weekend weather (2021–2025) and the climate archive for the would-be 17–19 April 2026 dates |
| Pirelli press releases | Tyre compound nomination for the Saudi Arabia round |

## Methodology

1. **2026 pace baseline (regulation-shift normalization)** — historical Jeddah lap times are under old-regs cars and cannot anchor 2026 predictions directly. Every historical result is converted to a relative metric (gap-to-field-median) rather than an absolute lap time; absolute 2026 pace comes exclusively from 2026-season sessions.
2. **Track affinity / similarity index** — a per-circuit feature fingerprint (corner-speed distribution, % full throttle, braking severity, street vs. permanent) used to borrow signal for Jeddah from Baku, Melbourne, Miami, and Las Vegas.
3. **Tyre degradation modeling** — per-compound degradation curves fit from Jeddah historical stints, adjusted for track temperature.
4. **Safety car & DNF hazard modeling** — a Bayesian (PyMC) model over Jeddah's historical SC/VSC count, timing, and duration, plus a DNF hazard model per driver/team.
5. **Qualifying simulation** — run first, before the race: N simulated one-lap qualifying performances per driver (deterministic pace ± a qualifying-specific variance term, since one-lap pace has a different error distribution than race pace) produce a probability distribution over grid slots per driver, and a most-likely starting grid.
6. **Monte Carlo race simulation** — deterministic pace and tyre degradation combined with stochastic draws (safety car scenario, DNFs, pit-strategy reactions) across N simulated races, each one run from a grid sampled from the qualifying simulation's output rather than a fixed assumed grid, to produce most-likely podiums and finishing positions.

## Quickstart

*Ingestion and modeling are not implemented yet — this section will be filled
in as each phase lands.*

```bash
git clone https://github.com/<your-username>/ghost-grid.git
cd ghost-grid
# setup + run instructions go here once src/ghost_grid is implemented
```

## Results

*To be added once the Monte Carlo pipeline produces output — podium
probability chart and link to the full results notebook/dashboard.*

## Limitations & Honest Caveats

- No real telemetry ever existed for this weekend — every result here is a
  distribution, not a fact.
- The regulation-shift normalization (relative-gap transform) is a modeling
  choice, not ground truth, and is stated explicitly rather than hidden.
- Track-affinity weighting and tyre compound nomination (where not publicly
  confirmed) involve reasonable assumptions, documented inline in the
  relevant modules as they're built.

## License / Credits

© 2026 TheHappyBaloney
