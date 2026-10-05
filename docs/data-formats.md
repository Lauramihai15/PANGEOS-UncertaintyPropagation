# Input data formats

This page lists, for every notebook, exactly which input files it expects,
where it expects to find them, and in what format, so that you can
substitute your own measurements and run the same code.

Every notebook loads its data from a folder under `data/`, using a path
relative to the notebook itself (e.g. `../data/case-piccolo-plot103-full/`). To use
your own data, create a similarly structured folder and point the notebook's
`DATA_DIR` variable (or the file path at the top of the notebook) at it.

## Piccolo Doppio (`CaseEx1_Piccolo_UncProp.ipynb`)

Folder: `data/case-piccolo-plot103-full/`

Plain-text `.txt` files, one value or one column of repeat measurements per
file, named `<detector>_<quantity>_<stage>.txt` (detector = `FLMS` or
`QEP`). The following are required for **each of the calibration stages**:

| Stage | Files needed (per detector) |
|---|---|
| Laboratory calibration | `{det}_L_lg_ITCorr_LabCal2023.txt`, `{det}_L_dk_ITCorr_LabCal2023.txt`, `{det}_L_IT_Lab2023.txt` (and the same three for `_E_`), `RadianceStd_{det}_Lab2023.txt`, `IrradianceStd_{det}_Lab2023.txt`, `coeffsNonlin_{det}_Lab2023.txt`, `u_{det}_L_std_abs_Lab2023.txt`, `u_{det}_E_std_abs_Lab2023.txt`, `u_{det}_nonlin_residual_Lab2023.txt` |
| Laboratory validation (ValGEOS) | `{det}_L_lg_ITCorr_LabVal2023.txt`, `{det}_L_dk_ITCorr_LabVal2023.txt`, `{det}_L_IT_ValLab2023.txt` (and the `_E_` equivalents) |
| Field calibration coefficients | `{det}_L_Wavelength_Val2025.txt`, `{det}_E_Wavelength_Val2025.txt`, `{det}_L_lg_ITCorr_FieldVal2025.txt`, `{det}_L_dk_ITCorr_FieldVal2025.txt`, `{det}_L_IT_Val2025.txt` (and the `_E_` equivalents) |
| Field measurement, per point | For each point `p1`…`p5`: `{det}_L_lg_ITCorr_Field2025_{plot}p{n}.txt`, `{det}_L_dk_ITCorr_Field2025_{plot}p{n}.txt`, `{det}_L_IT_Field2025_{plot}p{n}.txt` (and the `_E_` equivalents) |

Each `_lg_`/`_dk_` file is a repeat-measurement matrix: one row per
wavelength, one column per repeat scan. `_IT_` files contain a single number
(the integration time). `RadianceStd`/`IrradianceStd` are the known values of your
laboratory reference standard, one per wavelength. `coeffsNonlin` contains the
coefficients of the detector's non-linearity correction polynomial.

## ASD FieldSpec 4 and SVC HR-1024i (`CaseEx2_ASD-SVC_UncProp.ipynb`)

Folder: `data/case-asd-svc-plot103/`

| File | Format |
|---|---|
| `raw_ASD_plot103_105.csv` | Raw ASD field-day export. One row per scan. Columns 0-21: instrument metadata (must include `Acquisition Time` and `File Name`; the latter identifies each row as a target scan (`plot_<id>_<position>_*`) or a white-reference scan (`plot_<id>_fsfwr_*` for panel WP1, `plot_<id>_laurawr_*` for panel WP2)). Columns 22 onwards: one column per wavelength (nm), 350-2500 nm, raw digital numbers. |
| `raw_SVC_plot103_105.csv` | Raw SVC export with the same row types. The `File Name` is in column 8 and the wavelengths start at column 16. The wavelength grid is **not** uniformly spaced; the notebook interpolates it onto a uniform 1 nm grid, so you only need to provide the raw export as it is. |
| `panel_certificate_WP1_AM.csv` | Columns: `lambda`, `rho` (panel reflectance factor per wavelength). |
| `panel_uncertainty_WP1_AM.csv` | Columns: `lambda`, `sigma (k=2)` (panel calibration uncertainty, k=2, per wavelength). |
| `panel_certificate_WP2_LM.csv`, `panel_uncertainty_WP2_LM.csv` | The same quantities for the second panel, in that panel's certificate format (integer + decimal columns; see the `rho_panel_interp` function in the notebook for the exact parsing). |

The same two panel certificates are used for the ASD and the SVC.

## Altum and REMX, and cross-sensor agreement (`CaseEx3_Altum-REMX_UncProp.ipynb`, `MeasAgreement_4Sensors.ipynb`)

Folder: `data/case-intercomparison-plot103-full/`

| File | Format |
|---|---|
| `Indexes-7_cos_16062026.xlsx` | The full campaign results workbook. Two sheets are used for Plot 103: `Reflectance_103_all` and `Indexes103` (the same naming pattern, `Reflectance_<plot>_all` / `Indexes<plot>`, applies to the other plots). |

**`Reflectance_103_all`**: one block of columns per sensor/panel, with the data
starting at row 3 (rows 1-2 are headers). Columns 1-7 = FLMS (wavelength,
reflectance, u_random, u_systematic, empty, u_combined, U_expanded k=2); 8-14 =
QEP (same layout); 15-25 = ASD (one wavelength column, then the WP1/WP2
reflectance and uncertainty columns interleaved); 26-36 = SVC (same
pattern); 37-60 = Altum GP/WP2 and REMX GP/WP2 (6 columns each: wavelength,
reflectance, u_panel, u_plot, u_ROI, u_total).

**`Indexes103`**: one row per sensor at fixed row numbers (FLMS=8, QEP=16,
ASD_WP1=20, ASD_WP2=21, SVC_WP1=26, SVC_WP2=27, Altum_GP=31, Altum_WP2=32,
REMX_GP=37, REMX_WP2=38), with index blocks of 7 columns each (point,
value, u_r, u_s, u_homogeneity, u_c, U_k2) in a fixed order: PRI, NDVI,
NIRv, EVI, MTCI, OSAVI. Not every sensor has every index (e.g. Altum has no PRI
or MTCI, and REMX has no MTCI); those cells are empty.

To use your own campaign data, either reproduce this exact sheet and column
layout, or edit the `sensors`, `row_map` and `index_start` dictionaries in the
notebooks to match your own workbook.

---

To adapt any of these notebooks to your own instrument or campaign, keep
the column names and the file naming pattern, or edit the file paths at the
top of the notebook (`DATA_DIR`, `RAW_PATH`, etc.) to match your own naming.
