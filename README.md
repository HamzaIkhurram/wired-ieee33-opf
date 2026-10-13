# WIRED Grid Data Challenge: Optimization Module

**IEEE 33-Bus Optimal Power Flow (Branch Flow Model)**

This is the optimization module for the WIRED Center Grid Data Challenge, *Data-Driven Grid Reconfiguration and Optimal Power Flow under Extreme Weather*. It gives you a clean, working OPF for the IEEE 33-bus distribution system so you can learn the model and experiment with it before the final prompt.

The official challenge runs on the Grid Data Hub at [datahub.wiredcenter.org](https://datahub.wiredcenter.org/). Launch this module from there.

---

## What the notebook does

`IEEE_33_OPF.ipynb` solves a time-series optimal power flow for the IEEE 33-bus system using only the benchmark (rated) line and load data. There is no PV, no storage and no external load data. It's a pure OPF.

- **Model:** branch flow (DistFlow) model. Voltage angles are removed from the equations, and the network is described by bus voltage magnitudes, branch power flows and branch currents.
- **Objective:** minimise total active power losses, the sum of I²R over all branches and time steps.
- **Constraints:**
  - substation (slack) bus voltage fixed at 1.0 p.u.
  - voltage drop along each branch
  - branch power and current: P² + Q² = V²·I²
  - active and reactive power balance at every bus
  - bus voltage limits of 0.90–1.05 p.u.
- **Solver:** IPOPT, through Pyomo.
- **Time series:** set up for 24 hourly time steps (`days = 1`). All equations are indexed by time, so you can change the horizon in the Configuration cell.

## The test system

| Parameter | Value |
|---|---|
| Rated voltage | 12.66 kV (line to line) |
| Total load | 3.715 MW, 2.30 MVAR |
| Buses / branches | 33 / 32 (radial) |
| Base power | 1 MVA |

Line impedances and bus loads are the standard benchmark values from Baran & Wu (1989). Buses are numbered 0–32 in the code (bus 0 is the substation), which is bus 1–33 in most papers.

## Expected results

With rated load at every hour, each hour gives the same answer:

| Quantity | Value |
|---|---|
| Substation supply | 3917.68 kW |
| Total losses | 202.68 kW |
| Minimum voltage | 0.9131 p.u. at bus 17 (bus 18 in 1-indexed numbering) |

These match the published IEEE 33-bus base case, so you can use them to check your setup.

---

## Running on the Grid Data Hub

1. Open the module and read the instructions.
2. Scroll to **Launch Options** and select **NSF National Research Platform**.
3. In JupyterHub, reserve 1 core and 16 GB RAM, no GPU, with the **Minimal NDP Starter JupyterLab** image, then click **Start Server**. Allow about five minutes.
4. Open a terminal and run `pip install -r requirements.txt`.
5. Open `IEEE_33_OPF.ipynb` and run all cells from top to bottom. The solver setup cell installs IPOPT if it isn't already there.
6. When you're done, stop your server: **File → Hub Control Panel → Stop Server**.

## Running locally

```bash
git clone https://github.com/HamzaIkhurram/wired-ieee33-opf.git
cd wired-ieee33-opf
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab IEEE_33_OPF.ipynb
```

IPOPT is not a Python package. The notebook installs it for you, or you can install it yourself with `conda install -c conda-forge ipopt`, with `brew install ipopt` on macOS, or with `pip install idaes-pse && idaes get-extensions`.

## Outputs

Results are written to `Results/Obj(MinLosses)/Results_<Vmag>pu/`:

- `Voltage_Profiles_Data.csv`: bus voltages (p.u.) for each hour
- `Pg_Data.pkl`: substation active power (kW) for each hour
- `Total_losses_Data.pkl`: total losses (kW) for each hour
- `Branches_Data.pkl`: P, Q and current on every branch for each hour
- `Voltage_Profiles.pdf`: voltage profiles over the day

---

## Things to try

- **Change the load.** Replace the nominal `load_p_nominal` / `load_q_nominal` values, or multiply them by a daily load profile so each hour is different. The line impedances stay the same, because they are the benchmark.
- **Change the horizon.** Set `days` in the Configuration cell and rerun everything below it.
- **Change the substation voltage.** Try `Vmag = 1.02` or `1.05` and see how losses and the minimum voltage move.
- **Change the objective.** For example, minimise voltage deviation from 1.0 p.u. instead of losses.
- **Add resources.** PV at the weak end-of-feeder buses, or a battery with state-of-charge and charge/discharge limits. Each one adds a variable to the power balance at its bus.

Note that with fixed loads and a fixed substation voltage, a radial feeder has a single power flow solution. The objective only starts to matter once you add something the optimiser can control.

## Repo layout

```
.
├── IEEE_33_OPF.ipynb   # Time-series OPF for the IEEE 33-bus system
├── requirements.txt    # Python dependencies (IPOPT installed separately)
├── .gitignore
└── README.md
```

## Questions?

- **Muhammad Hamza Imtiaz**: hamza.imtiaz@ucalgary.ca
- **Ilkay Altintas**: ialtintas@ucsd.edu
- **Mostafa Farrokhabadi**: mostafa.farrokhabadi@ucalgary.ca

## Credits

OPF model and notebook by Ala'a. Packaged as a WIRED module by Muhammad Hamza.

## References

1. M. E. Baran and F. F. Wu, "Network reconfiguration in distribution systems for loss reduction and load balancing," *IEEE Transactions on Power Delivery*, vol. 4, no. 2, pp. 1401–1407, 1989.
2. M. Farivar and S. H. Low, "Branch flow model: relaxations and convexification, Part I," *IEEE Transactions on Power Systems*, vol. 28, no. 3, 2013.
