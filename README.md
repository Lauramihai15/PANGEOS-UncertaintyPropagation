# PANGEOS Uncertainty Propagation — Examples of case studies

Worked examples of uncertainty propagation for multi-sensor optical
measurements under sub-optimal field conditions, extending the [PANGEOS
COST Action uq-training course](https://github.com/pangeos-cost/uq-training)
with real 2025 Norway field-campaign data.

Four notebooks, each with its documentation page:

- `CaseEx1_Piccolo_UncProp.ipynb` — Piccolo Doppio (FLMS, QEP): from the field calibration coefficients and the raw field signal to reflectance and vegetation indices
- `CaseEx2_ASD-SVC_UncProp.ipynb` — ASD FieldSpec 4 and SVC HR-1024i: reflectance and vegetation indices
- `CaseEx3_Altum-REMX_UncProp.ipynb` — UAV sensors (Altum and REMX): uncertainty budget and vegetation indices
- `MeasAgreement_4Sensors.ipynb` — cross-sensor measurement agreement (E<sub>N</sub>)

## Building the site locally

```bash
pip install -r docs-requirements.txt
mkdocs serve
```

## Running the notebooks

To run the notebooks in `notebooks/` on your own machine:

1. Install Python (miniconda recommended).
2. Install the required packages: `pip install -r notebooks-requirements.txt`
3. Launch Jupyter from the repository folder: `jupyter lab`
4. Open a notebook from the `notebooks/` folder and run its cells.

Each notebook loads its own input data from the `data/` folder. See the
[input data formats](docs/data-formats.md) if you want to use your own data.

## License

MIT — see [LICENSE](LICENSE). This repository builds on code and course
material from [pangeos-cost/uq-training](https://github.com/pangeos-cost/uq-training),
also MIT-licensed.

**Reference:** Werfeli, M., Antala, M., Abdelmajeed, A.Y.A. et al. Uncertainty propagation and intercomparison of multi-sensor measurements of vegetation stress in sub-optimal conditions. Precision Agric 27, 153 (2026). https://doi.org/10.1007/s11119-026-10455-1
