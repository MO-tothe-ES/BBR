# BBR · Historical building data

Prepare and explore historical Danish building records from **Bygnings- og Boligregistret (BBR)**. The notebooks filter registration history, harmonise selected characteristics, construct an area measure, and examine changes over time for subsequent density and accessibility analysis.

## Notebooks at a glance

| Notebook | Purpose |
| --- | --- |
| [`BBR_prefilter.ipynb`](BBR_prefilter.ipynb) | Filter raw BBR history and correct coordinates, construction years and retired use codes. |
| [`BBR_Beskrivende_Bitemporal.ipynb`](BBR_Beskrivende_Bitemporal.ipynb) | Build derived variables and produce descriptive tables and figures. |
| [`BBR_current_betingelse_tjek.ipynb`](BBR_current_betingelse_tjek.ipynb) | Explore whether a date filter reproduces Datafordeleren’s current building extract. |

## 1 · Download the inputs

Start at **[BBR Fildownload on Datafordeleren](https://datafordeler.dk/dataoversigt/bygnings-og-boligregistret-bbr/bbr-fildownload/)**. Follow its instructions to create a user, an IT system and API-key/OAuth authentication. Download and unzip:

| Setting | Main input | Additional input for the current-data check |
| --- | --- | --- |
| Register / entity | BBR / **Bygning** | BBR / **Bygning** |
| Download / format | Totaldownload / **CSV** | Totaldownload / **CSV** |
| History type | **Bitemporal** | **Current** |

The notebooks use filenames such as `BBR_V3_Bygning_TotalDownload_csv_Bitemporal_653.csv`. The extract number and filename may differ for newer downloads; update the notebook paths accordingly. For the current-data check, use matching extract reference times and set the comparison date to the current extract’s reference time.

## 2 · Set up Python and paths

Install the packages used by the notebooks:

```bash
python -m pip install jupyterlab pandas numpy matplotlib geopandas tqdm scienceplots ipywidgets
python -m jupyterlab
```

Update all input/output paths, including paths inside plotting functions, and create the output directories. The descriptive notebook’s input filename must point to the actual prefilter output. Its external Matplotlib stylesheet requires internet access; remove that style line to work offline.

## 3 · Run the workflow

### Prefilter → descriptive analysis

In **`BBR_prefilter.ipynb`**, run the imports and function definitions, then choose either the selected-column or all-column execution cell.

The default processing:

- Keeps `registreringTil` missing and `status == 6`, preserving historical **validity** periods within the retained registrations.
- Groups building histories by `id_namespace` and `id_lokalId`.
- Uses the latest dated, nonmissing construction year within each building.
- Replaces coordinates with the temporally nearest original coordinate of strictly better quality; conflicting ties remain unchanged.
- Replaces only retired use codes when the latest validity date provides one unambiguous modern code.

Corrections can be disabled individually. Chunking and temporary CSV partitions reduce memory demand while preserving input row order. Allow disk space for intermediate files.

Then open **`BBR_Beskrivende_Bitemporal.ipynb`**. It creates `modelleret_areal`, using total building area, then floors × footprint, then footprint when earlier values are missing. It also records use-code transitions and produces figures on building counts, characteristics and category changes. Outputs include a derived CSV and PNG figures. This notebook loads the data into memory.

### Optional: check current records

**`BBR_current_betingelse_tjek.ipynb`** selects rows satisfying:

```python
(virkningFra <= date) & (virkningTil.isna() | (date < virkningTil))
```

It displays sample rows and status counts alongside the downloaded Current extract. This is an exploratory comparison, **not a complete row-by-row equality test**.

## Notes on the supplied versions

- The current-check prefilter does not assign its status mask back to `chunk`, and its row counters need correction. Its subsequent date filter also expects a separate prefiltered input file.
- The descriptive notebook contains unfinished diagnostic cells referencing `bbr_harmonized` and calling `valuecounts()` instead of `value_counts()`; skip or correct these.
- The descriptive status filter allows `[6, 7, 8]`, while the main prefilter defaults to `6`.
- Harmonised values are processing assumptions; use-code harmonisation can affect measured historical transitions. `registreringTil` and `virkningTil` describe different timelines.
