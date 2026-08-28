# Flexible EV Charging for Renewable Surplus Absorption

This repository contains a master's dissertation project on **forecast-informed EV demand response** for Keele University. The work extends campus energy forecasting into a complete decision workflow: estimate renewable surplus, predict when it may occur, model flexible EV charging demand, schedule charging into useful windows, and explain the resulting actions.

## Problem

Keele can generate solar and wind energy that exceeds campus demand at some times. If flexible demand is not coordinated, this energy may be exported at low value, constrained, curtailed, or replaced later by grid energy. This project focuses on the middle decision layer:

```text
forecast surplus -> identify flexible EV demand -> schedule charging -> explain the action
```

The project uses EV charging because workplace vehicles are often connected for longer than the minimum time needed to charge. That spare dwell time creates flexibility, provided the schedule still respects arrival times, departure times, charger power, site limits, and required energy.

## Important Scope Note

The surplus signal is a **potential renewable-surplus proxy**:

```text
surplus_kw = max((solar_kw + wind_kw) - consumption_kw, 0)
```

It identifies intervals where measured or estimated renewable generation exceeds campus consumption. It is not direct proof of physical curtailment unless export-limit or curtailment telemetry is added.

ACN-Data is also not treated as observed Keele driver behaviour. It supplies empirical workplace charging patterns that are used to build synthetic Keele EV scenarios.

## Repository Structure

```text
Data/
  ACN/
    acquired/       Standardised ACN source files
    chunks/         Downloaded ACN API chunks
    processed/      Cleaned ACN sessions and Keele EV scenarios
  Cleaned/          Cleaned campus datasets
  DEOP/             Keele SEND/DEOP campus energy data
  Processed/        Integrated SEND, Solcast and surplus features
  Results/          Metrics, schedules, explanations and final tables
  Solcast/          Weather and irradiance data

Drafts/             Dissertation Word documents and rendered QA outputs
Figures/            Plots used by the dissertation
Models/             Saved forecasting model artefacts
Notebooks/          Reproducible project workflow
Scripts/            Dissertation and utility scripts
```

## Environment Setup

The project uses Python 3.12+ and Poetry.

```bash
cd /Users/mac/Dissertation/SEND-EV-Project
python3 -m pip install poetry
poetry install
poetry run jupyter lab
```

Useful dependency notes:

- `cvxpy` is required for optimisation notebooks.
- `pyarrow` is required for Parquet output. If it is unavailable, some notebooks save CSV fallbacks.
- On macOS, `xgboost` may require OpenMP:

```bash
brew install libomp
```

If Poetry is not available, install the packages listed in `pyproject.toml`.

## ACN API Token

Existing ACN files can be reused from `Data/ACN/acquired/` and `Data/ACN/chunks/`.

To refresh ACN data from the API, set:

```bash
export ACN_API_TOKEN="your_token_here"
```

Then run `Notebooks/11A_ACN_Data_Acquisition.ipynb`.

## Notebook Run Order

Run the notebooks from the project root. This order rebuilds the whole project.

| Step | Notebook | Purpose | Main outputs |
|---:|---|---|---|
| 1 | `Notebooks/Format_DEOP.ipynb` | Clean and format Keele SEND/DEOP energy data. | Cleaned campus energy tables |
| 2 | `Notebooks/Format_Solcast.ipynb` | Load Solcast weather and irradiance data. | Cleaned Solcast features |
| 3 | `Notebooks/Solar_Missing.ipynb` | Repair missing or implausible solar and wind values. | Repaired generation series |
| 4 | `Notebooks/Data_Loader.ipynb` | Define shared data-loading helpers. | Reusable loader functions |
| 5 | `Notebooks/Feature_Loader.ipynb` | Build time, lag, rolling and weather features. | Forecast-ready feature tables |
| 6 | `Notebooks/Training_Models.ipynb` | Define forecasting models and metrics. | Reusable XGBoost model classes |
| 7 | `Notebooks/Analysis_Solar.ipynb` | Analyse and model solar generation. | Solar model outputs and checks |
| 8 | `Notebooks/Analysis_Consumption.ipynb` | Analyse and model campus consumption. | Consumption model outputs and checks |
| 9 | `Notebooks/Renewable_Surplus.ipynb` | Build the renewable-surplus target and events. | `daily_surplus_summary.csv`, `surplus_events.csv` |
| 10 | `Notebooks/11A_ACN_Data_Acquisition.ipynb` | Acquire or validate ACN workplace charging data. | ACN raw and standardised files |
| 11 | `Notebooks/11B_ACN_Data_Cleaning_and_Flexibility.ipynb` | Describe raw behaviour, clean ACN sessions and estimate flexibility. | ACN quality, exclusion and site statistics |
| 12 | `Notebooks/EV_Scenario_Generation.ipynb` | Create synthetic Keele EV charging scenarios. | `keele_ev_scenarios.parquet`, `scenario_summary.csv` |
| 13 | `Notebooks/Charging_Baselines.ipynb` | Build immediate and fixed-delay charging baselines. | `baseline_metrics.csv`, baseline schedules |
| 14 | `Notebooks/14_EV_Charging_Optimisation.ipynb` | Optimise EV charging against actual and forecast surplus. | `optimisation_metrics.csv`, optimised schedules |
| 15 | `Notebooks/15_Rolling_Horizon_Control.ipynb` | Test repeated short-horizon charging control. | `rolling_horizon_metrics.csv`, rolling schedules |
| 16 | `Notebooks/16_SHAP_Analysis.ipynb` | Explain the surplus forecast with SHAP or model contributions. | `shap_feature_importance.csv`, SHAP figures |
| 17 | `Notebooks/17_Counterfactual_Explanations.ipynb` | Turn schedule differences into vehicle-level explanations. | `counterfactual_recommendations.csv`, `counterfactual_metrics.csv` |
| 18 | `Notebooks/18_Final_Evaluation.ipynb` | Combine all metrics into final tables and plots. | final strategy, bootstrap and sensitivity outputs |
| 19 | `Notebooks/Project_Summary.ipynb` | Summarise the full reproducible workflow. | Project-level overview |

## Workflow Summary

### 1. Campus Energy Preparation

`Format_DEOP.ipynb`, `Format_Solcast.ipynb`, `Solar_Missing.ipynb`, `Data_Loader.ipynb`, and `Feature_Loader.ipynb` prepare the SEND/DEOP and Solcast data. These notebooks create the clean 5-minute data foundation used for forecasting and optimisation.

### 2. Renewable Surplus

`Renewable_Surplus.ipynb` calculates potential surplus from renewable generation and campus consumption. It also identifies surplus events, which matter because many opportunities are short rather than one smooth daily block.

Key outputs:

```text
Data/Results/daily_surplus_summary.csv
Data/Results/surplus_events.csv
Data/Processed/send_solcast_surplus_5min.parquet
Figures/renewable_surplus_profile.png
```

### 3. Forecasting

The forecasting notebooks model solar generation, consumption and renewable surplus using engineered time, lag, rolling and weather features. The dissertation evaluates the surplus forecast as a decision input, not only as a machine-learning score.

Key outputs:

```text
Data/Results/surplus_test_features.csv
Data/Results/surplus_predictions.csv
Models/Forecasting/xgb_surplus.joblib
```

### 4. EV Data and Flexibility

`11A_ACN_Data_Acquisition.ipynb` gets the ACN data. `11B_ACN_Data_Cleaning_and_Flexibility.ipynb` first describes the raw data behaviour before cleaning, then validates sessions and estimates flexibility from dwell time, required energy and charger power.

Key outputs:

```text
Data/ACN/processed/acn_sessions_cleaned.parquet
Data/ACN/processed/acn_sessions_optimisation_ready.parquet
Data/Results/acn_quality_summary.csv
Data/Results/acn_site_statistics.csv
Figures/acn_arrival_distribution.png
Figures/acn_energy_distribution.png
Figures/acn_flexibility_distribution.png
```

### 5. Synthetic Keele EV Scenarios

`EV_Scenario_Generation.ipynb` maps ACN workplace charging behaviour onto Keele surplus-rich dates. Scenarios vary fleet size, participation rate, random seed and site power limit.

Key outputs:

```text
Data/ACN/processed/keele_ev_scenarios.parquet
Data/Results/scenario_summary.csv
```

### 6. Baselines and Optimisation

`Charging_Baselines.ipynb` builds simple comparison strategies. `14_EV_Charging_Optimisation.ipynb` uses constrained optimisation to move charging into surplus windows while preserving driver service. `15_Rolling_Horizon_Control.ipynb` tests a more operational control approach where schedules are repeatedly updated.

Key outputs:

```text
Data/Results/baseline_metrics.csv
Data/Results/optimisation_metrics.csv
Data/Results/rolling_horizon_metrics.csv
Data/Results/final_model_comparison.csv
Figures/baseline_charging_profile.png
Figures/optimised_charging_profile.png
```

### 7. Explainability

`16_SHAP_Analysis.ipynb` explains the surplus forecast. `17_Counterfactual_Explanations.ipynb` explains charging changes by comparing factual and counterfactual schedules.

Key outputs:

```text
Data/Results/shap_feature_importance.csv
Data/Results/counterfactual_recommendations.csv
Data/Results/counterfactual_metrics.csv
Figures/shap_global_bar.png
Figures/shap_beeswarm.png
Figures/shap_waterfall_high_surplus.png
```

### 8. Final Evaluation and Dissertation

`18_Final_Evaluation.ipynb` combines the forecasting, charging and explanation results. `Project_Summary.ipynb` gives a notebook-level overview of the whole project. The dissertation Word document is generated from the scripts in `Scripts/`.

Key outputs:

```text
Data/Results/final_operational_metrics.csv
Data/Results/final_strategy_summary.csv
Data/Results/final_sensitivity_analysis.csv
Data/Results/final_bootstrap_confidence_intervals.csv
Figures/final_renewable_absorption.png
Figures/final_residual_surplus.png
Figures/final_grid_energy.png
Figures/final_service_rate.png
Drafts/SEND_EV_Masters_Dissertation_Final.docx
```

## Current Results Snapshot

The latest generated outputs show:

- 671 daily surplus records.
- 4,035 surplus events.
- The highest-surplus day is 2023-07-07, with about 12,713.8 kWh of potential surplus.
- ACN cleaned-site medians show several hours of charging flexibility at Caltech, JPL and office001.
- The actual-surplus optimiser increases renewable absorption by about 448.1 kWh compared with immediate charging while keeping 100% service in the capped operational subset.
- The XGBoost-informed optimiser keeps 100% service but gives a smaller renewable-absorption gain, showing that forecast timing is important.
- Counterfactual explanations are feasible for almost all tested cases and translate schedule changes into vehicle-level recommendations.

## Verification Runs and Full Runs

Some notebooks use capped scenario subsets for quick development. Check for `MAX_SCENARIOS` in:

```text
Notebooks/Charging_Baselines.ipynb
Notebooks/14_EV_Charging_Optimisation.ipynb
Notebooks/15_Rolling_Horizon_Control.ipynb
```

Use a small cap for testing. Set `MAX_SCENARIOS = None` only for full experiments.

## Main Metrics

Forecasting metrics:

- MAE
- RMSE
- R-squared
- feature contribution importance

Operational metrics:

- service rate
- renewable energy absorbed
- residual surplus
- EV grid energy
- unserved energy
- peak EV demand

Explanation metrics:

- feasible explanation share
- changed intervals
- schedule distance
- additional surplus-window energy

## Troubleshooting

If Parquet saving fails, install `pyarrow`:

```bash
poetry add pyarrow
```

If optimisation fails with `ModuleNotFoundError: No module named 'cvxpy'`, install CVXPY:

```bash
poetry add cvxpy
```

If ACN acquisition fails:

- check `ACN_API_TOKEN`;
- or reuse the existing ACN files already saved under `Data/ACN/`.

If optimisation is slow:

- use the capped scenario subset;
- reduce fleet size;
- reduce `MAX_SCENARIOS`;
- run the full optimisation only for final results.

If SHAP fails because of package compatibility, use the XGBoost contribution fallback built into the workflow.

## Research Limitations

- Potential surplus is not the same as measured curtailment.
- ACN behaviour is transferred into synthetic Keele EV scenarios and should be validated with Keele charger logs in future work.
- Perfect-information optimisation is an upper benchmark, not a deployable controller.
- Forecast-informed and rolling-horizon methods are more realistic but depend on interval-level forecast timing.
- The project does not model vehicle-to-grid operation, battery degradation, tariff response, charger communication failures or user override behaviour.

## One-Line Summary

This project shows how forecasted renewable surplus can be converted into explainable EV charging actions, while keeping the limits of the available data explicit.
