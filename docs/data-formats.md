# Input data formats

This page lists, for every notebook, exactly which input files it expects,
where it expects to find them, and in what format, so that you can
substitute your own measurements and run the same code.

Every notebook loads its data from a folder under `data/`, using a path
relative to the notebook itself (e.g. `../data/piccolo-field/`). To use
your own data, create a similarly structured folder and point the notebook's
`DATA_DIR` variable (or the file path at the top of the notebook) at it.

## Piccolo Doppio (`CaseEx1_Piccolo_UncProp.ipynb`)

Settings at the top of the notebook: `SENSOR` (`"FLMS"` or `"QEP"`), `PLOT_ID` (any plot of the campaign), `N_POINTS`
(measurement points per plot), the relative cosine-response uncertainty of each group of plots (`COS_GROUPS`) and the three
folders described below.

### Field measurements: `data/piccolo-field/`

Plain-text `.txt` files, for the points `p1`…`p5` of each plot:

| File | Format |
|---|---|
| `<det>_<L or E>_lg_ITCorr_Field2025_<plot>p<n>.txt`, `<det>_<L or E>_dk_ITCorr_Field2025_<plot>p<n>.txt` | Light (`lg`) and dark (`dk`) scans of the radiance (`L`) or irradiance (`E`) channel: one row per wavelength, one column per repeat scan. |
| `<det>_<L or E>_IT_Field2025_<plot>p<n>.txt` | Integration time: a single number. |
| `<det>_<L or E>_Wavelength_Val2025.txt` | Wavelength grid of the field measurements. |

The detector `<det>` is `FLMS` or `QEP`.

### Calibration coefficients: `data/piccolo-coefficients/`

The coefficients of each calibration stage with their uncertainties, per detector:

| File | Content |
|---|---|
| `<det>_Stage1_LabCalCoeffs_L.txt`, `<det>_Stage1_LabCalCoeffs_E.txt` | Stage 1, laboratory calibration coefficients. Three columns: value, random uncertainty, systematic uncertainty; one row per wavelength. |
| `<det>_Stage2_ValGEOS_L.txt`, `<det>_Stage2_ValGEOS_E.txt` | Stage 2, validated radiance and irradiance (ValGEOS). Same format. |
| `<det>_Stage3_FieldCalCoeffs_L.txt`, `<det>_Stage3_FieldCalCoeffs_E.txt` | Stage 3, field calibration coefficients. Same format. |
| `<det>_NonlinPolynomial.txt` | Coefficients of the detector non-linearity correction polynomial. |
| `<det>_NonlinResidual_u_rel.txt` | Relative systematic uncertainty (%) of the non-linearity correction, one value per wavelength. |
| `<det>_<L or E>_Wavelength_Lab2023.txt`, `<det>_<L or E>_Wavelength_ValLab2023.txt` | Wavelength grids of the laboratory measurements (used for the plots). |

### Raw calibration measurements (optional, not part of the repository)

If the folder `data/piccolo-calibration-raw/` exists, the notebook computes Stages 1-3 from the raw calibration
measurements instead of reading the coefficients. With `SAVE_COEFFICIENTS = True` it then writes the coefficients to
`data/piccolo-coefficients/`. The folder must contain, per detector:

| Stage | Files needed |
|---|---|
| Laboratory calibration | `<det>_<L or E>_lg_ITCorr_LabCal2023.txt`, `<det>_<L or E>_dk_ITCorr_LabCal2023.txt`, `<det>_<L or E>_IT_Lab2023.txt`, `RadianceStd_<det>_Lab2023.txt`, `IrradianceStd_<det>_Lab2023.txt`, `coeffsNonlin_<det>_Lab2023.txt`, `u_<det>_L_std_abs_Lab2023.txt`, `u_<det>_E_std_abs_Lab2023.txt`, `u_<det>_nonlin_residual_Lab2023.txt` |
| Laboratory validation (ValGEOS) | `<det>_<L or E>_lg_ITCorr_LabVal2023.txt`, `<det>_<L or E>_dk_ITCorr_LabVal2023.txt`, `<det>_<L or E>_IT_ValLab2023.txt` |
| Field calibration | `<det>_<L or E>_lg_ITCorr_FieldVal2025.txt`, `<det>_<L or E>_dk_ITCorr_FieldVal2025.txt`, `<det>_<L or E>_IT_Val2025.txt` |

The `_lg_`/`_dk_` files are repeat-measurement matrices (one row per wavelength, one column per repeat scan), the `_IT_`
files contain the integration time, `RadianceStd`/`IrradianceStd` are the known values of your laboratory reference standard (one per
wavelength), and `coeffsNonlin` contains the non-linearity correction polynomial coefficients.

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

The plot is selected with the variable `PLOT_ID` at the top of the notebooks. To use your own campaign data, either reproduce this exact sheet and column
layout, or edit the `sensors`, `row_map` and `index_start` dictionaries in the
notebooks to match your own workbook.

---

To adapt any of these notebooks to your own instrument or campaign, keep
the column names and the file naming pattern, or edit the file paths at the
top of the notebook (`DATA_DIR`, `RAW_PATH`, etc.) to match your own naming.
