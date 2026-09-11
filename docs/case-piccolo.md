# Case: Piccolo Doppio (FLMS/QEP)

The [Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb)
notebooks teach the *concepts* of uncertainty propagation for the Piccolo
Doppio system using one clean, minimal example. This case study extends that
same approach to the full 2025 Norway field campaign — 12 wheat plots
measured around solar noon under sub-optimal, variable-cloud illumination —
applying the same ideas at full scale: two detectors (FLMS and QEP), a
calibration chain, and five vegetation indices, all with random *and*
systematic uncertainty tracked separately and propagated with `punpy`.

This page is a **map of that pipeline**, focused on what each stage produces
and how uncertainty flows from it to the next — not on how each stage is
implemented internally. The laboratory and field **calibration methodology**
itself is deliberately treated as a black box here and not described; what
matters for this page is its role as a *supplier of numbers* (calibrated
values and their uncertainty) to everything downstream.

!!! note "Status"
    This describes the pipeline as it exists in the underlying research
    code, not yet published in this repository in adapted form. Findings and
    figures referenced throughout are from Werfeli, Mihai *et al.* (in
    review), *"Uncertainty propagation and intercomparison of multi-sensor
    measurements of vegetation stress in sub-optimal conditions."*

!!! tip "Not just for this dataset"
    The Piccolo Doppio system is one example of a broader instrument class:
    a **dual-fibre spectrometer built around Ocean Insight QE-series
    detectors**, with a cosine-diffuser fore optic for irradiance and a
    collimator (or bare fibre) for radiance. The same measurement equation,
    calibration-chain structure, and uncertainty propagation approach apply
    directly to other systems built the same way — including **FLOX** and
    similar custom dual-optic QE setups. The 2025 Norway data is used
    throughout as a concrete worked example so the process is easy to
    follow; every step is written to be re-run on your own data by
    substituting your own files in the same format.

## How it differs from Case 1

| | Case 1 teaching notebook | Full pipeline |
|---|---|---|
| Plots | 1 example dataset | 12 field plots |
| Detectors | FLMS only | FLMS **and** QEP, combined |
| Calibration | lab calibration only | full lab + field calibration chain (see below) |
| Uncertainty propagation | manual arithmetic, deterministic | `punpy` Monte Carlo, 10,000 samples |
| Random vs. systematic | not separated | tracked as two separate components throughout, combined only at the end |
| Correlation structure | not modelled | explicit correlation matrices between wavelengths/bands feeding into each index |
| Cosine response | not included | applied as an additional systematic component on irradiance, per plot (see below) |
| Outputs | a single reflectance spectrum | reflectance and 5 vegetation indices, all with combined uncertainty, per point and per plot mean |

## Why a calibration chain is needed at all

A spectrometer's raw output is not a physical quantity — it is a digital
count that depends on the specific detector, its exposure time, its
temperature, and how it has drifted since it was last checked against a
known reference. Before anything scientifically meaningful (radiance,
irradiance, reflectance) can be computed, that raw count has to be traced
back to a physical unit through an unbroken chain of comparisons against
reference standards — this is what *traceability* means.

In this pipeline, that traceability chain has three links, each one
re-anchoring the calibration closer to the actual conditions the
measurement was taken in:

1. **Laboratory calibration** — the detector's raw response is related to a
   radiance/irradiance standard under controlled laboratory conditions.
2. **Laboratory validation against a transfer standard (ValGEOS)** — the lab
   calibration is checked against an independent, NIST-traceable transfer
   standard, confirming it is trustworthy before it leaves the lab.
3. **Field calibration** — the validated calibration is re-anchored using a
   field validation measurement, to account for whatever has changed
   (temperature, transport, time elapsed) between the lab and the actual
   field campaign.

Skipping any of these links would mean trusting that nothing changed between
one context and the next — exactly the kind of unquantified assumption this
whole training module exists to avoid.

**The methodology behind each of these three steps is not documented on this
page.** What matters here is what each step hands off to the rest of the
pipeline.

## What the calibration chain produces

Each of the three calibration steps outputs, per detector (FLMS and QEP) and
per wavelength:

- a **calibrated value** (a calibration coefficient, or a validated
  radiance/irradiance), and
- its **uncertainty, already split into a random component and a systematic
  component** — not a single combined number.

That split matters downstream: a random component shrinks when you average
repeated measurements; a systematic (common) one does not.
By the time the field calibration step hands off its result, every
downstream calculation treats it as: *a value, a random uncertainty, and a
systematic uncertainty* — three numbers (per wavelength) in, three numbers
out, at every subsequent step.

## How the uncertainty propagates from there to the final products

From the field calibration coefficients onward, the chain is:

```
field calibration (value, u_random, u_systematic)
        │
        ▼
field measurement → calibrated radiance (L) and irradiance (E)
        │                                    (each with u_random, u_systematic)
        │
        │   + cosine-response uncertainty added to E's systematic component
        │     (see below — this part *is* public, it's in the paper)
        ▼
reflectance R = π·L/E        (u_random, u_systematic propagated through the ratio)
        │
        ├──► plot-level mean over 5 points
        │     (u_random shrinks by averaging + plot heterogeneity added back in;
        │      u_systematic does not shrink — see Eq. 3 in the paper)
        │
        ▼
vegetation indices (NDVI, EVI, OSAVI, MTCI, PRI)
        │     (each index's uncertainty combines the reflectance-band
        │      uncertainties through that index's own formula, using a
        │      correlation matrix sized to how many bands it depends on)
        ▼
ready for cross-sensor comparison — see Case: cross-sensor intercomparison
```

At every arrow, uncertainty is propagated with `punpy`'s Monte Carlo engine
(`propagate_random` / `propagate_systematic`, 10,000 samples) rather than
carried forward as a single hand-derived formula — the same principle taught
in Case 1, just applied consistently through many more steps and to many
more output quantities.

### Cosine response: the one calibration-adjacent detail that is public

One correction is described in detail here because it is already published
in the paper's Results section, not because it is part of the confidential
calibration methodology: the irradiance sensor's cosine receptor deviates
from the ideal cosine law, and this deviation is applied as an *additional*
systematic uncertainty on irradiance at the field-measurement step — sized
per plot according to solar zenith angle (9.94%, 9.66%, or 10.11% relative
uncertainty, matching the paper's reported `u(ccos)` values), treated as a
rectangular (uniform) distribution.

## Field measurements, per point

For each of the 5 measurement points per plot, the field-calibrated radiance
and irradiance are combined into reflectance, with the cosine-response term
folded in as described above. This is the direct, full-scale counterpart of
what Case 1 does for a single example point.

### Plot-level mean over 5 points

The 5 per-point spectra are combined into a plot mean in a way that does two
things at once: it averages the random component down (more points → less
random noise) *and* adds the point-to-point spread back in as an extra
random term — this is what turns "5 repeated measurements" into the
`u(cinhomogeneity)` component from Eq. 3 of the paper. The systematic
component is *not* reduced by averaging, because it affects every point in
the same way — only its mean is carried forward.

!!! tip "Try it yourself"
    [`case_1b-Ex1_PlotMean_Uncertainty.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_1b-Ex1_PlotMean_Uncertainty.ipynb)
    walks through exactly this step with real Plot 103 data (5 points,
    already-calibrated reflectance handed to you as a black box), and checks
    your result against the official pipeline output.

## Vegetation indices

At each point and at the plot-mean level, **NDVI, EVI, OSAVI, MTCI, and
PRI** — all five indices from the paper's Table 3 — are computed with
uncertainty propagated using a correlation matrix sized to the number of
reflectance bands each index depends on (2 bands or 3 bands), reflecting
that those bands share correlated systematic errors from the same
calibration chain.

## Running across the full campaign

The full run loops over all 12 plots, both detectors, and both
cosine-correction settings, saving every intermediate and final product as
CSV, organised per sensor and per plot.

## Where this could go next

Most of this page is documentation only, describing code not yet adapted
into training notebooks. One piece has been adapted so far:

- ✅ **Case 1b** — done, see the "Try it yourself" box above: field
  measurement → reflectance → plot-mean, at real scale, calibration treated
  as a black box.
- A **Case 2b** notebook covering the vegetation-index correlation-matrix
  handling, as a more realistic counterpart to `case_2-Ex1-VI.ipynb`. Not
  started.
