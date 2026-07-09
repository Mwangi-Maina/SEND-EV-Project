# SEND Energy Forecasting Rebuild

This is a simplified rebuild of a dissertation project predicting campus solar PV generation and electricity consumption using SEND energy data, Solcast weather data, and XGBoost models.

The project keeps the workflow readable:

1. Format raw DEOP campus power data.
2. Format Solcast weather data and estimate expected solar generation.
3. Repair likely missing solar and wind generation.
4. Load cleaned datasets consistently.
5. Create consumption and solar model features.
6. Train XGBoost forecasting models.
7. Analyse solar and consumption predictions.
8. Summarise the project.

## Folder Structure

```text
Data/       input, cleaned, and generated data files
Models/     saved XGBoost model files
Notebooks/  project notebooks
```

## Notebook Order

Run the notebooks in this order:

```text
Notebooks/Format_DEOP.ipynb
Notebooks/Format_Solcast.ipynb
Notebooks/Solar_Missing.ipynb
Notebooks/Data_Loader.ipynb
Notebooks/Feature_Loader.ipynb
Notebooks/Training_Models.ipynb
Notebooks/Analysis_Solar.ipynb
Notebooks/Analysis_Consumption.ipynb
Notebooks/Project_Summary.ipynb
```

## Setup

Create and activate a virtual environment, then install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab
```

On macOS, XGBoost may also need OpenMP:

```bash
brew install libomp
```

## Portability

The notebooks should avoid hard-coded machine-specific paths. They should find the project folder by looking for the `Data/` and `Notebooks/` folders, then use paths relative to that project folder.

## Notes

The original report trained on 2023 data and tested on March-December 2022 data. The key limitation is the small amount of historical data: the models only see one full seasonal cycle.
