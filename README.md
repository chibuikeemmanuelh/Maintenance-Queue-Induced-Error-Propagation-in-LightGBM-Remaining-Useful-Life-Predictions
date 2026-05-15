
# Aircraft Engine RUL Prognostics & Maintenance Queue Simulation

[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://python.org)
[![LightGBM](https://img.shields.io/badge/LightGBM-3.3.5-green.svg)](https://lightgbm.readthedocs.io/)
[![SimPy](https://img.shields.io/badge/SimPy-4.0.2-orange.svg)](https://simpy.readthedocs.io/)

## Overview

This project develops a predictive maintenance framework for aircraft turbofan engines using NASA's C-MAPSS dataset. It combines **machine learning (LightGBM)** for Remaining Useful Life (RUL) prediction with **discrete-event simulation (SimPy)** to model the impact of maintenance queue delays on engine fleet health.

Unlike traditional approaches that assume instant maintenance access, this framework quantifies how real-world constraints, such as limited repair shops and queuing delays degrade prediction accuracy and engine condition.

**Key Insight:** Even accurate RUL predictions become operationally obsolete if engines continue to degrade while waiting for repair.

---

## Features

- **Data preprocessing** for run-to-failure turbofan data (FD001)
- **Manual feature engineering** (177 physics-informed features) to avoid "black-box" model
- **LightGBM** with Optuna hyperparameter tuning
- **SHAP** model interpretability (global and local explanations)
- **Discrete-event simulation** using SimPy with:
  - Priority-based repair queues (lowest RUL first)
  - Continuous degradation during waiting periods
  - Stochastic arrival processes (exponential inter-arrival times)
  - RUL-dependent repair durations (triangular distributions)
- **Systematic capacity planning** (1–15 repair servers, 10 replications each)

---

## Dataset

**Source:** NASA C-MAPSS FD001 ([Download](https://data.nasa.gov/dataset/c-mapss-aircraft-engine-simulator-data))

After Target Creation:

| Component | Rows | Columns |
|-----------|------|---------|
| Training and Validation set | 20,631 | 27 |
| Hold-out set | 100 | 27 | 

**Variables:** 'ID', 'Cycle', 'OpSet1', 'OpSet2', 'OpSet3', 'T2', 'T24', 'T30', 'T50', 'P2', 'P15', 'P30', 'Nf', 'Nc', 'epr', 'Ps30', 'phi', 'NRf', 'NRc', 'BPR', 'farB', 'htBleed', 'Nf_dmd', 'PCNFR_dmd', 'W31', 'W32', 'RUL'.

---

## Approach

### 1. Data Preprocessing
- RUL calculation: `RUL = EOL_ID - Cycle`
- Train/validation split: First 80% of cycles per engine (respects temporal ordering)
- Outlier assessment using IQR method (retained due to minimal RMSE impact)

### 2. Feature Engineering
- 177 manually engineered features based on:
  - Exponential degradation models: `h(t) = 1 - exp(at^b)`
  - Thermodynamic interactions (temperature-pressure-speed)
  - Rolling statistics (5-cycle moving averages)
  - Physics-informed combinations (e.g., `log1p(NRf) × sinh(W31) × W32 / (|phi| × Cycle)`)
- Feature reduction via Permutation Importance (removed 105 non-informative features)

### 3. Model Training
- **Algorithm:** LightGBM 
- **Optimization:** Optuna (500 trials)
- **Evaluation:** RMSE (primary), MAE (secondary)

### 4. Simulation Framework (SimPy)
- **Entities:** 100 engines from hold-out set
- **Arrival process:** Exponential (mean 11 days between arrivals)
- **Repair durations:**
  - Low-risk: Triangular(45, 55, 65) days
  - Medium-risk: Triangular(55, 60, 65) days
  - High-risk: Fixed 80 days
- **Degradation in queue:** `RUL_degraded = RUL_true - (wait_days / 7.5) × 24`
- **Experiment:** Vary repair servers from 1 to 15, 10 replications each

---

## Results Summary

### Model Performance (Hold-out Set)
| Metric | Value |
|--------|-------|
| RMSE | 28.17 cycles |
| MAE | 22.95 cycles |
| R² (variance explained) | 83.27% |

### Simulation Findings
**Operational actionable threshold:** 8 repair shops 

### Repository Structure
```
README.md
requirements.txt
ml_rul_prediction.ipynb
maintenance_simulation.ipynb
figures/

```

## Installation

```bash
# Clone the repository
git clone https://github.com/chibuikeemmanuelh/aircraft-engine-rul-prognostics-and-maintenance-queue-simulation.git

cd aircraft-engine-rul-prognostics-and-maintenance-queue-simulation

# Install dependencies
pip install -r requirements.txt
```

---


## Limitations & Future Work

- **Single dataset** — FD001 only (sea level, HPC faults only). Extend to FD002–FD004 (multiple operating conditions).
- **Static degradation rate** — Currently 7.5 flight hours/cycle. Could be made engine-specific.
- **No spare parts constraints** — Assumes unlimited parts availability.
- **Deterministic repair priorities** — Could implement stochastic or cost-based prioritization.

**Proposed extensions:**
- Reinforcement learning for dynamic server allocation
- Integration with digital twin frameworks
- Economic analysis (cost of servers vs. cost of engine degradation)

---


## References

- Frederick, D. K., DeCastro, J. A., & Litt, J. S. (2007). User's guide for the Commercial Modular Aero-Propulsion System Simulation (C-MAPSS). NASA TM-2007-215026.
- Saxena, A., Goebel, K., Simon, D., & Eklund, N. (2008). Damage propagation modeling for aircraft engine run-to-failure simulation. IEEE PHM.
- Fisher, A., Rudin, C., & Dominici, F. (2019). All Models are Wrong, but Many are Useful. JASA.
- Aircraft Commerce (2008, 2011). GE90 engine maintenance and shop visit analysis.
- MTU Maintenance Hannover (2016). Widebody engine overhaul turnaround times.
---

**Contact:** chibuike.emmanuelh@gmail.com | GitHub: @chibuikeemmanuelh