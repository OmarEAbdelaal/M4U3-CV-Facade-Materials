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
| E3 | 02 | False negative (all classes) | No predictions on the test image | Model did not converge: very short training, a different dataset version, and (found later) corrupted boxes from mixed polygon/box label files |

---

## Label problems found in the released dataset (version 4)

Source: inspection of the released zip and the annotation examples in
[`results/evidence/annotations/`](../results/evidence/annotations/).
Details in `project_log.md`, entry 8. These are errors in the **training
data**, not the model, but they cause model errors.

| # | Problem | Evidence | Effect on the model | Handled |
|---|---|---|---|---|
| A1 | SAM polygons and plain boxes mixed in the same label file (181 files) | Label files; example 2 vs. examples 1 and 3 | The YOLOv8 detection loader misreads the boxes in those files | Yes: converted to boxes in the training notebook |
| A2 | About 3,500 boxes under 4 px, mostly SAM fragments | Label files; small outlines along the bottom of example 1 | Noise labels teach the model to fire on nothing | Yes: dropped in the training notebook |
| A3 | `glass` and `stone_cladding` labelled per pane / per stone instead of one box per continuous area | Examples 2, 3 and 4 (each window pane is its own `glass` label) | These classes become many tiny objects, unlike the other classes | No: known inconsistency, fix in the next version |
| A4 | Likely mislabel: in example 5, the purple `stone_cladding` outlines run along the **roofs**, and the facade itself looks like cream render | `annotation_05_stone_cladding.jpg` | Teaches the model that roof tiles are stone cladding: a likely source of false positives | No: to review in Roboflow |
| A5 | Black-and-white source photo labelled `painted_render` (historic survey photo) | `annotation_04_painted_render.jpg` | Colour is one of the main cues for material; grey images blur the render / concrete boundary | No: remove or keep few, flag as hard case |

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

Check whether E1 (missed brick wall) still happens with the final model, and
whether A4 (roofs labelled as stone) shows up as roof false positives, and
say so here.

---

## 3 prioritized next data improvements

Ordered by expected impact. Update after Run 03 if the final errors point
somewhere else.

1. **Relabel to one standard (fixes A3, A4, and part of E2).** One box per
   continuous material area for every class, including `glass` and
   `stone_cladding`; review every `stone_cladding` image for roofs or render
   labelled as stone; split L-shaped areas into 2–3 rectangles.
2. **Balance the classes.** `cladding_panel` (58 training images) and
   `painted_render` (only 26 validation boxes) need the most. Target at least
   80 images per class, including large, frame-filling walls (the case in E1).
3. **Hard cases on purpose.** Strong shadows, glare on glass, overcast or
   dusk light, partial occlusion (trees, scaffolding), and look-alike
   materials (stone-look panels, brick slips). Keep black-and-white images
   out of training, or tag them so they can be evaluated separately (A5).
