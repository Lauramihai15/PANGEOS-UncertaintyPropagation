# Cross-sensor intercomparison and agreement case

The three cases in this section ([Piccolo Doppio](case-piccolo.md),
[ASD & SVC](case-asd-svc.md), [UAV sensors](case-uav.md)) each derive
reflectance and vegetation indices their own way, under the same real
constraint: the 2025 Norway field campaign was carried out under sub-optimal,
variable cloud illumination, not the stable clear-sky conditions that every
calibration method quietly assumes. This case is about the final question all
three lead to: **given that every instrument's uncertainty budget is properly evaluated,
can their measurements still be meaningfully compared?**

The diagram below shows how uncertainty flows from calibration to indices for
all five instruments, converging on that question.

<img src="images/sensor-fleet-full-traceability.png" alt="Complete traceability and uncertainty propagation diagram for the five optical sensors" width="100%"/> 

> **Note:** Findings referenced throughout are from Werfeli, M., Antala, M., Abdelmajeed, A.Y.A., Sánchez-Virosta, Á., Halem, Z., Merrington, A., El-Mejjaouy, Y., Petrovic, B., Hueni, A., Bouras, E.H., Kefauver, S.C., Shafiee, S., Rastogi, A., Mihai, L. (2026). *Uncertainty propagation and intercomparison of multi-sensor measurements of vegetation stress in sub-optimal conditions.* Precision Agric 27, 153. https://doi.org/10.1007/s11119-026-10455-1

> **Note:** This material can be applied to any similar devices.
    The E<sub>N</sub>-ratio comparison method shown here applies to **any
    independent instruments** measuring the same target, not only the
    sensors in this campaign. The 2025 Norway data is a concrete worked
    example; the code is written to run on your own paired measurements and
    their own uncertainty budgets in the same way.

## The comparisons run

Rather than asking a single "do the sensors agree?" question, the real analysis
runs several distinct, targeted comparisons, each isolating one specific
source of disagreement:

| Comparison | Isolates |
|---|---|
| Piccolo FLMS vs. ASD, SVC, QEP | Ground point spectrometers against each other → tests instrument-to-instrument agreement across different measurement methods |
| Piccolo FLMS vs. Altum / REMX | UAV vs. ground point measurement → tests agreement across completely different spatial footprints and calibration methods |

Each comparison is repeated for both white reference panels used in the
ground campaign (labelled WP1 and WP2, or GP and WP2, in this training material), since the
panel itself carries its own calibration uncertainty and is part of what
could cause two instruments to disagree.
[Try it yourself](https://github.com/Lauramihai15/PANGEOS-UncertaintyPropagation/blob/main/notebooks/MeasAgreement_4Sensors.ipynb)
with real Plot 103 data: the notebook covers both the reflectance and the vegetation indices.

## Why does each sensor still need its own uncertainty budget first?

The E<sub>N</sub> ratio (the normalised error, defined below) only needs each
sensor's **final value and its own combined uncertainty**, it does not depend on
how that uncertainty was built up internally, which is exactly why
[Piccolo](case-piccolo.md), [ASD/SVC](case-asd-svc.md), and
[UAV](case-uav.md) are allowed to follow completely different calibration
routes. But it does mean the comparison is only as trustworthy as the
least complete uncertainty budget feeding into it: if one sensor's budget is
missing a real source of error, its E<sub>N</sub> values will look better than
they should, not worse; this is an easy way to be fooled into false agreement.

$$E_N = \frac{|x_1 - x_2|}{k\sqrt{u(x_1)^2 + u(x_2)^2}}$$

- E<sub>N</sub> ≤ 1 (at k=1, ~68% confidence, or k=2, ~95%) → the disagreement
  is fully explained by the two sensors' own uncertainty.
- E<sub>N</sub> > 1 → something is happening that neither sensor's budget
  accounts for.

## What sub-optimal conditions do to the comparison, specifically

Two effects of the variable cloud conditions during this campaign feed
directly into these E<sub>N</sub> comparisons, and both work in the same
direction, inflating disagreement between instruments that would otherwise
agree:

- **Changing cloud cover** adds a shared, time-varying error to every
  ground-based reflectance measurement (quantified via the wavelength-pair
  panel regression described in [ASD & SVC](case-asd-svc.md#measurement-and-uncertainty-propagation-approach)),
  and it does not affect every instrument identically if they were not
  reading at the exact same instant.
- **Acquisition timing** compounds this: instruments were measured sequentially,
  not simultaneously, so a comparison between two instruments is really a
  comparison across a time gap during which illumination could already have
  shifted. This is why the 10-minute acquisition window (remeasuring the
  white panel at least every 10 minutes) matters; beyond it, apparent
  disagreement is dominated by illumination drift, not by anything the
  instruments themselves did differently.

The practical consequence: an E<sub>N</sub> comparison run on data from this
kind of campaign has to treat "were these two readings taken close enough in
time under similar enough sky conditions" as a precondition to check *before*
trusting the E<sub>N</sub> value itself: a large E<sub>N</sub> can mean real
sensor disagreement, or it can mean that the comparison was never valid to begin
with.

## Going further

- [Case: Piccolo Doppio](case-piccolo.md), [Case: ASD & SVC](case-asd-svc.md),
  [Case: UAV sensors](case-uav.md) — the three uncertainty budgets that feed
  into every comparison on this page.
