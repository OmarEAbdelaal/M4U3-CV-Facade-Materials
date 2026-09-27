# Error Analysis — Facade Material Detection

This document lists what the model gets wrong and why. The assignment asks
for 3 false positives, 3 false negatives and 3 prioritized data improvements,
taken from the final model (Run 04, trained on the cleaned Release v1.1). Observations from the earlier runs are
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
| A1 | SAM polygons and plain boxes mixed in the same label file (181 files) | Label files; example 2 vs. examples 1 and 3 | The YOLOv8 detection loader misreads the boxes in those files | Yes: Release v1.1 (notebook 01b, step 1) |
| A2 | About 3,500 boxes under 4 px, mostly SAM fragments; plus about 2,250 more boxes under 1% of the image | Label files; small outlines along the bottom of example 1 | Noise labels teach the model to fire on nothing | Yes: Release v1.1 (steps 2 and 5) |
| A3 | `glass`, `stone_cladding` and `brick` labelled per pane / per stone (median 101, 76 and 38 boxes per image) instead of one box per continuous area (3–10 for the other classes) | Examples 2, 3 and 4 (each window pane is its own `glass` label) | These classes become many tiny objects: `glass` alone is about 60% of all training boxes | Yes, in code: Release v1.1 merges touching same-class boxes into one per area for all classes (step 3). Relabelling in Roboflow remains the long-term fix |
| A4 | Likely mislabel: in example 5, the purple `stone_cladding` outlines run along the **roofs**, and the facade itself looks like cream render | `annotation_05_stone_cladding.jpg` | Teaches the model that roof tiles are stone cladding: a likely source of false positives | No: to review in Roboflow |
| A5 | Black-and-white source photo labelled `painted_render` (historic survey photo) | `annotation_04_painted_render.jpg` | Colour is one of the main cues for material; grey images blur the render / concrete boundary | No: remove or keep few, flag as hard case |
| A6 | Missing labels: 14 of 39 `painted_render` images have no `painted_render` label, and 31 images across the dataset are less than 25% labelled | Label coverage check, 27 Sep | Unlabelled render walls teach the model that render is background; correct detections then count as false positives | Partly: the 24 images with no label of their own material are removed in v1.1 (`results/data_cleaning/removed_images.csv`); partly labelled images remain |
| A7 | Nested duplicates and stacked labels: boxes inside larger boxes of the same class (144 of 216 `exposed_concrete` boxes), and two different classes on the same area (21 cases) | Label analysis, 27 Sep | Double-counted objects; contradictory targets for the same pixels | Yes: Release v1.1 (steps 3 and 4) |

---

## Final model (Run 04, trained on Release v1.1)

Validation: precision 0.708, recall 0.345, mAP@50 0.394. Source: the
normalised confusion matrix in
[`results/yolov8s_clean/curves/`](../results/yolov8s_clean/curves/) and the
prediction images in
[`val_predictions/`](../results/yolov8s_clean/evidence/val_predictions/) and
[`new_predictions/`](../results/yolov8s_clean/evidence/new_predictions/).

**Overall pattern.** The model misses far more than it invents. The
background row of the confusion matrix (true objects predicted as nothing)
is the largest error for every class:

| Class | Correct | Missed (predicted as background) | Main confusion |
|---|---|---|---|
| brick | 0.36 | 0.55 | glass 0.09 |
| cladding_panel | 0.23 | **0.77** | — |
| exposed_concrete | 0.55 | 0.45 | — |
| glass | 0.34 | **0.63** | brick, cladding 0.01 |
| painted_render | 0.25 | 0.56 | **brick 0.12**, cladding 0.06 |
| stone_cladding | 0.53 | 0.42 | glass 0.05 |

Of all detections on unlabelled areas (background column), 66% are `glass`,
11% `stone_cladding` and 9% `painted_render`.

### 3 false positives

| # | What was predicted | What is really there | Why I think it happened | Evidence |
|---|---|---|---|---|
| FP1 | `glass` on unlabelled areas: 66% of all background detections | Mostly real windows that were never labelled in partly labelled images; some dark or reflective panels | Label gap A6: the answer key is incomplete, so a correct window counts as a false positive. Dark panels share glass's colour and reflections | Confusion matrix, background column |
| FP2 | `stone_cladding` on background: 11% of background detections | Roof edges, sills and light render near stone areas | Problem A4: some `stone_cladding` labels outline roofs, so the model learned roof texture as stone | Confusion matrix, background column |
| FP3 | `brick` on `painted_render` walls: 12% of true render | Warm-coloured render, often with joint lines or shadows | Colour and texture overlap (terracotta or ochre render); only 77 `painted_render` training boxes after 14 render images were removed for missing labels | Confusion matrix, `painted_render` column |

### 3 false negatives

| # | What was missed | Why I think it happened | Evidence |
|---|---|---|---|
| FN1 | `cladding_panel`: 77% of validation boxes missed (recall 0.23) | Fewest examples (54 training images, 13 validation boxes), and very varied appearance: flat, folded, perforated and reflective metal, which looks like glass or sky | Confusion matrix; per-class table in README section 6 |
| FN2 | `glass`: 63% missed (recall 0.27) although it is the largest class | After merging, curtain walls are one large box while punched windows stay small separate boxes, so the class has two very different object scales; sky reflections hide the edges | Confusion matrix; validation predictions |
| FN3 | `brick` walls on the test split: recall 0.06 (7 images, 16 boxes) | E1 again: frame-filling brick walls are under-represented, and a 7-image split makes the score very unstable (validation recall is 0.32) | Test metrics, `metrics.csv` |

**Early errors revisited.** E1 (missed brick wall) is reduced but not gone:
brick recall is 0.32 on validation and 0.06 on the 7-image test split.
E2 (loose glass boxes) is largely fixed by merging per area. A4 (roofs
labelled as stone) shows up as FP2.

---

## 3 prioritized next data improvements

Ordered by expected impact, and confirmed by the Run 04 errors above: the
largest errors are misses on small classes and "false" glass detections on
unlabelled windows.

1. **Relabel to one standard (fixes A3, A4, A6 properly; v1.1 only enforces it in code).** Start with the 24 images in `results/data_cleaning/removed_images.csv`. One box per
   continuous material area for every class, including `glass` and
   `stone_cladding`; review every `stone_cladding` image for roofs or render
   labelled as stone; split L-shaped areas into 2–3 rectangles.
2. **Balance the classes.** `cladding_panel` (54 training images, 13
   validation boxes, 77% missed) and `painted_render` (16 validation boxes,
   confused with brick) need the most. Target at least
   80 images per class, including large, frame-filling walls (the case in E1).
3. **Hard cases on purpose.** Strong shadows, glare on glass, overcast or
   dusk light, partial occlusion (trees, scaffolding), and look-alike
   materials (stone-look panels, brick slips). Keep black-and-white images
   out of training, or tag them so they can be evaluated separately (A5).
