# Case: ASD FieldSpec 4 & SVC HR-1024i

This case extends the reflectance-uncertainty approach introduced in
[Case 2](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_2-Ex1-VI.ipynb)
to the two white-reference-panel ground spectroradiometers used in the 2025
Norway field campaign: the **ASD FieldSpec 4** and the **SVC HR-1024i**. Both
instruments follow the *same calibration method* — this is what makes them a
natural pair to study together — but remain two independent instruments with
their own uncertainty budgets, measured under the same sub-optimal,
variable-cloud illumination as every other case in this section.

!!! note "Status"
    This describes the method as it exists in the underlying research code,
    not yet published in this repository in adapted form. As with
    [Piccolo](case-piccolo.md#why-a-calibration-chain-is-needed-at-all), the
    calibration methodology itself is treated as a black box here. Findings
    referenced throughout are from Werfeli, Mihai *et al.* (in review),
    *"Uncertainty propagation and intercomparison of multi-sensor
    measurements of vegetation stress in sub-optimal conditions."*

!!! tip "Not just for this dataset"
    The white-panel-ratio method shown here applies to **any field
    spectroradiometer that derives reflectance from a target/reference-panel
    ratio** — not only the ASD FieldSpec 4 and SVC HR-1024i used in this
    campaign. The 2025 Norway data is a concrete worked example; the code is
    written to run on your own repeated field measurements and your own
    panel calibration certificate in the same way.

## The method: white-panel ratio, not a lab calibration chain

Unlike Piccolo Doppio, which is anchored through a lab calibration chain,
ASD and SVC follow the field convention described in the campaign's
reflectance equation: the target measurement is divided by a white reference
panel (WP) measurement taken immediately before/after it, scaled by the
panel's own known reflectance factor:

$$R = \frac{DN_T}{DN_R} \cdot \rho_R \cdot c_\text{clouds}$$

Two different white panels were used across the campaign (referred to here
as WP1 and WP2), each independently characterised, each with its own
reflectance-uncertainty file. Which panel was used is tracked throughout as
part of the uncertainty budget, not assumed away.

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

!!! tip "Try it yourself"
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
