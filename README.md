# ICP-MS TRA Data Extraction

Jupyter notebook for processing time-resolved ICP-MS data acquired with a SeaFAST SP3 inline preconcentration system. Extracts element concentrations and propagated uncertainties from raw transient-signal files.

## Notebook

**`DataExtraction_Final.ipynb`** — main working notebook. Run cells top to bottom after updating the config cell.

## Workflow

1. **Imports** — standard scientific Python stack.
2. **Config cell** — all user-adjustable parameters (edit this before each run).
3. **Function definitions** — `_read_file`, `file_import`, `error_prep_calc`, `simple_error_calc`, `calibration_calculation`.
4. **Optional: window visualiser** — plots the signal plateau to help set `T_START_S` / `T_STOP_S`.
5. **`file_import`** — loads all files in `DATA_DIR`, subtracts background, normalises to In115, integrates within the time window, and plots TRA traces.
6. **In115 drift check** — bar chart of per-file In115 integrated signal; warns if the run-to-run spread exceeds 20% of the median.
7. **Spike detection** — flags isolated anomalous points within the integration window using a rolling-median / MAD algorithm.
8. **Calibration** — fits a linear model (forced-origin by default) to the standards in `CAL_STDS`, converts In-normalised signals to concentrations, propagates errors.
9. **F-test diagnostic** — tests per element whether a non-zero intercept is statistically warranted; guides the choice of `FIT_INTERCEPT`.
10. **Output** — displays concentration and error tables; writes `Results.xlsx` (three sheets: concentrations, errors, combined interleaved) into `DATA_DIR`.

## Configuration

All parameters are in the second cell of the notebook.

| Parameter | Description |
|---|---|
| `DATA_DIR` | Path to folder containing the TRA files |
| `FILE_EXTENSION` | `'FIN2'` or `'TXT'` |
| `BACKGROUND_FILE` | Exact file stem of the blank/background run |
| `T_START_S` | Integration window start (seconds) |
| `T_STOP_S` | Integration window end (seconds) |
| `SN_IN_RATIO` | ¹¹⁵Sn/¹¹⁸Sn natural abundance ratio (IUPAC 2016 default: 0.0140) |
| `CAL_STDS` | Dict mapping calibration file stems to known concentrations |
| `MIN_R2` | Minimum acceptable R² for a calibration curve (default 0.99) |
| `FIT_INTERCEPT` | `'FIXED'` (forced origin) or `'VARIABLE'` (free intercept) |
| `REPS` | Replicate injections per sample — reduces reported uncertainty if > 1 |

### Setting the integration window

Run the **window visualiser cell** (cell 5) to identify the signal plateau. It auto-detects the elution band and plots it against the current config window. Adjust `IDX_START`, `IDX_STOP`, and `PEAK_BUFFER` (rows) until the window covers the plateau, then copy the printed `T_START_S` / `T_STOP_S` values into the config cell.

### Choosing `FIT_INTERCEPT`

Default is `'FIXED'` (forced-origin regression, appropriate when the blank has been subtracted and calibration standards are clean). Switch to `'VARIABLE'` if the **F-test diagnostic cell** reports significant non-zero intercepts for multiple elements with `int_pct` > ~5%, which indicates contamination in the calibration standards.

## File formats

Both file types export from the ICP-MS instrument software:

- **FIN2** — tab-delimited; metadata header of 8 lines; element columns in counts-per-second.
- **TXT** — semicolon-delimited; single header line; same column structure.

## Dependencies

Create and activate the included conda environment:

```bash
conda env create -f environment.yml
conda activate icpms-tra
```

## Output

`Results.xlsx` is written to `DATA_DIR` and contains three sheets:

| Sheet | Contents |
|---|---|
| `Concentrations ({units})` | One row per sample, one column per element |
| `Errors ({units})` | Propagated 95% prediction intervals, same layout |
| `Combined ({units})` | Interleaved columns: `element`, `element_err`, … |
