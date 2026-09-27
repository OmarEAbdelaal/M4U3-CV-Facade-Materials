# Error Analysis — Facade Material Detection

This document lists what the model gets wrong and why. The assignment asks
for 3 false positives, 3 false negatives and 3 prioritized data improvements,
taken from the final model (Run 03). Observations from the earlier runs are
recorded below so the final analysis can show whether they were fixed.

Error types used in this document:

| Error | Meaning | Example in this project |
|---|---|---|
| False positive | Detects a material that isn't there | Shadow on painted render called `exposed_concrete` |
| False negative | Misses a material that is there | Brick wall not detected |
| Class confusion | Right region, wrong material | Reflective panel called `glass` |
| Localization error | Right material, badly placed box | `glass` box includes boarded panel below |

In this project a **false negative costs more than a false positive**: a
missed material means a value-engineering substitution could go unflagged
(see `02_class_definitions.md`).

---

## Observed in early runs (Runs 01–02)

Source: `results/evidence/training_runs/`. Details in `project_log.md`, entry 6.

| # | Run | Type | What happened | Likely cause |
|---|---|---|---|---|
| E1 | 01 | False negative | The brick wall covering most of the test image was not detected at all | Too few `brick` examples; large boxes around whole walls are underrepresented in about 50 images |
| E2 | 01 | Localization error | `glass` boxes around the windows also contain the boarded-up panels under each pane | Inconsistent box tightness between hand-drawn boxes and SAM-derived boxes; few glass examples with mixed infill |
| E3 | 02 | False negative (all classes) | No predictions on the test image | Model did not converge: very short training, a different dataset version, possible Grayscale preprocessing |

---

## Final model (Run 03) — to be filled in

Take these from the validation and new-image predictions saved in
`results/evidence/val_predictions/` and `results/evidence/new_predictions/`.
For each one, link the screenshot.

### 3 false positives

| # | Image | What was predicted | What is really there | Why I think it happened |
|---|---|---|---|---|
| FP1 | _TBD_ | _TBD_ | _TBD_ | _TBD_ |
| FP2 | _TBD_ | _TBD_ | _TBD_ | _TBD_ |
| FP3 | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

### 3 false negatives

| # | Image | What was missed | Why I think it happened |
|---|---|---|---|
| FN1 | _TBD_ | _TBD_ | _TBD_ |
| FN2 | _TBD_ | _TBD_ | _TBD_ |
| FN3 | _TBD_ | _TBD_ | _TBD_ |

Check whether E1 (missed brick wall) still happens with the final model and
say so here.

---

## 3 prioritized next data improvements

Ordered by expected impact. Update after Run 03 if the final errors point
somewhere else.

1. **More images for weak classes, starting with `brick`.** Target at least
   40 labelled examples per class, and include large, frame-filling walls,
   since that is exactly the case the baseline missed.
2. **Hard cases on purpose.** Add images with strong shadows, glare on glass,
   night or overcast light, and partial occlusion (trees, scaffolding) so the
   model sees the conditions that cause false positives on site.
3. **One box-drawing standard.** Re-check boxes so hand-drawn and SAM-derived
   boxes are equally tight, and split L-shaped material areas into 2–3
   rectangles instead of one loose box.
