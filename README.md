# Power Grid EMS Simulator

A Python simulator for power grid energy management. Models solar generation, battery storage, and load dispatch from scratch using NumPy, SciPy, and matplotlib.

## Why I Built This

I wanted to understand how energy management systems make dispatch decisions in real time. When you have intermittent generation like solar, limited storage, and loads that can't be interrupted, something has to decide where every watt goes at every moment. I built this simulator to work through that problem hands-on: model the generation sources, define the constraints, implement the dispatch logic, and see how different strategies perform against each other. The long-term goal is to be able to run scenarios that answer practical sizing and cost questions before any hardware gets deployed.

## What I Learned

**How simulation architectures work.** The system runs a discrete time-step loop at 1-minute resolution. Each step reads inputs, computes available generation, runs the dispatch algorithm, updates battery state, and logs metrics. Structuring it this way taught me how to separate the physical models (solar, battery, loads) from the control logic (the EMS), which is how real systems are architected.

**How inputs feed into EMS decisions.** Solar irradiance, ambient temperature, load profiles, battery state of charge, grid electricity prices. Each of these is a variable the dispatch algorithm has to weigh at every timestep. I learned that the hard part isn't any one input, it's how they interact. A cloudy afternoon changes the dispatch strategy completely because it cascades through storage, load curtailment, and cost.

**Real-world challenges that make this problem hard.** A simple "supply meets demand" model breaks down fast. Batteries lose 10-15% of energy per cycle to heat. You can't discharge below 20% SoC without degrading the cells. Solar output can swing 50% in minutes when clouds pass. Critical loads like hospital equipment can't be interrupted, ever. Each of these constraints narrows the set of valid dispatch decisions and forces real tradeoffs.

**How to measure tradeoffs in EMS systems.** There's no single "right answer" in dispatch. Minimizing cost might mean more load curtailment. Maximizing reliability might mean burning expensive generator fuel. I built the simulator to track self-consumption ratio, unserved critical load, battery utilization, grid dependency, and total system cost so you can see exactly what each strategy trades off against.

**Cost analysis and infrastructure sizing.** Running different scenarios lets you answer questions like: does doubling battery capacity actually halve grid dependency, or is there diminishing return? Is it cheaper to oversize solar and curtail the excess, or to right-size solar and rely on the grid during peaks? These are the kinds of decisions utilities and developers make when planning real infrastructure.

## System Architecture

Generation sources feed into a central power bus, loads draw from it, and the EMS controller decides how power is allocated at each timestep.

![System Architecture](docs/system-architecture.svg)

**Generation:**
- **Solar PV** - Output modeled as a function of irradiance, temperature, and panel area. Supports real data from NREL or synthetic profiles.
- **Conventional generator** - Dispatchable backup with fuel cost modeling. Activated when renewables and storage can't meet demand.

**Storage and grid interconnection:**
- **Battery storage** - Tracks state of charge over time with charge/discharge rate limits, round-trip efficiency, and depth-of-discharge constraints.
- **Grid interconnection** - Optional tie to an external grid for importing/exporting at time-varying rates. Can be disabled to simulate islanded operation.

**Loads:**
- **Critical loads** - Demand that must always be served (hospitals, water treatment, emergency systems).
- **Flexible loads** - Demand that can be shifted or curtailed (commercial HVAC, EV charging, industrial processes).

## Simulation Loop

![Simulation Loop](docs/simulation-loop.svg)

The dispatch algorithm allocates power using a priority-based strategy:

1. **Serve critical loads.** Renewable generation goes here first. If insufficient, the battery discharges. If both fall short, the conventional generator starts or we import from the grid.
2. **Charge the battery.** Excess generation beyond critical demand charges the battery, subject to rate limits and SoC bounds.
3. **Serve flexible loads.** Remaining generation goes to flexible loads. These get curtailed if supply is tight.
4. **Export surplus.** Any remaining generation is exported to the grid (grid-tied mode only).

Two dispatch modes are planned: the rule-based priority approach above, and an optimized approach using linear programming (`scipy.optimize.linprog`) to minimize total system cost over a rolling time horizon.

## Key Metrics

| Metric | What it measures |
|--------|-----------------|
| Self-consumption ratio | How much renewable generation is used on-site vs. exported |
| Unserved critical load | Energy demand that could not be met (primary reliability indicator) |
| Battery utilization | Cycles per day, average SoC, time at min/max bounds |
| Grid dependency | Net energy imported from the external grid |
| Total system cost | Fuel + grid import cost - grid export revenue |
| Load curtailment | Flexible load energy deferred or shed |

## Project Structure

```
powergrid-ems-simulator/
├── README.md
├── docs/
│   ├── system-architecture.svg
│   └── simulation-loop.svg
├── src/
│   ├── __init__.py
│   ├── solar.py          # Solar PV generation model
│   ├── battery.py        # Battery storage model (SoC tracking)
│   ├── loads.py           # Load profile generation
│   ├── grid.py            # Grid interconnection model
│   ├── generator.py       # Conventional generator model
│   ├── ems.py             # EMS dispatch controller
│   └── simulator.py       # Main simulation loop
├── data/
│   └── profiles/          # Solar irradiance and load profile CSVs
├── tests/
│   ├── test_solar.py
│   ├── test_battery.py
│   └── test_ems.py
├── examples/
│   └── run_24h_sim.py     # 24-hour simulation with output plots
├── requirements.txt
└── .gitignore
```

## Tech Stack

- **Python 3.10+**
- **NumPy** for array operations and time-series data
- **SciPy** for optimization (LP-based dispatch)
- **matplotlib** for visualization (power flow plots, SoC curves, load profiles)

No external power systems frameworks (PyPSA, OpenDSS, etc.). All models implemented from first principles.

## Roadmap

- [x] System architecture design
- [x] Simulation loop design
- [ ] Solar PV generation model
- [ ] Battery storage model
- [ ] Load profile generator
- [ ] Rule-based EMS dispatch
- [ ] Main simulation loop
- [ ] 24-hour simulation with plots
- [ ] Grid interconnection model (import/export pricing)
- [ ] Conventional generator model
- [ ] Optimized dispatch with scipy LP
- [ ] Real irradiance data from NREL NSRDB

## License

MIT
