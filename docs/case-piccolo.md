# Case: Piccolo Doppio (FLMS/QEP)

The [Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb)
notebooks teach the *concepts* of uncertainty propagation for the Piccolo Doppio system using one clean, minimal example. This case study extends this case using the same approach to the full 2025 Norway field campaign, on 12 wheat plots measured around solar noon under sub optimal, variable cloud illumination, applying the same ideas at full scale: two detectors (FLMS and QEP, from Ocean Insight, USA), a calibration chain, and five vegetation indices, all with random and systematic uncertainty tracked separately and propagated with `punpy` (Comet toolkit).

This page focuses on what each stage produces and how uncertainty flows to the next. The complete implementation, including the calibration chain itself, is available as a runnable notebook below, for readers who want to reproduce every step on their own data.

!!! NOTE: The Piccolo Doppio system is one example of a broader instrument class: a **dual fibre spectrometer built around Ocean Insight QE series spectrometers**, with a cosine diffuser fore optic for irradiance and a
collimator (or bare fibre) for radiance. The same measurement equation, calibration chain structure, and uncertainty propagation approach can be applied directly to other systems built the same way,  such as **FLOX box** and similar custom dual optic QE setups. 
The 2025 Norway data is used throughout as a concrete worked example so the process is easy to follow; every step is written to be re run on your own data by substituting your own files in the same format.

## Calibration chain

A spectrometer's raw output is not a physical quantity, it is a digital count/number (DN) that depends on the specific detector, its exposure time, its temperature, and how it has drifted since it was last checked against a known reference (a standard). Before anything scientifically meaningful (radiance, irradiance, reflectance) can be computed, the DNs has to be traced back to a physical unit through an unbroken chain of comparisons against reference standards, this is what *traceability* means.

In this pipeline, that traceability chain has three links, each one re linking the calibration closer to the actual conditions the measurement was performed:

1. **Laboratory calibration** — the detector's raw response is related to a radiance/irradiance standard under controlled laboratory conditions.
2. **Laboratory validation against a transfer standard (ValGEOS)** — the lab. calibration is checked against an independent, NIST traceable transfer standard, confirming it is trustworthy before it leaves the lab.
3. **Field calibration** — the validated calibration is relinked using a field validation measurement, to account for whatever has changed (temperature, transport, time elapsed) between the lab and the actual field campaign. 

Skipping any of these steps would mean trusting that nothing changed between one context and the next, exactly the kind of unquantified assumption this whole training module exists to avoid.

## Outputs of traceability chain

Each of the three calibration steps outputs, per detector (FLMS and QEP) and per wavelength:
- a **calibrated value** (a calibration coefficient, or a validated radiance/irradiance), and
- its **uncertainty, already split into a random component and a systematic component**, not a single combined number. The uncertainties from previous step is propagated to the next one, so at the end we obtain the results we need with total propagated uncertainty, for the entire chain.

That split matters downstream: a random component shrinks when you average repeated measurements; a systematic (common) one does not. By the time the field calibration step hands off its result, every
downstream calculation treats it as: *a value, a random uncertainty, and a systematic uncertainty* — three numbers (per wavelength) in, three numbers out, at every subsequent step.

How the uncertainty is propagated along the full pipeline can be seen in 
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
