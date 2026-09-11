# Case studies: real field data, sub-optimal conditions

The [training course](https://github.com/pangeos-cost/uq-training#contents) teaches uncertainty propagation
using one clean example at a time. These case studies show the same ideas
applied to a real, messy situation: the 2025 Norway field campaign, where
five different optical sensors measured the same wheat plots under
**sub-optimal, variable-cloud illumination** — not the stable clear-sky
conditions every textbook example quietly assumes.

The question these four cases build up to, together, is a practical one:

> **When conditions are not ideal, how do you propagate uncertainty for each
> instrument honestly enough that you can still meaningfully compare
> different optical sensors used in the same campaign?**

**Reference:** Werfeli, M., Antala, M., Abdelmajeed, A.Y.A., Sánchez-Virosta,
Á., Halem, Z., Merrington, A., El-Mejjaouy, Y., Petrovic, B., Hueni, A.,
Bouras, E.H., Kefauver, S.C., Shafiee, S., Rastogi, A., Mihai, L. (in
review). *Uncertainty propagation and intercomparison of multi-sensor
measurements of vegetation stress in sub-optimal conditions.*

!!! tip "Worked examples, written to be reused on your own data"
    Every notebook in this section uses real 2025 Norway campaign data so
    the process is concrete and easy to follow — but the code itself is
    written as a general-purpose template. Point it at your own files in the
    same format and it runs on your own measurements. This applies most
    directly to the [Piccolo Doppio case](case-piccolo.md): the same
    approach works for any dual-fibre spectrometer built around Ocean
    Insight QE-series detectors (cosine diffuser for irradiance, collimator
    or bare fibre for radiance) — including **FLOX** and similar custom
    dual-optic setups, not just this specific instrument.

<img src="images/sensor-fleet-uncertainty-pipeline.svg" alt="Diagram: uncertainty propagation from calibration to vegetation indices, for five optical sensors, converging into a cross-sensor intercomparison step using the E_N ratio" width="100%"/>

## The four cases

**Authors:** Laura Mihai. See [Contributors](contributors.md) for who
wrote the underlying research code each case is based on.

1. **[Piccolo Doppio (FLMS/QEP)](case-piccolo.md)** — extends
   [Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb)'s
   lab-calibrated dual-detector spectrometer to the full 12-plot campaign,
   including all five vegetation indices.
   Try it yourself: [`case_1b-Ex1_PlotMean_Uncertainty.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_1b-Ex1_PlotMean_Uncertainty.ipynb),
   with real Plot 103 data.
2. **[ASD FieldSpec 4 & SVC HR-1024i](case-asd-svc.md)** — extends
   [Case 2](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_2-Ex1-VI.ipynb)'s
   reflectance-to-index propagation to the white-panel-ratio method shared by
   these two instruments, including how cloud-driven uncertainty is measured
   directly from the data rather than assumed away.
   Try it yourself: [`case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb)
   (real Plot 103 repeats) and
   [`case_asdsvc-Ex2_VegetationIndices.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex2_VegetationIndices.ipynb)
   (all five indices, validated exactly against the official campaign values).
3. **[UAV sensors (Altum & REMX)](case-uav.md)** — orthomosaic-based
   reflectance instead of point measurements, two independent calibration
   methods compared against each other, and why flight timing mattered more
   than anything else in this case's uncertainty budget.
   Try it yourself: [`case_uav-Ex1_AltumREMX_MethodComparison.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_uav-Ex1_AltumREMX_MethodComparison.ipynb)
   (the two calibration methods disagree by 15-28% at most bands) and
   [`case_uav-Ex2_VegetationIndices.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_uav-Ex2_VegetationIndices.ipynb)
   (Altum matches the official indices exactly; REMX is close but not
   exact — a real, reported discrepancy).
4. **[Cross-sensor intercomparison](case-intercomparison.md)** — brings the
   first three together: which comparisons were actually run, what the
   variable-cloud conditions specifically do to a cross-sensor comparison,
   and why an honest uncertainty budget in each case is the precondition for
   any of it being trustworthy.
   Try it yourself: [`case_intercomparison-Ex1_ASD_vs_SVC.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_intercomparison-Ex1_ASD_vs_SVC.ipynb) —
   a real ASD-vs-SVC agreement test at Plot 103, including real acquisition
   timestamps showing the 10-minute window being exceeded in practice.

## Why start here instead of with the course

If you've already worked through [Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb)
or [Case 2](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_2-Ex1-VI.ipynb),
these cases are the natural next step: same GUM/Monte Carlo principles, same
`punpy` propagation, but applied at real campaign scale, across multiple
instrument types, under conditions that don't cooperate. If you haven't yet,
[Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb)
is still the better starting point — these cases assume you already know why
random and systematic uncertainty are tracked separately.
