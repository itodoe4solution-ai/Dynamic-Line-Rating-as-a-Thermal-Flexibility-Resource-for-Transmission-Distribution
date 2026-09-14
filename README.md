# Dynamic Line Rating as a Thermal Flexibility Resource for Transmission–Distribution Coordination of EVs and PV under Forecast Uncertainty

This repository contains the **code, derived study data, frozen scenario definitions, model inputs, statistical outputs, and publication figures/tables** supporting the manuscript submitted to *Electric Power Systems Research (EPSR)*.

The study evaluates **dynamic line rating (DLR)** against an independently specified **static thermal rating (STR)** in a controlled transmission–distribution benchmark. The workflow combines IEEE Std 738-style line-rating calculations, a canonical IEEE-118 transmission model, heterogeneous IEEE-33 distribution feeders, reconstructed EV charging, model-derived PV, uncertainty-aware receding-horizon scheduling, and independent nonlinear AC/network-security validation.

## Repository purpose

The repository is organized so that a reader can:

1. inspect the exact processed/derived data and frozen operating windows used in the study;
2. review the Python implementation for data preparation, DLR, forecasting, optimization, statistical inference, ACPF/ACOPF, selected N-1 analysis, and IEEE-33 voltage-control sensitivity;
3. reproduce the principal Phase 4–6 analyses from the supplied derived inputs; and
4. regenerate source data from the original providers when API access is available.

## Main manuscript evidence

The repository contains the outputs underlying the central paper claims, including:

- paired STR–DLR scheduling results and feasibility restoration (`09_simulation_inputs/results/phase4*`);
- window-cluster statistical inference and sign-flip tests (`phase4e_*`);
- IEEE-118 ACPF/ACOPF validation (`phase5a_*`, `phase5b_*`);
- selected N-1 validation (`phase5c_*`);
- IEEE-39 directional robustness (`phase5d_*`);
- voltage-control sensitivity with OLTC and Volt–VAR (`phase5f_*`); and
- consolidated publication tables, figures, and claim-evidence files (`09_simulation_inputs/results/phase6/`).

The manuscript's principal four-hour high-stress result is the observed restoration of **22 of 130 paired cases (16.9%)** under DLR, with lower rating-normalized thermal utilization among pairs feasible under both policies. The study does **not** claim a statistically supported systematic improvement in EV unserved energy.

## Repository structure

```text
.
├── README.md
├── config.yaml
├── requirements.txt
├── .env.example
├── 00_documentation/          # source provenance and research notes
├── 01_raw/                    # source-data instructions only; raw downloads excluded
├── 03_processed/              # synchronized EV, PV, weather and EIA series
├── 04_master/                 # classified 2019 master study dataset
├── 05_quality_control/        # QC, reconstruction and validation reports
├── 06_scenarios/              # frozen operating-window definitions
├── 07_forecasts/              # forecast-model selection and uncertainty outputs
├── 08_dlr/                    # STR/DLR time-series products
├── 09_simulation_inputs/
│   ├── ieee33_opendss/        # IEEE-33 feeder/OpenDSS files
│   ├── phase2_td/             # T-D mapping, corridor and feeder inputs
│   └── results/               # Phase 4–6 numerical evidence
├── epsr_pipeline/             # reusable data/model functions
├── notebooks/                 # master research notebook(s)
├── scripts/                   # complete scripted workflow
└── tests/                     # core automated tests
```

## Data sources

The primary chronology is geographically coherent around **Caltech/Pasadena and Southern California**. Source provenance is recorded in `00_documentation/data_sources.csv`. The major external sources are:

- **Caltech ACN-Data** — EV charging session metadata and available charging-current information;
- **NSRDB GOES CONUS PSM v4** — meteorology and irradiance;
- **NOAA/NCEI ISD** — independent weather validation;
- **EIA-930** — California ISO system-demand chronology;
- **pvlib** — model-derived PV power; and
- **IEEE Std 738-2023** — thermal-rating methodology/standard reference.

Raw third-party downloads are intentionally **not redistributed**. See `01_raw/README.md` for provider links. The repository contains the derived study products used for the paper and the scripts required to retrieve/rebuild source data where access permits.

## Installation

Python **3.11** is recommended for the reproducibility environment.

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux/macOS
# source .venv/bin/activate

pip install -r requirements.txt
```

If you intend to re-download source data, copy `.env.example` to `.env` and add your own provider credentials. **Never commit `.env` or API keys.**

## Restore the compressed corridor-rating input

One derived file is stored as gzip to keep every repository file small enough for ordinary GitHub upload. After cloning/downloading the repository, run:

```bash
python scripts/00_restore_repository_inputs.py
```

This creates:

```text
09_simulation_inputs/phase2_td/corridor_ratings_15min.csv
```

from the tracked file `corridor_ratings_15min.csv.gz`.

## Quick reproducibility route

For readers interested primarily in the paper's reported findings rather than re-downloading all external source data:

```bash
python scripts/00_restore_repository_inputs.py
python scripts/26_run_phase4c_receding.py
python scripts/27_run_phase4d_ablation.py
python scripts/28_run_phase4e_statistics.py
python scripts/30_run_phase5a_acpf.py
python scripts/31_run_phase5b_acopf.py
python scripts/32_run_phase5c_n1.py
python scripts/33_run_phase5d_ieee39.py
python scripts/34_run_phase5e_consolidation.py
python scripts/35_run_phase5f_voltage_control.py
python scripts/37_run_phase6a_final_audit.py
python scripts/38_run_phase6b_consolidation.py
python scripts/39_run_phase6c_figures.py
python scripts/40_run_phase6d_tables.py
python scripts/41_run_phase6e_publication_package.py
```

The scripts use repository-relative path resolution. Run them from the repository root unless a script states otherwise.

## Master notebook

`notebooks/EPSR_Final_Master_Research_Notebook.ipynb` provides a phase-by-phase research workflow. The default execution switches are conservative so that expensive rebuilds are not triggered automatically. Enable only the phases you intend to rerun.

## Full source-data rebuild

The original data pipeline is retained in `scripts/00_prepare_structure.py` through `scripts/23_run_research_stage1.py`. A full rebuild requires access to the external providers and, for some sources, API credentials. The main sequence is documented in `00_documentation/` and the scripts themselves.

Important study boundaries:

- ACN session energy/connection information is empirical; the synchronized interval EV profile is reconstructed/derived.
- PV output is model-derived from NSRDB irradiance and meteorology, not measured PV production.
- EIA-930 provides a regional chronology driver rather than measured IEEE-33 feeder demand.
- The five IEEE-118 DLR corridors use a controlled common conductor/weather benchmark; they are not claimed to represent physical IEEE-118 assets.
- The reduced-order scheduling model is not treated as a network-wide AC security certificate; nonlinear validation is performed separately.
- Short-horizon DLR chronology is treated as known in the final scheduling experiments; probabilistic DLR forecast uncertainty is outside the study scope.

## Key configuration

`config.yaml` records the frozen study settings, including:

- 2019 study chronology;
- Caltech/Pasadena reference location;
- 15-minute study interval;
- 795-kcmil Drake ACSR conductor assumptions;
- 138-kV line voltage;
- explicit STR design-weather assumptions;
- PV configuration; and
- data-quality and forecasting settings.

## Tests

Run the available core tests with:

```bash
python -m pytest tests
```
## Data and code availability statement for the manuscript

After the GitHub repository is public, the manuscript can use the following wording (replace the placeholder with the final repository URL):

> **Data and code availability:** The code, configuration files, derived study data, frozen scenario definitions, and numerical outputs supporting the findings of this study are publicly available at **[GITHUB_REPOSITORY_URL]**. Raw third-party data from Caltech ACN-Data, NSRDB, NOAA/NCEI, and EIA-930 are not redistributed in the repository; source links, provenance information, and retrieval/processing scripts are provided to support reproducibility subject to the original providers' access and usage terms.

## Citation

If you use this repository, please cite the associated manuscript after publication. Until a DOI is assigned, cite the manuscript title and authors as provided in the submitted paper.

## Authors

- Emmanuel Samson Itodo
- Jiashen Teh (corresponding author)
- Ching-Ming Lai

## License and third-party material

No new blanket license is imposed by this packaging step.
