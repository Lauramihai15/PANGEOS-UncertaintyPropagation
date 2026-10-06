# Piccolo Doppio (FLMS/QEP) uncertainty propagation case

The [Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb)
notebooks teach the basics of uncertainty propagation for the Piccolo Doppio system using one clean, minimal example. This case study extends this case, using the same approach, to the full 2025 Norway field campaign: 12 wheat plots measured around solar noon under sub-optimal, variable cloud illumination. The same ideas are applied at full scale: two detectors (FLMS and QEP, from Ocean Insight, USA), a calibration chain, and five vegetation indices, all with random and systematic uncertainty tracked separately and propagated with `punpy` (CoMet toolkit).

This page focuses on what each stage produces and how uncertainty flows to the next. The runnable notebook below starts from the field calibration coefficients, which are the result of the calibration chain, and processes the field measurements of any plot of the campaign up to the vegetation indices, for readers who want to reproduce these steps on their own data.

> **Note:** This example can be applied to any similar device.
> The Piccolo Doppio system is one example of a broader instrument class:
> a **dual-fibre spectrometer built around Ocean Insight QE-series spectrometers**,
> with a cosine-diffuser fore-optic for irradiance and a collimator (or bare fibre) for radiance.
> The same measurement equation, calibration chain structure, and uncertainty propagation approach
> can be applied directly to other systems built in the same way, such as **FLOX** and similar custom dual-optic QE setups.
> The 2025 Norway data is used throughout as a concrete worked example so that the process is easy to follow;
> every step can be re-run on your own data by substituting your own files in the same format.

## Calibration chain

A spectrometer's raw output is not a physical quantity; it is a digital number (DN) that depends on the specific detector, its exposure time, its temperature, and how much it has drifted since it was last checked against a known reference (a standard). Before anything scientifically meaningful (radiance, irradiance, reflectance) can be computed, the DNs have to be traced back to a physical unit through an unbroken chain of comparisons against reference standards. This is what traceability means. Traceability is what allows the uncertainty of the final results (reflectance, vegetation indices) to be quantified, and measurements from different instruments, places and times to be compared on a common basis.

The chain of the Piccolo Doppio follows the traceability diagram below (left side). Each level carries the uncertainty of the previous ones, split into a random (u_r) and a systematic (u_s) component:

- **National standards (SI):** national primary standard of spectral irradiance (250-2500 nm) and national secondary standards of spectral irradiance and of spectral radiance (350-2500 nm).
- **Laboratory reference standards:** spectral irradiance and radiance standards (350-2500 nm) with stated uncertainty (k = 2), traceable to the national standards (for the Piccolo Doppio, to NIST).
- **Laboratory calibration (L1):** the radiance (L) and irradiance (E) sensors of the Piccolo Doppio (400-1000 nm) are calibrated against the laboratory reference standards, using a transfer-standard spectroradiometer (350-2500 nm). The non-linearity of the detectors and the cosine response of the irradiance optics are characterised. The field transfer standard ValGEOS (350-2500 nm) is calibrated in the laboratory as well.
- **Field calibration:** the Piccolo Doppio is calibrated again with ValGEOS, under the conditions of the campaign. This assesses whether the laboratory calibration remains valid after transport and under the environmental conditions of the campaign, and accounts for changes associated with factors such as temperature, elapsed time or instrument handling.
- **Field measurements (L0, raw):** radiance L and irradiance E measured at 5 points of each plot, with their random and systematic uncertainty.
- **Reflectance (L2):** R = &pi; L / E, with u_r(R) and u_s(R).
- **Vegetation indices (L3):** NDVI, EVI, OSAVI, MTCI and PRI, with u_r(VI) and u_s(VI).

The Piccolo Doppio was calibrated in the laboratory and in the field, and the uncertainty of each calibration step is propagated into the final calibration coefficients. The calibration procedure itself is not repeated in this material: users normally receive from the calibrating laboratory only the calibration coefficients and their uncertainties, which are the input of the notebook. The procedure is the same when no field system is available to validate the calibration: the coefficients provided by the laboratory are used together with their uncertainties.

## Outputs of the traceability chain

The calibration chain provides, per detector (FLMS and QEP) and per wavelength:
- a **calibrated value** (the calibration coefficient), and
- its **uncertainty, already split into a random component and a systematic component**, not a single combined number. The uncertainties from each previous step are propagated to the next one, so that at the end we obtain the required results with the total propagated uncertainty for the entire chain.

That split matters downstream: a random component shrinks when you average repeated measurements; a systematic (common) one does not. By the time the calibration coefficients are handed over, every
downstream calculation treats it as: *a value, a random uncertainty, and a systematic uncertainty*, three numbers (per wavelength) in, three numbers out, at every subsequent step.

How the uncertainty is propagated along the full pipeline can be seen in the diagram, on the left side:

<img src="images/sensor-fleet-full-traceability.png" alt="Complete traceability and uncertainty propagation diagram for the five optical sensors" width="100%"/> 

and here:

```
field calibration → CalCoeffsL, CalCoeffsE, uL_random, uL_systematic, uE_random, uE_systematic
        │
        ▼
field measurement → calibrated radiance (L) and irradiance (E), (each with u_random, u_systematic)
        │                                    
        │
        │   + *cosine response uncertainty added to E's systematic component
        │    
        ▼
reflectance R = π·L/E        (u_random, u_systematic propagated through the ratio)
        │
        ├──► *plot level mean over 5 points
        │     (u_random shrinks by averaging + plot heterogeneity added back in;
        │      u_systematic does not shrink, see Eq. 3 in the paper)
        │
        ▼
vegetation indices (NDVI, EVI, OSAVI, MTCI, PRI)
        │     (each index's uncertainty combines the reflectance band
        │      uncertainties through that index's own formula, using a
        │      correlation matrix sized to how many bands it depends on)
        ▼
ready for cross-sensor comparison, see Case: cross-sensor intercomparison
```

At every arrow, uncertainty is propagated with the `punpy` functions from the CoMet toolkit (`propagate_random` / `propagate_systematic`, 10,000 samples) rather than carried forward as a single hand-derived formula. This is the same principle taught in Case 1, applied consistently through many more steps and to many more output quantities.

## Cosine correction

The irradiance sensor's cosine receptor deviates from the ideal cosine law. This deviation is applied as an *additional* systematic uncertainty on irradiance at the field-measurement step, sized per plot according to the solar zenith angle (9.94%, 9.66%, or 10.11% relative uncertainty, matching the paper's reported `u(ccos)` values), treated as a rectangular (uniform) distribution.

## Field measurements, per point
For each of the 5 measurement points per plot, the field calibrated radiance and irradiance are combined into reflectance, with the cosine response term folded in as described above. This is the direct, full scale counterpart of what Case 1 does for a single example point.

### Plot level mean over 5 points
The 5 per point spectra are combined into a plot mean in a way that does two things at once: it averages the random component down (more points → less random noise) and adds the point-to-point spread back in as an extra random term. This is what turns "5 repeated measurements" into the `u(c_inhomogeneity)` component from Eq. 3 of the paper. The systematic component is *not* reduced by averaging, because it affects every point in
the same way; only its mean is carried forward.

TRY IT YOURSELF:

[`CaseEx1_Piccolo_UncProp.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/CaseEx1_Piccolo_UncProp.ipynb)
starts from the field calibration coefficients (with their propagated random and systematic uncertainties), applies them to the raw field measurements to obtain the radiance and irradiance, and continues with the reflectance, the plot mean and the five vegetation indices, using real data (Plot 103 by default).
The detector (FLMS or QEP) and the plot are selected at the top of the notebook.

The notebook is organised in the following steps:

1. Import the Python packages and set the parameters (detector, plot, folders)
2. Measurement functions
3. Load and plot the calibration coefficients
4. Field measurements, per point: conversion from DN to radiance and irradiance (SI units), and reflectance
5. Plot-level mean over the points
6. Vegetation indices, per point

## Vegetation indices
At each point, **NDVI, EVI, OSAVI, MTCI, and PRI**, all five indices from the paper's Table 3, are computed with uncertainty propagated using a correlation matrix sized to the number of
reflectance bands each index depends on (2 bands or 3 bands), reflecting that those bands share correlated systematic errors from the same calibration chain.
The QEP detector covers only about 640-805 nm, so for it only the indices whose bands lie within this range (NDVI, OSAVI and MTCI) are calculated; the FLMS detector gives all five.

## Running for other plots and detectors
The notebook processes one plot and one detector at a time. The plot (`PLOT_ID`) and the detector (`SENSOR`, FLMS or QEP) are selected at the top of the notebook; the same code runs for all 12 plots and for both detectors.
