# PANGEOS Uncertainty Propagation — Case Studies

Worked examples of uncertainty propagation for multi-sensor optical
measurements under sub-optimal field conditions, extending the [PANGEOS
COST Action uq-training course](https://github.com/pangeos-cost/uq-training)
with real 2025 Norway field-campaign data.

Four cases, each with documentation and runnable notebooks:

- Piccolo Doppio (FLMS/QEP)
- ASD FieldSpec 4 & SVC HR-1024i
- UAV sensors (Altum & REMX)
- Cross-sensor intercomparison

**Reference:** Werfeli, M., Antala, M., Abdelmajeed, A.Y.A., Sánchez-Virosta,
Á., Halem, Z., Merrington, A., El-Mejjaouy, Y., Petrovic, B., Hueni, A.,
Bouras, E.H., Kefauver, S.C., Shafiee, S., Rastogi, A., Mihai, L. *Uncertainty propagation and intercomparison of multi-sensor
measurements of vegetation stress in sub-optimal conditions.*

## Building the site locally
pip install -r docs-requirements.txt
mkdocs serve

## Running the notebooks
To run the notebooks in `notebooks/` on your own machine:

1. Install Python (miniconda recommended).
2. Install the required packages: pip install -r notebooks-requirements.txt
3. Launch Jupyter from the repository folder: jupyter lab
4. Open a notebook from the `notebooks/` folder and run its cells.

Each notebook loads its own input data from the `data/` folder — see each
case's page for the expected file format if you want to use your own data.

## License

MIT — see [LICENSE](LICENSE). This repository builds on code and course
material from [pangeos-cost/uq-training](https://github.com/pangeos-cost/uq-training),
also MIT-licensed.
