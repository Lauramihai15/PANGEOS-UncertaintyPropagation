# Case: Uncertainty propagation for ASD FieldSpec 4 & SVC HR-1024i

The previous training material, [Case 2](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_2-Ex1-VI.ipynb), introduces the basic principles of uncertainty propagation for reflectance factor measurements, using data from an ASD spectroradiometer as an example.

The present material extends **Case 2** and provides a step by step workflow for propagating uncertainty through the complete measurement and processing chain for two field spectroradiometers: the **ASD FieldSpec 4** and the **SVC HR-1024i**.

Two white reference panels were used in the field:

* **WP2**, with a NIST-traceable calibration;
* **WP1**, without a traceable calibration certificate.

The measurements were acquired during the same Norway 2025 field campaign as the Piccolo Doppio and UAV Altum and REMX measurements, over the same plots, during approximately the same period of the day, and under the same **sub-optimal and variable illumination conditions**.

The ASD and SVC systems were operated using the same measurement protocol, with the two instruments measuring the same targets and reference panels sequentially. Consequently, the two systems follow a similar uncertainty propagation methodology, while retaining their own instrument specific results: reflectance values, vegetation indices, and their corresponding propagated uncertainties.

For these two systems, the internal calibration and metrological traceability of the spectroradiometers are treated as a **black box**, following the approach described by *Werfeli, Mihai et al. (2026)*. The uncertainty analysis therefore focuses on the quantities and sources of variability that can be evaluated from the field measurement procedure and the available calibration information.

The complete methodology and the performance of the two systems are described in:

*Werfeli, M., Mihai, L., et al. (2026). “Uncertainty propagation and intercomparison of multi-sensor measurements of vegetation stress in sub-optimal conditions.”*

> **Note:** The methodology presented in this case, together with the associated code, is based on a **white-reference-panel ratio approach** for reflectance retrieval. It can therefore be adapted to other optical systems that derive reflectance from the ratio between measurements of a target and a reference panel. The workflow is not limited to the **ASD FieldSpec 4** and **SVC HR-1024i** instruments used during the Norway 2025 campaign.

## Methodology used: Target Plot 1 / White Panel WP1 / White Panel WP2 / … / Target Plot n

Compared with the **Piccolo Doppio example (Case 1)**, where a complete traceability chain is considered, the ASD and SVC workflow uses the following relationship to derive reflectance:

$$
R = \frac{DN_T}{DN_R} \cdot \rho_R \cdot c_{\mathrm{clouds}}
$$

where:

* \(DN_T\) is the raw signal measured over the **target**, corresponding to the area of the plot within the spectroradiometer field of view;
* \(DN_R\) is the raw signal measured over the **white reference panel** (WP1 or WP2);
* \(\rho_R\) is the reflectance scaling factor of the reference panel, obtained from its calibration information;
* \(c_{\mathrm{clouds}}\) represents the correction applied to account for changes in illumination caused by variable cloud conditions.

Unlike the Piccolo Doppio configuration, where radiance and irradiance are measured simultaneously through two optical channels, ASD and SVC measurements of the target and reference panel are acquired sequentially. Consequently, changes in incoming illumination between consecutive target and reference measurements can directly influence the retrieved reflectance. The cloud related correction and its associated uncertainty therefore become important components of the uncertainty budget under sub-optimal field conditions.


**What this method needs, that a lab-calibrated instrument does not:** the
panel reading has to be *interpolated in time* to match the target reading,
since the two are never taken at the exact same instant. That interpolation
step is itself propagated with uncertainty (via `punpy`'s Monte Carlo
propagation) — treating the panel's timing as free/exact would understate the
final reflectance uncertainty.

## The cloud problem, and how it was measured

This is the sub-optimal-conditions problem in its most direct form for this
instrument type. If the sky is stable, a white panel reading taken a few
minutes before the target is an excellent stand-in for the panel reading you
*would* have taken at the exact target time. If clouds are moving, that
assumption breaks down — and it breaks down differently at different
wavelengths, because different parts of the spectrum are affected differently
by changing diffuse/direct irradiance.

The campaign's answer: **estimate the cloud-driven uncertainty directly from
the data**, using sequential white-panel pairs. For each pair of panel
readings bracketing a plot measurement, the two readings are regressed
against each other across three wavelength-pair combinations (covering VNIR,
SWIR1, SWIR2), and the residuals from that regression become the cloud
uncertainty term, `u(clouds)`, at every wavelength — the same idea shown for
one plot pair in the paper's cloud-regression figure, applied here to
determine an actual number rather than assumed to be zero.

This is treated as a **systematic** uncertainty component, alongside the
panel's own reflectance uncertainty and the instrument's plot-level
inhomogeneity — combined via covariance matrices (not simply added) so that
correlation between components isn't lost, then converted back into a single
combined uncertainty and correlation structure for the reflectance spectrum.

!!! tip "Try it yourself"
    [`case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb)
    builds a simplified version of this budget from real ASD FieldSpec 4
    field repeats (Plot 103): repeatability across positions, a real panel
    calibration certificate, and a cloud-uncertainty proxy estimated from
    real white-reference self-checks.

## What this produces, and how it feeds forward

For each measurement, this case produces — per wavelength — a reflectance
value with its random uncertainty, systematic uncertainty (cloud + panel +
plot inhomogeneity, combined), and the correlation structure between
wavelengths. From there, the same pattern used throughout this training
module applies:

```
reflectance (value, u_random, u_systematic, correlation structure)
        │
        ▼
vegetation indices: NDVI, EVI, OSAVI, MTCI, PRI
        │   (propagated with punpy, using a 2- or 3-band correlation
        │    matrix built from the reflectance's own correlation structure
        │    at exactly the bands each index needs)
        ▼
ready for cross-sensor comparison — see Case: cross-sensor intercomparison
```

The vegetation indices are computed with the *same five formulas* used for
Piccolo Doppio (Case 1/1b) — NDVI, EVI, OSAVI, MTCI, PRI — which is precisely
what makes a later cross-instrument comparison of index values meaningful:
the formulas are identical, only the reflectance inputs and their
uncertainty differ.

TRY IT YOURSELF:
    [`case_asdsvc-Ex2_VegetationIndices.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex2_VegetationIndices.ipynb)
    computes all five indices from the real Plot 103 spectrum with `punpy`,
    and checks the result against the official campaign values — all five
    index **values** match to floating-point precision; their propagated
    **uncertainty** matches only approximately (as with the reflectance
    budget above), which is itself informative about how uncertainty
    propagates differently through different index formulas.

## Going further

- [Case: Piccolo Doppio](case-piccolo.md) — the lab-calibrated alternative to
  this field-panel method, for the same campaign.
- [Case: cross-sensor intercomparison](case-intercomparison.md) — how ASD and
  SVC results (and everyone else's) get tested against each other.
- **Field practice note:** this method depends on the operational
  10-minute rule — re-measure the white panel at least every 10 minutes,
  since panel-to-target timing directly drives the cloud-uncertainty term
  above. See [Case: cross-sensor intercomparison](case-intercomparison.md)
  for what happens in practice when that window is exceeded.
