# UAV sensors (Altum & REMX) uncertainty propagation case 

The two ground-based cases in this section ([Piccolo](case-piccolo.md),
[ASD/SVC](case-asd-svc.md)) measure one point at a time. This UAV sensors case covers the
2025 Norway campaign's two UAV-mounted multispectral cameras — **Altum** (5
bands + thermal) and **REMX Dual** (10 bands) — which instead produce a full
orthomosaic map of the field in each flight, and derive plot-level
reflectance from a region of interest (ROI) within that map rather than a
point measurement.

> **Note:** This example can be applied to any UAV multispectral camera
    The grey panel (GP) versus white panel(WP2) comparison and the panel/plot/ROI
    uncertainty model shown here apply to **any UAV multispectral camera**
    processed through a similar orthomosaic + ROI extraction workflow, not
    only Altum and REMX. The 2025 Norway data is a concrete worked example;
    the code is written to run on your own per plot reflectance and
    uncertainty exports in the same way.

## Why this case looks different from the other two

Both flights in this campaign happened early, under the most stable,
cloud free part of the day, since UAV missions cannot be re-flown on demand
the way a ground reading can be repeated. This is why the
UAV uncertainty budget in this campaign ends up smaller than the ground
instruments': not because the sensors are inherently more precise, but
because they were flown at the best possible moment. That is itself a
finding worth taking seriously, when you fly matters as much as what you
fly with: plan UAV missions for the calmest, most cloud-free window
available, since that timing decision alone can dominate the whole
uncertainty budget.

## The method: orthomosaic + panel calibration, compared two ways

Each flight (30 m above ground level, 80% front/side overlap) is processed
into an orthomosaic reflectance map through the standard photogrammetry
pipeline: keypoint extraction and bundle adjustment, georeferencing against
ground control points, and generation of the final reflectance orthomosaic.

Radiometric calibration: converting the camera's raw digital numbers to
reflectance, is done **two different ways for the same imagery**, and both
are kept:

1. **Manufacturer's grey panel (GP) workflow** (the standard MicaSense method).
2. **A manual white panel (WP2) correction**, using empirically derived
   correction factors from manually delineated panel areas in the imagery.

Running both isn't redundant, the difference between the two is itself
treated as a **method dependent uncertainty component**, on top of the
within ROI spatial variability. This mirrors the same instinct behind
comparing ASD and SVC in the other case: when two independent ways of doing
the same thing agree, that is evidence the number is trustworthy; when they
disagree, the size of that disagreement becomes part of the honest
uncertainty budget rather than being hidden by picking one method silently.

## Combined uncertainty components

For each plot and spectral band, three components are combined in simple
quadrature:

$$u_c(\rho) = \sqrt{u(\text{panel})^2 + u(\text{plot})^2 + u(\text{ROI})^2}$$

- **Panel calibration** (`u_panel`): zero for the grey panel workflow (no
  white reference correction involved), non zero for the manual white panel
  method.
- **Within plot variability** (`u_plot`): spread of reflectance within the
  plot's ROI.
- **ROI placement sensitivity** (`u_roi`): mean reflectance is extracted from
  the original ROI and from a set of deliberately perturbed ROIs (shifted in
  ±x/±y, buffer varied), the spread across these perturbed extractions
  captures sensitivity to exactly where the ROI boundary was drawn, and is
  the dominant component in practice.

This is a deliberately different (and simpler) uncertainty model than the
ground spectroradiometers: it does not track a laboratory calibration chain
or a per scan random noise term, because at UAV altitude and flight speed
those are not the dominant error sources, ROI placement and calibration
method choice are.

TRY IT YOURSELF:
    [`case_uav-Ex1_AltumREMX_MethodComparison.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_uav-Ex1_AltumREMX_MethodComparison.ipynb)
    uses real Plot 103 results to compare the grey panel and white panel methods directly and 
    reconstructs the official combined uncertainty from its three components exactly.

## Vegetation indices

The same index formulas are applied as elsewhere (NDVI, OSAVI always;
EVI where a blue band is available), but **not every index is available on
every camera**, this is a hardware constraint of the multispectral filters
each camera carries:

- **Altum** (no 580nm band): NDVI, OSAVI, EVI (no PRI).
- **REMX** (no red-edge band at ~709/754nm): NDVI, OSAVI, EVI, PRI (no MTCI).

Where an index needs a wavelength the camera does not have, the nearest
available band is used instead and the resulting spectral mismatch is
treated as an additional, acknowledged source of uncertainty for that
index, not silently absorbed into the result.

TRY IT YOURSELF:
    [`case_uav-Ex2_VegetationIndices.ipynb`](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/case_uav-Ex2_VegetationIndices.ipynb)
    computes NDVI/OSAVI/EVI (and PRI for REMX) from the real Plot 103 reflectance.

## Going further

- [Case: cross-sensor intercomparison](case-intercomparison.md): how
  Altum/REMX plot level reflectance is compared against the ground
  instruments.
- Flight timing was the single biggest lever on this case's uncertainty
  budget: see "Why this case looks different from the other two" above.
