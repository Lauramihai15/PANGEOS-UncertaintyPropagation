# ASD FieldSpec 4 & SVC HR-1024i uncertainty propagation case

The previous training material, [Case 2](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_2-Ex1-VI.ipynb), presented a basic case of uncertainty propagation for reflectance factor measurements, using ASD data as an example.

Here, is presented the extended version of **Case 2** going through the full uncertainty propagation workflow for two field spectroradiometers: the **ASD FieldSpec 4** and the **SVC HR-1024i**.

For this purpose, two white reference panels were used during the field measurements:

* **WP2**, with a NIST traceable calibration (calibration certificate with corresponding uncertainties at k=2);
* **WP1**, without a traceable calibration certificate.

The measurements were collected during the Norway 2025 field campaign, over the same plots and under the similar sub optimal and variable illumination conditions as the Piccolo Doppio and UAV measurements.

ASD and SVC followed the same field protocol, measuring the same targets and reference panels one after another. The uncertainty propagation approach is therefore similar for the two instruments, while the resulting reflectance values, vegetation indices, and propagated uncertainties are specific for each system.

For both devices, the internal calibration and metrological traceability are treated as a **black box**, following *Werfeli, Mihai et al. (2026)*. The uncertainty budget therefore focuses on the quantities that can be evaluated from the field measurements and from the available information on the reference panels.

> **Note:** The same approach can be used for other optical systems that calculate reflectance from the ratio between a target measurement and a white reference measurement. It is not specific to ASD FieldSpec 4 or SVC HR-1024i.

## Measurement and uncertainty propagation approach

Reflectance is calculated as:

$$R = \frac{DN_T}{DN_R} \cdot \rho_R \cdot c_{\mathrm{clouds}}$$

where:

* $$\(DN_T\)$$ is the raw signal measured over the target;
* $$\(DN_R\)$$ is the raw signal measured over the white reference panel;
* $$\(\rho_R\)$$ is the reflectance scaling factor of the reference panel;
* $$\(c_{\mathrm{clouds}}\)$$ accounts for changes in illumination caused by variable cloud conditions.

The main difference compared with the Piccolo Doppio case is that ASD and SVC do not measure target and reference simultaneously. The measurements are taken one after another, so the illumination can change between the two acquisitions.

To reduce this effect, the white panel signal is interpolated in time to the target acquisition time. The uncertainty introduced by this interpolation is also propagated using the Monte Carlo approach implemented in `punpy`.

The effect of changing cloud conditions is estimated directly from sequential white panel measurements. Panel measurements around the target acquisition are compared over wavelength ranges representative of VNIR, SWIR1, and SWIR2, and the regression residuals are used to estimate the wavelength dependent cloud uncertainty term, \(u(\mathrm{clouds})\).

This contribution is combined with the uncertainty of the reference panel and the plot inhomogeneity using covariance matrices, so that correlations between uncertainty components are retained.

## Best practice tips for field campaign under sub - optimal sky conditions, when using these types of instruments:

* keep the time between white panel and target measurements as short as possible;
* measure the white panel before and after the target whenever possible;
* record accurate timestamps;
* keep the measurement geometry consistent;
* avoid shading the target or the panel;
* keep the white panel clean and in good condition;
* use a traceably calibrated panel whenever possible;
* repeat measurements when illumination changes quickly;
* keep the raw data and all relevant metadata.

During the Norway 2025 campaign, the white panel was remeasured before/after each plot (aprox. at **10 minutes**).

TRY IT YOURSELF:

[`case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb)
builds a simplified uncertainty budget using real ASD FieldSpec 4 measurements from Plot 103.

## From reflectance to vegetation indices

For each wavelength, the workflow gives a reflectance value together with its random uncertainty, systematic uncertainty, and correlation structure.

These are then propagated to the same five vegetation indices used in the Piccolo Doppio case:

**NDVI, EVI, OSAVI, MTCI, and PRI.**

The uncertainty of each index is propagated with `punpy`, using the correlation information from the reflectance values at the wavelengths required by each index.

Using the same vegetation-index formulas for all instruments makes the later cross-sensor comparison more meaningful, because the differences come from the measured reflectance and its uncertainty, not from different index definitions.

TRY IT YOURSELF:

[`case_asdsvc-Ex2_VegetationIndices.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex2_VegetationIndices.ipynb)
calculates all five vegetation indices from the real Plot 103 reflectance spectrum and propagates their uncertainties using `punpy`.

## Going further

* [Case: Piccolo Doppio](case-piccolo.md) — simultaneous radiance and irradiance measurements with a full traceability chain.
* [Case: cross-sensor intercomparison](case-intercomparison.md) — comparison of ASD, SVC, Piccolo Doppio, and the other optical systems using their associated uncertainties.
