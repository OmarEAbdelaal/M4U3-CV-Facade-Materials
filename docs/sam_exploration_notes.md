# SAM Exploration Notes: What Helped and What Failed

The brief asks for notes on using SAM (Segment Anything) as an annotation aid.
In this project, SAM was used through Roboflow's Smart Polygon tool while
labelling facade materials for a **detection** model (YOLOv8, boxes). The
notes below are measured from the released dataset (Release `v1.0`,
Roboflow `facade-materials-3mrrt` version 4, training split).

## How SAM was used

- **Tool:** Roboflow Annotate → Smart Polygon (SAM), one click or a few clicks per region.
- **Scope:** mixed with hand-drawn boxes. 236 of the 354 training label files
  contain SAM polygons, 265 contain plain boxes, and 147 contain both.
- **Intended output:** polygons exported as boxes for YOLOv8. This assumption
  was wrong (see "What failed", item 1).

| Class | SAM polygons | Hand-drawn boxes | Median polygon size (share of image) | Median box size |
|---|---|---|---|---|
| brick | 1,915 | 276 | 0.25% | 1.6% |
| cladding_panel | 131 | 91 | 0.4% | 13.1% |
| exposed_concrete | 258 | 17 | 3.3% | 3.8% |
| glass | 3,161 | 5,852 | 0.19% | 0.27% |
| painted_render | 144 | 193 | 0.02% | 1.1% |
| stone_cladding | 3,919 | 558 | 0.20% | ~0% (mostly fragments) |
| **Total** | **9,528** | **6,987** | | |

## What helped

1. **Speed on large, uniform surfaces.** On `exposed_concrete` and plain
   `cladding_panel` walls, one click gave a clean outline. Polygon and box
   sizes are close for concrete (3.3% vs 3.8% median), so SAM matched a
   hand-drawn box there.
2. **Tight outlines on irregular edges.** SAM followed stepped parapets,
   curved facades and wall edges behind trees better than a rectangle does.
   Converted to its tightest box, a SAM polygon is a well-fitted detection box.
3. **Recoverable.** Because every polygon converts exactly to its tightest
   box, nothing had to be relabelled by hand. Notebook `01b_clean_dataset`
   converts all polygons in code (step 1).

## What failed

1. **Export format.** Roboflow's YOLOv8 export kept the SAM polygons as
   polygons. 181 label files mixed polygons and boxes, which the YOLOv8
   detection loader misreads. This silently corrupted boxes and is the most
   likely cause of the failed Run 02. *Lesson: open the exported label files
   before training.* (Project log, entry 8, P5.)
2. **SAM segments elements, not materials.** On `glass`, `brick` and
   `stone_cladding`, a click selects one pane, one brick course or one stone,
   not the whole material area. The median SAM polygon covers only about
   0.2% of the image, which breaks the "one box per continuous material area"
   rule in `02_class_definitions.md`. This produced 100+ boxes per image
   for glass (problem A3).
3. **Fragments.** SAM clicks left 1,360 polygons under 4 px wide or tall in
   the training split, mostly `stone_cladding` (1,028), `brick` (149) and
   `glass` (125). They are noise labels. Removed in Release `v1.1` (steps 2 and 5).
4. **Wrong region on low-contrast scenes.** SAM follows edges, not material.
   Where the roof, facade and sky have similar tones, the mask followed the
   roof line (example 5, `stone_cladding` outlines on roofs: problem A4), and
   on the black-and-white photo (A5) the render / concrete boundary was not
   found.
5. **Nested duplicates.** Clicking on a region that already had a hand box
   added a second, smaller label inside it (144 of 216 `exposed_concrete`
   boxes were nested; problem A7).

## Result after cleaning

Release `v1.1` keeps the useful part of SAM (tight outlines) and removes the
harmful part (fragments, per-element granularity, nested duplicates):
13,730 → 968 training boxes. Run 04, trained on v1.1, reached validation
mAP@50 0.394 against 0.143 for Run 03 on the uncleaned labels (the answer
keys differ; see README section 6).

## Recommendation for the next dataset version

- Use SAM **only** for large, uniform surfaces (`exposed_concrete`,
  `painted_render`, `cladding_panel`), then check the box.
- For `glass`, `brick` and `stone_cladding`, draw one box per continuous area
  by hand, or use SAM with several positive clicks across the whole area and
  merge.
- Export as **YOLOv8 bounding boxes** and open two label files to confirm
  each line has 5 values.
- If the task is ever moved to segmentation, SAM masks become the right
  label type and this workflow should be revisited.
