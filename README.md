# Colombo Ride-Hailing AI

**Event-Aware Proactive Driver Positioning & Rule-Based Dynamic Pricing**  
7COSC013W.1 Foundations of AI — University of Westminster

---

## Overview

A simulation framework that predicts demand surges in Colombo before they happen — driven by live weather, cricket fixtures, and Poya public holidays — and proactively repositions drivers across 10 city zones to cut wait times and smooth surge pricing.

Two positioning algorithms are compared against a static baseline:

| Algorithm | Approach |
|-----------|----------|
| **Greedy** | Iteratively assigns idle drivers to the highest-deficit zone reachable within 10 min |
| **Simulated Annealing (SA)** | Stochastic search over the same objective; avoids local optima at the cost of ~20× more compute |
| **Baseline** | No repositioning; drivers stay where they are |

Dynamic pricing applies graduated multipliers (1.15× → 1.50×, hard-capped at 2.00×) only when a coverage gap persists after positioning — so the engine tries to solve supply problems with drivers first, price second.

---

## Results

Evaluated across **30 scenarios** (3 event types × 10 random seeds) using a synthetic fleet of 200 drivers.

### Success Criteria (Greedy variant)

| Metric | Target | Achieved | |
|--------|--------|----------|-|
| Surge prevention rate | ≥ 60% | **95.0%** | ✅ |
| Avg surge multiplier | ≤ 1.35× | **1.25×** | ✅ |
| Constraint satisfaction | 100% | **100%** | ✅ |
| Computation time | < 5 s | **0.17 s** | ✅ |
| Wait time reduction | ≥ 25% | 16.9% | ❌ |
| Gini coefficient | < 0.20 | 0.228 | ❌ |

4 of 6 criteria met. Wait-time reduction falls short on Cricket scenarios (drivers are already optimally distributed for uniform-demand games); Gini is elevated in Monsoon because the city concentrates demand in Fort/Pettah.

### Per-Scenario Breakdown

| Scenario | Variant | Surge Prev. | Avg Mult. | Wait Reduction | Compute |
|----------|---------|-------------|-----------|----------------|---------|
| Monsoon | Baseline | 0% | 1.00× | 0% | — |
| Monsoon | Greedy | **95%** | 1.15× | **34.6%** | 0.036 s |
| Monsoon | SA | 88% | 1.15× | −4.8% | 0.700 s |
| Cricket | Baseline | 0% | 1.00× | 0% | — |
| Cricket | Greedy | **100%** | 1.27× | 0% | 0.001 s |
| Cricket | SA | **100%** | 1.27× | 0% | 0.003 s |
| Poya | Baseline | 0% | 1.00× | 0% | — |
| Poya | Greedy | **90%** | 1.33× | **16%** | 0.103 s |
| Poya | SA | 90% | 1.33× | 16% | 1.599 s |

Greedy matches or beats SA on every metric while running 20–700× faster — a key finding of this coursework.

---

## Architecture

```
Event Scraper ──┐
Weather API ────┤──► Demand Engine (rule-based) ──► Positioning Engine ──► Pricing Engine
Poya Calendar ──┘         (fired rules)              Greedy / SA              Graduated multipliers
Cricket Fixtures ─────────────────────────────────────────────────────────────────────────────►
                                                                                      Metrics & Evaluation
```

**10 zones** modelled across Greater Colombo (Fort/Pettah, Kollupitiya, Borella, Maitland Crescent, Bambalapitiya, Nugegoda, Rajagiriya, Kelaniya, Gangaramaya, Maradana/Premadasa).

---

## Project Structure

```
src/
├── config.py                  # Zones, hyperparameters, API endpoints
├── demand_rule_base.py        # Rule-based demand scoring
├── demand_model.py            # ML demand model (comparison)
├── positioning_engine.py      # Greedy + SA algorithms
├── pricing_engine.py          # Dynamic pricing multipliers
├── simulation.py              # Scenario runner
├── evaluation.py              # 30-scenario evaluation suite
├── event_scraper.py           # LankaEvents / Eventbrite scraper
├── weather_api.py             # Open-Meteo integration
├── road_network.py            # OSMnx graph + travel-time queries
├── travel_time_matrix.py      # Pre-computed zone-to-zone matrix
├── zone_builder.py            # GeoJSON zone definitions
└── ...
notebooks/
└── main_notebook.ipynb        # End-to-end walkthrough with plots
tests/                         # pytest suite (positioning, pricing, datasets)
```

---

## Quick Start

```bash
pip install -r requirements.txt
pip install -e .

# Run the full 30-scenario evaluation
python -m src.evaluation

# Or open the notebook
jupyter lab notebooks/main_notebook.ipynb
```

---

## Tech Stack

`osmnx` · `networkx` · `geopandas` · `folium` · `pandas` · `numpy` · `scipy` · `open-meteo` · `beautifulsoup4` · `matplotlib` · `seaborn` · `plotly` · `pytest`
