# Input data formats

This page lists, for every notebook, exactly what input files it expects,
where it expects to find them, and what format each file must be in — so
you can substitute your own measurements and run the same code.

Every notebook loads its data from a folder under `data/`, using a path
relative to the notebook itself (e.g. `../data/case1b-plot103/`). To use
your own data, create a similarly-structured folder and point the
notebook's `DATA_DIR` variable at it.

## Case 1b — plot-mean aggregation (`case_1b-Ex1_PlotMean_Uncertainty.ipynb`)

Folder: `data/case1b-plot103/`

| File | Format |
|---|---|
| `point1_reflectance.csv` … `point5_reflectance.csv` | One CSV per measurement point (5 total). Columns: wavelength (nm) and reflectance value, plus its random and systematic uncertainty. Already-calibrated reflectance is expected as input — this notebook does not calibrate raw data. |
| `official_plot_mean_reflectance.csv` | Optional. Only needed to reproduce the validation check against a reference plot-mean. |

## Piccolo Doppio — complete calibration pipeline (`case_piccolo-Ex_CompleteCalibrationPipeline.ipynb`)

Folder: `data/case-piccolo-plot103-full/`

Plain-text `.txt` files, one value or one column of repeat measurements per
file, named `<detector>_<quantity>_<stage>.txt` (detector = `FLMS` or
`QEP`). Required for **each of the three calibration stages**:

| Stage | Files needed (per detector) |
|---|---|
| Laboratory calibration | `{det}_L_lg_ITCorr_LabCal2023.txt`, `{det}_L_dk_ITCorr_LabCal2023.txt`, `{det}_L_IT_Lab2023.txt` (and the same three for `_E_`), `RadianceStd_{det}_Lab2023.txt`, `IrradianceStd_{det}_Lab2023.txt`, `coeffsNonlin_{det}_Lab2023.txt`, `u_{det}_L_std_abs_Lab2023.txt`, `u_{det}_E_std_abs_Lab2023.txt`, `u_{det}_nonlin_residual_Lab2023.txt` |
| Laboratory validation (ValGEOS) | `{det}_L_lg_ITCorr_LabVal2023.txt`, `{det}_L_dk_ITCorr_LabVal2023.txt`, `{det}_L_IT_ValLab2023.txt` (and the `_E_` equivalents) |
| Field calibration coefficients | `{det}_L_Wavelength_Val2025.txt`, `{det}_E_Wavelength_Val2025.txt`, `{det}_L_lg_ITCorr_FieldVal2025.txt`, `{det}_L_dk_ITCorr_FieldVal2025.txt`, `{det}_L_IT_Val2025.txt` (and the `_E_` equivalents) |
| Field measurement, per point | For each point `p1`…`p5`: `{det}_L_lg_ITCorr_Field2025_{plot}p{n}.txt`, `{det}_L_dk_ITCorr_Field2025_{plot}p{n}.txt`, `{det}_L_IT_Field2025_{plot}p{n}.txt` (and the `_E_` equivalents) |

Each `_lg_`/`_dk_` file is a repeat-measurement matrix: one row per
wavelength, one column per repeat scan. `_IT_` files are a single number
(integration time). `RadianceStd`/`IrradianceStd` are your laboratory
reference standard's known values, one per wavelength. `coeffsNonlin` is
the detector's non-linearity correction polynomial coefficients.

## ASD FieldSpec 4 (`case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb`, `case_asdsvc-Ex2_VegetationIndices.ipynb`)

Folder: `data/case-asd-svc-plot103/`

| File | Format |
|---|---|
| `raw_ASD_plot103_105.csv` | Raw ASD field-day export. One row per scan. Columns 0-21: instrument metadata (must include `Acquisition Time` and `File Name`, the latter identifying each row as a target scan (`plot_<id>_<position>_*`) or white-reference scan (`plot_<id>_fsfwr_*` for panel WP1, `plot_<id>_laurawr_*` for panel WP2)). Columns 22 onward: one column per wavelength (nm), 350-2500nm, raw digital numbers. |
| `panel_certificate_WP1_AM.csv` | Columns: `lambda`, `rho` (panel reflectance factor per wavelength). |
| `panel_uncertainty_WP1_AM.csv` | Columns: `lambda`, `sigma (k=2)` (panel calibration uncertainty, k=2, per wavelength). |
| `panel_certificate_WP2_LM.csv`, `panel_uncertainty_WP2_LM.csv` | Same quantities, second panel's certificate format (integer + decimal columns; see the notebook's `rho_panel_interp` function for the exact parsing). |
| `asd_reflectance_repeats.csv`, `panel_uncertainty_k2.csv`, `official_vegetation_indices.csv` | Reference/validation data for Ex2 — not required to run your own data through the pipeline, only to reproduce the validation check. |

## SVC HR-1024i (`case_svc-Ex1_FieldUncertaintyBudget.ipynb`)

Folder: `data/case-asd-svc-plot103/`

Same structure as ASD above, except:
- `raw_SVC_plot103_105.csv` — File Name at column 8, wavelengths start at column 16, and the wavelength grid is **not** uniformly spaced (interpolated onto a uniform 1nm grid inside the notebook — no action needed on your part beyond providing the raw export as-is).
- Uses the **same** panel certificate files as ASD (`panel_certificate_WP1_AM.csv`, `panel_certificate_WP2_LM.csv`, and their uncertainty files).

## UAV — Altum & REMX (`case_uav-Ex1_AltumREMX_MethodComparison.ipynb`, `case_uav-Ex2_VegetationIndices.ipynb`)

Folder: `data/case-uav-plot103/`

| File | Format |
|---|---|
| `altum_greypanel.csv`, `altum_whitepanel.csv` | Per-band reflectance for the Altum camera, one calibration method per file. Columns: wavelength (nm), reflectance, `u_panel`, `u_plot`, `u_roi`. |
| `remx_greypanel.csv`, `remx_whitepanel.csv` | Same columns, for the REMX camera. |
| `official_altum_indices.csv`, `official_remx_indices.csv` | Reference/validation data only. |

## Cross-sensor intercomparison (`case_intercomparison-Ex1_ASD_vs_SVC.ipynb`)

Folders: `data/case-intercomparison-plot103/` and `data/case-asd-svc-plot103/`

| File | Format |
|---|---|
| `svc_raw/*.sig` | Raw SVC instrument files (plain text), one per scan — self-describing header plus a data block with wavelength, reference DN, target DN, and reflectance ratio, and two acquisition timestamps. |
| `official_svc_reflectance.csv`, `official_asd_wp1_reference.csv` | Reference reflectance spectra used for the agreement test. |
| `asd_u_total_all_plots.csv`, `svc_u_total_all_plots.csv` | Combined uncertainty per plot/panel — columns named `<plot>_<panel>` (e.g. `103_AM`). |
| `official_ASD_SVC_agreement_k1.csv`, `official_ASD_SVC_agreement_k2.csv` | Reference E_N agreement values — validation only. |

---

To adapt any of these notebooks to your own instrument or campaign, keep
the column names and file-naming pattern the same, or edit the file paths
at the top of the notebook (`DATA_DIR`, `RAW_PATH`, etc.) to match your own
naming.
