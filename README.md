# Microgrid EMS Simulator

A Python-based energy management system (EMS) simulator for modeling microgrid power flow, battery storage optimization, and load dispatch strategies. Built from scratch using NumPy, SciPy, and matplotlib — no black-box power systems libraries.

## Motivation

Over 750 million people worldwide still lack reliable access to electricity. Microgrids — small-scale power networks combining solar generation, battery storage, and intelligent load management — are one of the most promising paths to closing that gap. This simulator models the core engineering challenges of microgrid design: balancing intermittent generation with variable demand, optimizing battery utilization, and ensuring critical loads are always served.

The goal is to build an open-source tool that can simulate realistic microgrid scenarios (a rural clinic, a small community, a commercial building) and evaluate different dispatch strategies before deploying hardware.

## System Architecture

The simulator models five interconnected components connected through a central power bus:

![System Architecture](docs/system-architecture.svg)

**Generation sources** produce power that feeds into the bus:
- **Solar PV array** — Models power output as a function of solar irradiance, temperature, and panel area using a simplified single-diode model. Irradiance profiles can be loaded from real-world datasets (e.g., NREL NSRDB) or generated synthetically.
- **Diesel generator** — Backup generation with fuel consumption and cost modeling. Activated by the EMS when solar + battery cannot meet critical load demand.

**Storage and grid connections** exchange power bidirectionally with the bus:
- **Battery storage** — Tracks state of charge (SoC) over time with configurable capacity, charge/discharge rate limits, round-trip efficiency losses, and depth-of-discharge constraints. Models degradation as a function of cycle count.
- **Main grid connection** — Optional grid-tied mode allowing import/export of power at time-varying electricity rates (for economic optimization scenarios). Can be disabled to simulate off-grid operation.

**Loads** consume power from the bus:
- **Critical loads** — Non-negotiable demand that must always be served (medical equipment, refrigeration, water pumps). Unserved critical load is tracked as a key reliability metric.
- **Flexible loads** — Deferrable demand that the EMS can shift or curtail (EV charging, HVAC, non-essential lighting). Enables demand response strategies.

## Simulation Loop

The simulator runs a discrete time-step loop over a configurable duration (default: 24 hours at 1-minute resolution). Each timestep executes the following sequence:

![Simulation Loop](docs/simulation-loop.svg)

The **EMS dispatch algorithm** is the core decision engine. At each timestep, it determines how to allocate available generation across loads, storage, and grid export using a priority-based strategy:

1. **Serve critical loads first** — All available solar generation is directed to critical loads. If solar is insufficient, the battery discharges to cover the deficit. If both are insufficient, the diesel generator starts (or grid import is used if grid-tied).
2. **Charge battery if SoC is low** — Excess solar beyond critical load demand charges the battery, subject to charge rate limits and SoC ceiling.
3. **Serve flexible loads** — Remaining generation (after critical loads and battery charging) serves flexible loads. Flexible loads can be curtailed if generation is scarce.
4. **Export surplus to grid** — Any generation remaining after all loads and storage are satisfied is exported to the grid (grid-tied mode only).

Two dispatch modes are planned:
- **Rule-based dispatch** — Fixed priority ordering as described above. Simple, interpretable, and robust.
- **Optimized dispatch** — Uses linear programming (via `scipy.optimize.linprog`) to minimize total cost (fuel + grid import - grid export revenue) or maximize self-consumption over a rolling time horizon.

## Key Metrics

The simulator tracks and visualizes:

| Metric | Description |
|--------|-------------|
| **Self-consumption ratio** | Fraction of solar generation consumed on-site vs. exported |
| **Unserved critical load** | Total energy demand that could not be met (reliability indicator) |
| **Battery utilization** | Cycles per day, average SoC, time at min/max bounds |
| **Grid dependency** | Net energy imported from the grid over the simulation period |
| **Total cost** | Fuel cost + grid import cost - grid export revenue |
| **Load curtailment** | Total flexible load energy deferred or shed |

## Project Structure

```
microgrid-ems-simulator/
├── README.md
├── docs/
│   ├── system-architecture.svg
│   └── simulation-loop.svg
├── src/
│   ├── __init__.py
│   ├── solar.py          # Solar PV generation model
│   ├── battery.py        # Battery storage model (SoC tracking)
│   ├── loads.py           # Load profile generation
│   ├── grid.py            # Grid connection model
│   ├── diesel.py          # Diesel generator model
│   ├── ems.py             # EMS dispatch controller
│   └── simulator.py       # Main simulation loop
├── data/
│   └── profiles/          # Solar irradiance & load profile CSVs
├── tests/
│   ├── test_solar.py
│   ├── test_battery.py
│   └── test_ems.py
├── examples/
│   └── run_24h_sim.py     # Example: 24-hour simulation with plots
├── requirements.txt
└── .gitignore
```

## Tech Stack

- **Python 3.10+** — Core language
- **NumPy** — Array operations, time-series data
- **SciPy** — Optimization (linear programming for dispatch)
- **matplotlib** — Visualization (power flow plots, SoC curves, stacked area charts)

No external power systems frameworks (PyPSA, OpenDSS, etc.) — all models are implemented from first principles to demonstrate understanding of the underlying physics and control logic.

## Roadmap

- [x] System architecture design
- [x] Simulation loop design
- [ ] Solar PV generation model (irradiance → power)
- [ ] Battery storage model (SoC tracking, efficiency, constraints)
- [ ] Load profile generator (critical + flexible)
- [ ] Rule-based EMS dispatch controller
- [ ] Main simulation loop integration
- [ ] 24-hour simulation with matplotlib visualization
- [ ] Grid connection model (import/export pricing)
- [ ] Diesel generator model (fuel cost)
- [ ] Optimized dispatch (scipy LP)
- [ ] Real-world irradiance data integration (NREL NSRDB)

## License

MIT
