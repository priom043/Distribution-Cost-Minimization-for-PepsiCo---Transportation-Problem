# 🚚 PepsiCo Bangladesh — Distribution Network Optimization
### A Linear Programming Approach to the Transportation Problem

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![Pyomo](https://img.shields.io/badge/Pyomo-6.9.5-orange?style=flat)
![Gurobi](https://img.shields.io/badge/Solver-Gurobi%20%7C%20CPLEX-red?style=flat)
![Course](https://img.shields.io/badge/Course-IPE%20307-blue?style=flat)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat)

---

## 📌 Table of Contents

1. [Project Overview](#project-overview)
2. [Repository Structure](#repository-structure)
3. [Business Context](#business-context)
4. [Network Structure](#network-structure)
5. [Mathematical Formulation](#mathematical-formulation)
6. [Why This is an Unbalanced Problem](#why-this-is-an-unbalanced-problem)
7. [Cost Structure](#cost-structure)
8. [Solution Methodology](#solution-methodology)
9. [Results](#results)
10. [Managerial Insights](#managerial-insights)
11. [How to Run](#how-to-run)
12. [Dependencies](#dependencies)
13. [Academic Context](#academic-context)
14. [References](#references)

---

## Project Overview

This project models and solves a real-world **supply chain distribution problem** for PepsiCo Bangladesh using **Linear Programming (LP)** — specifically the classical **Transportation Problem** formulation from Operations Research.

The objective is to determine the optimal number of cartons to ship from each PepsiCo production facility to each distribution center (DC) such that **total transportation cost is minimized**, while respecting the physical supply and demand constraints of the network.

The problem is modeled and solved entirely in **Python** using the **Pyomo** algebraic modeling language (v6.9.5) with **Gurobi** as the backend LP solver — replacing the manual tabular methods (Northwest Corner Rule, Vogel's Approximation, MODI method) traditionally used to solve transportation problems by hand.

> **Key Result:** The optimal shipping plan achieves a minimum total cost of **Tk 4,394,300,000**, with all available supply fully utilized across 7 active shipping routes.

---

## Repository Structure

```
pepsico-transportation-problem/
│
├── README.md
├── PepsiCo_Data.xlsx                               ← Data description (distance matrix, toll costs, cost matrix)
└── PepsiCo_Transportation_Problem (IPE 307).ipynb  ← Full solution notebook
```

---

## Business Context

PepsiCo Bangladesh operates multiple production facilities across the country and needs to fulfill demand at regional distribution centers efficiently. Transportation costs are driven by two factors:

- **Distance** between each factory and distribution center (km)
- **Toll costs** along specific road segments (highways, bridges, flyovers)

Each factory has a fixed production capacity (supply), and each DC has a known regional demand. The challenge is to allocate shipments across all factory–DC routes to satisfy as much demand as possible at minimum cost — a textbook **unbalanced transportation problem**.

---

## Network Structure

The distribution network consists of **3 production factories** (supply nodes) and **5 distribution centers** (demand nodes), forming a bipartite network with **15 possible shipping routes**.

### Supply Nodes (Factories)

| Factory | Location | Supply Capacity (Cartons) |
|---|---|---|
| Konabari | Gazipur, Dhaka | 45,000 |
| Bagher Bazar | Dhaka | 90,000 |
| Chittagong | Chittagong | 80,000 |
| | **Total Supply** | **215,000** |

### Demand Nodes (Distribution Centers)

| Distribution Center | Demand (Cartons) |
|---|---|
| Sylhet DC | 52,000 |
| Bogura DC | 35,000 |
| Chittagong DC | 50,000 |
| Khulna DC | 50,000 |
| Gazipur DC | 60,000 |
| **Total Demand** | **247,000** |

### Network Diagram

```
                          ┌─────────────┐
               ┌─────────►│  Sylhet DC  │  Demand: 52,000
               │          └─────────────┘
               │
  [45,000] ────┤          ┌─────────────┐
  Konabari     ├─────────►│  Bogura DC  │  Demand: 35,000
               │          └─────────────┘
               │
               │          ┌──────────────────┐
               └─────────►│  Chittagong DC   │  Demand: 50,000
                          └──────────────────┘
  [90,000] ────┐
 Bagher Bazar  │          ┌─────────────┐
               ├─────────►│  Khulna DC  │  Demand: 50,000
               │          └─────────────┘
               │
  [80,000] ────┤          ┌──────────────┐
  Chittagong   └─────────►│  Gazipur DC  │  Demand: 60,000
                          └──────────────┘
```

---

## Mathematical Formulation

### Decision Variables

Let $X_{ij}$ be the number of cartons shipped from factory $i$ to distribution center $j$, where:

- $i \in \{K,\ B,\ C\}$ — Konabari (K), Bagher Bazar (B), Chittagong factory (C)
- $j \in \{1,\ 2,\ 3,\ 4,\ 5\}$ — Sylhet (1), Bogura (2), Chittagong DC (3), Khulna (4), Gazipur (5)

### Objective Function

Minimize total transportation cost:

$$\min Z = \sum_{i} \sum_{j} c_{ij} \cdot X_{ij}$$

Expanded:

$$\min Z = 35480X_{K1} + 23380X_{K2} + 43280X_{K3} + 38900X_{K4} + 2940X_{K5}$$
$$+ 35900X_{B1} + 27160X_{B2} + 45940X_{B3} + 41700X_{B4} + 5600X_{B5}$$
$$+ 52920X_{C1} + 62040X_{C2} + 2800X_{C3} + 67300X_{C4} + 39960X_{C5}$$

### Supply Constraints

All production must be shipped out (equality — see [why below](#why-this-is-an-unbalanced-problem)):

$$X_{K1} + X_{K2} + X_{K3} + X_{K4} + X_{K5} = 45{,}000 \quad \text{(Konabari)}$$

$$X_{B1} + X_{B2} + X_{B3} + X_{B4} + X_{B5} = 90{,}000 \quad \text{(Bagher Bazar)}$$

$$X_{C1} + X_{C2} + X_{C3} + X_{C4} + X_{C5} = 80{,}000 \quad \text{(Chittagong)}$$

### Demand Constraints

DCs receive *at most* their demanded quantity — total supply is insufficient to meet all demand:

$$X_{K1} + X_{B1} + X_{C1} \leq 52{,}000 \quad \text{(Sylhet)}$$

$$X_{K2} + X_{B2} + X_{C2} \leq 35{,}000 \quad \text{(Bogura)}$$

$$X_{K3} + X_{B3} + X_{C3} \leq 50{,}000 \quad \text{(Chittagong DC)}$$

$$X_{K4} + X_{B4} + X_{C4} \leq 50{,}000 \quad \text{(Khulna)}$$

$$X_{K5} + X_{B5} + X_{C5} \leq 60{,}000 \quad \text{(Gazipur)}$$

### Non-negativity

$$X_{ij} \geq 0 \quad \forall\ i,\ j$$

---

## Why This is an Unbalanced Problem

A transportation problem is **balanced** when total supply equals total demand. Here:

| | Cartons |
|---|---|
| Total Supply | 215,000 |
| Total Demand | 247,000 |
| **Shortfall** | **−32,000** |

Since **supply < demand**, this is an **unbalanced transportation problem with excess demand**. Two direct implications for the Pyomo formulation:

**1. Supply constraints use `==`** — every carton produced at every factory must be shipped out. No factory retains inventory.

**2. Demand constraints use `<=`** — since there is insufficient supply to meet all demand, DCs receive *at most* their demanded quantity. The solver determines which DC absorbs the shortfall.

In the classical manual approach, a **dummy supply node** of 32,000 cartons (with zero cost) would be added to artificially balance the table before applying Vogel's or MODI. Pyomo handles this natively through inequality constraints — no dummy node is needed.

---

## Cost Structure

Transportation cost per carton (Bangladeshi Taka, Tk) is computed as:

$$\text{Cost}_{ij} = (\text{Distance}_{ij} \times 140\ \text{Tk/km}) + \text{Toll Cost}_{ij}$$

The rate of **140 Tk/km** represents the minimum operating cost per kilometer for a delivery truck in Bangladesh. Toll costs vary by route based on which highways, bridges, and flyovers are traversed.

### Distance Matrix (km)

| Factory \ DC | Sylhet | Bogura | Chittagong | Khulna | Gazipur |
|---|---|---|---|---|---|
| Konabari | 232 | 157 | 292 | 250 | 21 |
| Bagher Bazar | 235 | 184 | 311 | 270 | 40 |
| Chittagong | 378 | 441 | 20 | 450 | 274 |

### Notable Toll Routes

| Route | Key Toll Roads | Total Toll (Tk) |
|---|---|---|
| Any Factory → Sylhet | Aushkandi–Sherpur Rd, Dhaka–Sylhet Hwy, Jamtola–Charshindur Rd | 3,000 |
| Any Factory → Bogura | Jamuna Bridge | 1,400 |
| Any Factory → Chittagong | Bostail–Madanpur Hwy, Dhaka–Chittagong Hwy | 2,400 |
| Any Factory → Khulna | Padma Bridge Rd, Dhaka–Mawa Hwy, Hanif Flyover | 3,900 |
| Chittagong → Khulna | Khulna City Bypass, Padma Bridge Rd, Dhaka–Mawa Hwy | 4,300 |
| Any Factory → Gazipur | No tolls | 0 |

### Final Cost Matrix (Tk per carton)

| Factory \ DC | Sylhet | Bogura | Chittagong | Khulna | Gazipur |
|---|---|---|---|---|---|
| Konabari | 35,480 | 23,380 | 43,280 | 38,900 | **2,940** |
| Bagher Bazar | 35,900 | 27,160 | 45,940 | 41,700 | 5,600 |
| Chittagong | 52,920 | 62,040 | **2,800** | 67,300 | 39,960 |

> Full derivation (distance × rate + toll breakdown) is in [`PepsiCo_Data.xlsx`](./PepsiCo_Data.xlsx).

---

## Solution Methodology

### Why Pyomo over Manual Methods?

| Method | Use Case | Limitation |
|---|---|---|
| Northwest Corner Rule | Initial BFS (hand calculation) | Ignores costs entirely |
| Vogel's Approximation | Better quality initial BFS | Still requires manual iteration |
| MODI / Stepping-Stone | Checking and improving solution by hand | Tedious and error-prone at scale |
| **Pyomo + Gurobi** | **Direct algebraic LP solution** | **None for this scale** |

Pyomo expresses the LP as a structured algebraic model and passes it to **Gurobi**, which solves it via the **Simplex Method**. No initial BFS method is required — the solver handles everything internally.

### Pyomo Implementation

```python
import pyomo.environ as pyo

sources      = ['Konabari', 'BagherBazar', 'Chittagong']
destinations = ['Sylhet', 'Bogura', 'Chittagong', 'Khulna', 'Gazipur']

supply = {'Konabari': 45000, 'BagherBazar': 90000, 'Chittagong': 80000}

demand = {'Sylhet': 52000, 'Bogura': 35000, 'Chittagong': 50000,
          'Khulna': 50000, 'Gazipur': 60000}

cost = {
    ('Konabari',    'Sylhet'):     35480,
    ('Konabari',    'Bogura'):     23380,
    ('Konabari',    'Chittagong'): 43280,
    ('Konabari',    'Khulna'):     38900,
    ('Konabari',    'Gazipur'):     2940,
    ('BagherBazar', 'Sylhet'):     35900,
    ('BagherBazar', 'Bogura'):     27160,
    ('BagherBazar', 'Chittagong'): 45940,
    ('BagherBazar', 'Khulna'):     41700,
    ('BagherBazar', 'Gazipur'):     5600,
    ('Chittagong',  'Sylhet'):     52920,
    ('Chittagong',  'Bogura'):     62040,
    ('Chittagong',  'Chittagong'):  2800,
    ('Chittagong',  'Khulna'):     67300,
    ('Chittagong',  'Gazipur'):    39960,
}

# Model
model   = pyo.ConcreteModel()
model.x = pyo.Var(sources, destinations, domain=pyo.NonNegativeReals)

# Objective
model.obj = pyo.Objective(
    expr=sum(cost[i, j] * model.x[i, j] for i in sources for j in destinations),
    sense=pyo.minimize
)

# Supply constraints (==): unbalanced TP — all supply must ship out
def supply_rule(m, i):
    return sum(m.x[i, j] for j in destinations) == supply[i]
model.supply_constraint = pyo.Constraint(sources, rule=supply_rule)

# Demand constraints (<=): supply < demand, DCs may be partially served
def demand_rule(m, j):
    return sum(m.x[i, j] for i in sources) <= demand[j]
model.demand_constraint = pyo.Constraint(destinations, rule=demand_rule)

# Solve
solver = pyo.SolverFactory('gurobi')
result = solver.solve(model, tee=False)

print(f"Status      : {result.solver.termination_condition}")
print(f"Minimum Cost: Tk {pyo.value(model.obj):,.0f}")
```

---

## Results

### Optimal Shipping Plan

| Route | Cartons Shipped | Unit Cost (Tk) | Route Cost (Tk) |
|---|---|---|---|
| Konabari → Bogura | 35,000 | 23,380 | 818,300,000 |
| Konabari → Khulna | 10,000 | 38,900 | 389,000,000 |
| Bagher Bazar → Sylhet | 22,000 | 35,900 | 789,800,000 |
| Bagher Bazar → Khulna | 8,000 | 41,700 | 333,600,000 |
| Bagher Bazar → Gazipur | 60,000 | 5,600 | 336,000,000 |
| Chittagong → Sylhet | 30,000 | 52,920 | 1,587,600,000 |
| Chittagong → Chittagong DC | 50,000 | 2,800 | 140,000,000 |
| **TOTAL** | **215,000** | | **Tk 4,394,300,000** |

### Supply Utilization

| Factory | Shipped | Capacity | Utilization |
|---|---|---|---|
| Konabari | 45,000 | 45,000 | 100% |
| Bagher Bazar | 90,000 | 90,000 | 100% |
| Chittagong | 80,000 | 80,000 | 100% |

### Demand Fulfillment

| DC | Received | Demanded | Fulfilled |
|---|---|---|---|
| Sylhet | 52,000 | 52,000 | ✅ 100% |
| Bogura | 35,000 | 35,000 | ✅ 100% |
| Chittagong DC | 50,000 | 50,000 | ✅ 100% |
| Khulna | 18,000 | 50,000 | ⚠️ 36% |
| Gazipur | 60,000 | 60,000 | ✅ 100% |

> **Note:** Khulna absorbs the entire 32,000-carton shortfall because all three factories face their highest unit costs on the Khulna route (Tk 38,900 – Tk 67,300). The solver exhausts supply on cheaper routes first, leaving Khulna underserved.

---

## Managerial Insights

### 1. Proximity Principle Dominates the Solution
The two cheapest routes in the network — **Konabari → Gazipur (Tk 2,940)** and **Chittagong → Chittagong DC (Tk 2,800)** — are both utilized at full capacity. The solver naturally assigns each factory primarily to its geographically closest DC, consistent with what Vogel's Approximation would suggest.

### 2. Khulna is the Sacrificed Market
Khulna receives only **18,000 of its 50,000 demanded cartons (36%)**. Every factory incurs its highest or near-highest unit cost to serve Khulna, so the solver routes scarce supply elsewhere first. This represents a significant lost revenue and customer service risk.

**Recommendation:** PepsiCo should evaluate whether a satellite storage or production facility near Khulna (e.g., Jessore or Faridpur) would be economically justified given the consistent shortfall.

### 3. Chittagong Factory Serves Two Roles
The Chittagong factory ships its entire output to just two DCs — **50,000 cartons to Chittagong DC** at the network's lowest cost (Tk 2,800) and **30,000 cartons to Sylhet** at a much higher cost (Tk 52,920). The Sylhet allocation is expensive but unavoidable: Konabari and Bagher Bazar exhaust their supply before Sylhet's demand is fully met.

### 4. Bogura Depends Entirely on Konabari
The full 35,000-carton demand at Bogura is met solely by Konabari (Tk 23,380/carton). The next cheapest alternative, Bagher Bazar, costs Tk 27,160 — a 16% premium. Any supply disruption at Konabari would significantly raise the cost of serving Bogura.

### 5. Quantified Value of Capacity Expansion
Expanding **any factory's capacity** to close the 32,000-carton gap would redirect additional supply to Khulna via Bagher Bazar (Tk 41,700/carton — the cheapest remaining Khulna route). A full 32,000-carton expansion would reduce total cost by approximately **Tk 749,800,000**.

---

## How to Run

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/pepsico-transportation-problem.git
cd pepsico-transportation-problem
```

### 2. Install Pyomo
```bash
pip install pyomo==6.9.5
```

### 3. Install a solver
```bash
pip install gurobipy   # Gurobi (recommended)
# or
pip install cplex      # IBM CPLEX
```

> Gurobi offers a free academic license at [gurobi.com/academia](https://www.gurobi.com/academia/academic-program-and-licenses/). CPLEX is available free for academics via [IBM Academic Initiative](https://www.ibm.com/academic/topic/data-science).

### 4. Open the notebook
```bash
jupyter notebook "PepsiCo_Transportation_Problem (IPE 307).ipynb"
```

---

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| `pyomo` | 6.9.5 | LP modeling — variables, objective, constraints |
| `gurobipy` | latest | LP solver backend (or use `cplex`) |
| `jupyter` | latest | Notebook environment |

```bash
pip install pyomo==6.9.5 jupyter
pip install gurobipy   # or: pip install cplex
```

---

## Academic Context

This project was completed as part of **IPE 307 — Operations Research**, exploring the application of Linear Programming to classical supply chain optimization problems.
This was originally a group project jointly implemented by Tawfique Ihsan (1908043), Mohiuddin Adnan (1908042) and Sadat Iqbal (1908043). Subsequently, the project was redone by using Pyomo and Gurobi solver by Sadat Iqbal.

### Topics Demonstrated
- Transportation Problem formulation (balanced vs. unbalanced)
- Linear Programming (objective function, supply/demand constraints, non-negativity)
- Distinction between `==` supply and `<=` demand constraints in unbalanced TPs
- Real-world cost modeling (distance × rate + toll costs)
- Algebraic modeling with Pyomo vs. manual tabular methods (NW Corner, Vogel's, MODI)

### Connection to Broader OR Theory
The transportation problem is a special case of the **minimum-cost flow problem** on a bipartite network. It can also be viewed as a degenerate LP where the constraint matrix is **totally unimodular** — guaranteeing integer optimal solutions whenever supply and demand values are integers. This is why Gurobi returns whole-number carton allocations without explicitly enforcing integrality constraints.

---

## References

- Hillier, F. S., & Lieberman, G. J. (2015). *Introduction to Operations Research* (10th ed.). McGraw-Hill.
- Taha, H. A. (2017). *Operations Research: An Introduction* (10th ed.). Pearson.
- Pyomo Documentation: https://pyomo.readthedocs.io
- Gurobi Optimizer: https://www.gurobi.com

---

<div align="center">

*IPE 307 — Operations Research &nbsp;|&nbsp; Built with Python 🐍 & Pyomo*

</div>
