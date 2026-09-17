# Case study: real field data under sub-optimal conditions

The [training course prepared in 2024-2025](https://github.com/pangeos-cost/uq-training#contents) introduces the propagation of measurement uncertainty through a relatively simple workflow, starting from raw radiance and irradiance measurements and progressing to absolute reflectance values using a single type of spectroradiometer, such as the Piccolo Doppio.
Building on that foundation, the present training material extends the uncertainty propagation workflow to vegetation indices and considers five different types of optical measurement systems operating in parallel. In this case: Piccolo Doppio (FLMS/QEP), ASD, SVC spectroradiometers and REMX and Altum camera. 
All five sensors were used to measure the same plots, making it possible to compare both, their results and their associated uncertainties under the same field conditions. 
The measurements were not acquired under ideal and stable clear-sky conditions. Instead, they were collected during a PANGEOS field campaign in Norway in June 2025 under sub-optimal and variable illumination conditions, with changing cloud cover affecting the incoming radiation. These fluctuations introduce additional complexity into the measurements and, consequently, into the propagation of their uncertainties.

This training example therefore demonstrates how uncertainty can be propagated from the original optical measurements (raw data), through reflectance retrieval, and ultimately to derived vegetation indices when multiple sensor systems are used under realistic and variable field conditions.

The main question addressed in this training material is:

**How can we propagate measurement uncertainty for different types of optical sensors so that we can assess the reliability of the results and meaningfully compare measurements from different sensors, even when field conditions are not ideal?**


This course material is based on work carried out by PANGEOS Working Groups 1, 2, and 4 during the PANGEOS “Joint WG1–WG2 Field Day: Remote Sensing & Proximal Phenotyping”, held in Norway on June 19–20, 2025. The methodology, measurements, and examples presented here build on the activities carried out during this field campaign and are also described in a recent PANGEOS publication:

"Werfeli, M., Antala, M., Abdelmajeed, A.Y.A., Sánchez-Virosta,Á., Halem, Z., Merrington, A., El-Mejjaouy, Y., Petrovic, B., Hueni, A.,Bouras, E.H., Kefauver, S.C., Shafiee, S., Rastogi, A., Mihai, L.* *Uncertainty propagation and intercomparison of multi-sensor
measurements of vegetation stress in sub-optimal conditions.*"

Note: This training material is designed to be reused with your own data.
Every notebook in this section uses real data from the 2025 Norway campaign so that the workflow is concrete and easy to follow. However, the code has been written as a general-purpose template and can be adapted to your own measurements, provided that your input data follow the required format.
This applies especially to the Piccolo Doppio case. The same uncertainty-propagation approach can be used for other dual-fibre spectrometer systems based on Ocean Insight QE-series detectors, using, for example, a cosine diffuser for irradiance measurements and a collimator or bare fibre for radiance measurements. This includes FLOX and similar custom dual-optic systems, not only the specific Piccolo Doppio configuration used in this campaign.

<img src="images/sensor-fleet-full-traceability.png" alt="Diagramă completă: trasabilitate și propagare a incertitudinii pentru cele 5 sisteme optice" width="100%"/>

## Case study examples:

1. **[Piccolo Doppio (FLMS/QEP)](case-piccolo.md)** - extended example of 
   [Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb)
   This example demonstrates how vegetation indices and their corresponding propagated uncertainties can be derived from traceable spectroradiometric measurements, where the instrument calibration is linked to recognized reference standards such as those maintained by NIST, PTB, NPL, or another National Metrology Institute (NMI).
The workflow is applied to measurements collected under sub - optimal illumination conditions across multiple consecutive plots (12 plots), showing how uncertainty can be propagated from the original measurements through reflectance and vegetation index calculations.

TRY IT YOURSELF:
You can also apply the same workflow to your own data using the notebook 
—> [case_1b-Ex1_PlotMean_Uncertainty.ipynb](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_1b-Ex1_PlotMean_Uncertainty.ipynb). The provided example uses real field measurements from plot 103 of the Norway 2025 campaign.

2. **[ASD FieldSpec 4 & SVC HR-1024i](case-asd-svc.md)** — extended example of [Case 2](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_2-Ex1-VI.ipynb).
   This example demonstrates how uncertainty is propagated from reflectance measurements to vegetation indices for two types of field spectroradiometers (ASD FieldSpec 4 & SVC HR-1024i) that use the white reference panel ratio method.

In contrast to the Piccolo Doppio case, where radiance and irradiance are measured simultaneously through two optical channels, thereby reducing the effect of short term changes in illumination, ASD and SVC measurements are acquired sequentially. As a result, variations in the incoming radiation between the white reference and target measurements can have a stronger influence on the retrieved reflectance.
This difference becomes particularly important under variable cloud conditions. Rather than assuming that illumination remains stable, the workflow estimates the associated uncertainty directly from repeated field measurements. This makes it possible to quantify the contribution of short-term illumination variability to the overall uncertainty budget and to assess how strongly this contribution depends on the measurement configuration.

  TRY IT YOURSELF:
—> [`case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex1_FieldUncertaintyBudget.ipynb) - builds the field uncertainty budget using repeated measurements from Plot 103.
—> [`case_asdsvc-Ex2_VegetationIndices.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_asdsvc-Ex2_VegetationIndices.ipynb) -propagates uncertainty to all five vegetation indices and compares the results with the field campaign values.

3. **[UAV sensors (Altum & REMX)](case-uav.md)**
   This example introduces a different measurement approach. Unlike the point based spectroradiometer measurements, the UAV sensors derive reflectance from orthomosaic imagery.
Two independent reflectance calibration methods are compared, allowing to investigate how calibration choices influence the resulting reflectance values and their associated uncertainties. The example also illustrates the importance of flight timing and changing illumination conditions, which can become major contributors to the uncertainty budget when measurements are acquired under variable cloud cover.

  TRY IT YOURSELF:
—> [`case_uav-Ex1_AltumREMX_MethodComparison.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_uav-Ex1_AltumREMX_MethodComparison.ipynb) - compares the two calibration approaches and shows differences of approximately 15 - 28% in several spectral bands.
—> [`case_uav-Ex2_VegetationIndices.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_uav-Ex2_VegetationIndices.ipynb) - derives vegetation indices from the calibrated UAV data. The Altum results reproduce the Norway field campaign values, while the REMX results show a small but real discrepancy that is retained and discussed rather than artificially corrected.
  
4. **[Cross-sensor intercomparison](case-intercomparison.md)** —
This final example brings the previous measurement approaches together and asks a broader question: when measurements come from different sensors, how can we determine whether their results agree within their respective uncertainties?

The case study shows which sensor comparisons were performed and how variable cloud conditions affect cross - sensor agreement. 
It also highlights an important principle: **a meaningful intercomparison is only possible when the uncertainty budget of each individual measurement system has first been evaluated properly.**

  TRY IT YOURSELF:
—> [`case_intercomparison-Ex1_ASD_vs_SVC.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_intercomparison-Ex1_ASD_vs_SVC.ipynb) — performs a real ASD versus SVC agreement test for Plot 103 and includes the actual acquisition timestamps, showing that the nominal 10 minute comparison window was exceeded in practice.


## START WITH THIS IF PASSED THROUGH BASIC CASES 1 and 2

If you already read and tested the basic examples: [Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb) and/or [Case 2](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_2-Ex1-VI.ipynb), you can go further with examples from this page. It follows the same GUM/Monte Carlo principles, uses same `punpy` propagation, but, as compared with the basic examples uses real field campaign data, at crop scale, using multiple instrument types, under non “ideal” sky conditions. 

If you haven't yet tested the basic example, you can test it here:
[Case 1](https://github.com/pangeos-cost/uq-training/blob/main/notebooks/case_1-Ex0-Piccolo.ipynb).

**Authors:** Laura Mihai. See [Contributors](contributors.md) for details on the contributors who developed the underlying research code on which each case study is based.
