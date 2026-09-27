# Facade Material Detection from Building Photos

MAICEN · Module 4 · Unit 3 · Computer Vision

A YOLOv8 object detector that finds six facade materials in building
photos, built as a reproducible, documented prototype.

| Notebook | Open |
|---|---|
| 01 — Baseline: pretrained YOLOv8 on facade photos | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/OmarEAbdelaal/M4U3-CV-Facade-Materials/blob/main/notebooks/01_baseline_inference.ipynb) |
| 02 — Train and evaluate YOLOv8s | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/OmarEAbdelaal/M4U3-CV-Facade-Materials/blob/main/notebooks/02_training_eval.ipynb) |

---

## 1. Problem and success criteria

**AECO problem.** Value-engineering decisions about facade materials are
made during design (pre-concept to IFC). Once construction starts, there is
rarely a systematic check that the material decided on is the material
actually installed, so substitutions can go unnoticed.

**What this model does.** Given a photo of a building, it detects which
facade materials are visible and where. In a wider workflow, its output is
compared with the design-stage specification, and any mismatch is flagged
for a person to review.

**What it does not do.** It does not certify compliance. It is a screening
aid (see [Governance](#9-governance-and-limitations)).

**Success criteria (prototype level)**

| Metric (validation set) | Target | Why |
|---|---|---|
| mAP@50 | ≥ 0.50 | Course scale: 0.5–0.7 is prototype level |
| Recall | ≥ 0.50 overall; prioritised for `glass`, `cladding_panel`, `stone_cladding`, `exposed_concrete` | A missed material means a substitution goes unflagged |
| Precision | ≥ 0.50 | False alarms cost review time, but less than misses |

## 2. Classes and label rules

| Class | Definition (short) |
|---|---|
| `brick` | Fired clay or concrete brick units exposed as the finish |
| `cladding_panel` | Metal, composite or rainscreen panel systems |
| `exposed_concrete` | Unpainted, unclad concrete left as the finish |
| `glass` | Glazing: curtain wall, windows, glass infill |
| `painted_render` | Painted plaster, stucco or render |
| `stone_cladding` | Natural or reconstituted stone panels |

Main rules: one box per continuous material area; boundary cases (glass vs.
reflective panels, concrete vs. grey render, stone vs. stone-look panels)
resolved as written. Full definitions:
[`docs/02_class_definitions.md`](docs/02_class_definitions.md).

> The original annotations did not follow these rules consistently (per-pane
> glass, nested boxes, fragments). Release `v1.1` enforces them in code; see
> section 3 and [`docs/error_analysis.md`](docs/error_analysis.md).

## 3. Dataset

Two frozen releases. The notebooks train on **v1.1**.

| | Release `v1.0`: original export | Release `v1.1`: cleaned (used for training) |
|---|---|---|
| Source | Roboflow Universe [facade-materials-3mrrt, **version 4**](https://universe.roboflow.com/omar-el-sayed-pzb69/facade-materials-3mrrt/dataset/4) | Release `v1.0` after [`notebooks/01b_clean_dataset.ipynb`](notebooks/01b_clean_dataset.ipynb) |
| File | [`facade-materials-v3-yolov8.zip`](https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/download/v1.0/facade-materials-v3-yolov8.zip) | [`facade-materials-v4-clean-yolov8.zip`](https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/download/v1.1/facade-materials-v4-clean-yolov8.zip) |
| SHA256 | `134e41be8c310452bbe8ea33fd55a891c6e29ca863f3c9b8df3827f59bb951cd` | `bdcf62ff01fd58892ee22dd07cf2b481fa1194d0946dd5f78c6b41dc04187164` |
| Images (train · valid · test) | 354 · 51 · 25 | 322 · 45 · 23 |
| Boxes (train · valid · test) | 13,730 · 2,179 · 756 (after converting polygons and removing fragments) | 968 · 149 · 78 |
| Format | YOLOv8, polygons mixed with boxes | YOLOv8, boxes only |

Common to both:

| Item | Value |
|---|---|
| Split of source images | 70 / 20 / 10 (v1.0: 177 · 51 · 25; v1.1: 161 · 45 · 23); training images doubled by augmentation |
| Preprocessing | Auto-orient; resize to 640 × 640 (stretch) |
| Augmentation (train only) | 2 outputs per image: 50% horizontal flip, rotation ±10°, brightness ±15% |
| Sources | Wikimedia Commons (CC0, public domain, CC BY, CC BY-SA) |
| License | Currently labelled CC BY 4.0 on Roboflow; see the note below |

The `v1.0` file name says `v3` but the content is Roboflow version 4; the
SHA256 fixes the exact file.

### Label cleaning (v1.0 → v1.1)

Rules applied to every label file, in this order (settings in notebook 01b):

1. SAM polygons → their tightest box (11,963 polygons).
2. Fragments narrower or shorter than 4 px removed (3,563).
3. Same-class boxes that overlap or are closer than 0.5% of the image width
   → one box per continuous area (all classes). This also removes boxes
   nested inside larger boxes of the same class, e.g. 144 of 216
   `exposed_concrete` boxes.
4. Two different classes on the same area (IoU > 0.7) → keep the class the
   photo was collected for, otherwise the larger box (21 cases).
5. Boxes smaller than 1% of the image area removed (2,258).
6. Images left with no label of their own material removed (24 source
   images; 14 are `painted_render`). Listed in
   [`results/data_cleaning/removed_images.csv`](results/data_cleaning/removed_images.csv)
   for relabelling.

Images are not changed. The zip is written deterministically, so running
notebook 01b again produces the same SHA256.
Before/after: [`results/data_cleaning/cleaning_before_after.jpg`](results/data_cleaning/cleaning_before_after.jpg).

**Boxes per class in v1.1** (images in brackets):

| Class | Train | Valid | Test |
|---|---|---|---|
| brick | 168 (109) | 22 (13) | 16 (7) |
| cladding_panel | 63 (54) | 13 (10) | 5 (3) |
| exposed_concrete | 94 (77) | 11 (8) | 4 (4) |
| glass | 434 (138) | 68 (22) | 36 (10) |
| painted_render | 77 (65) | 16 (13) | 3 (3) |
| stone_cladding | 132 (82) | 19 (14) | 14 (6) |

The test split is small (23 images; 3 for some classes), so per-class test
scores are noisy. Validation (45 images) is the main reference.

**License note.** Every image keeps its own source license. If any image is
CC BY-SA, the dataset as a whole must be CC BY-SA 4.0; this is being checked
(see [`docs/governance_checklist.md`](docs/governance_checklist.md)).

## 4. How to reproduce (Google Colab)

No accounts, API keys or local installs are needed.

1. Click a **Open in Colab** badge above.
2. For notebook 02: **Runtime → Change runtime type → T4 GPU**.
3. **Runtime → Disconnect and delete runtime**, then **Runtime → Run all**.
4. Notebook 02 downloads Release `v1.1`, checks the SHA256, checks the
   labels, trains, evaluates, and writes everything to
   `/content/results/yolov8s_clean/` (same layout as this repo's `results/`
   folder). Cell 11 downloads `results_yolov8s_clean.zip`; cell 12 compares
   with Run 03.

To rebuild `v1.1` from `v1.0`: run
[`notebooks/01b_clean_dataset.ipynb`](notebooks/01b_clean_dataset.ipynb)
(no GPU needed). It prints the SHA256 of the zip it writes, which should
match the one in section 3.

Without a GPU: set `QUICK_RUN = True` in cell 2 of notebook 02 for a 5-epoch
verification run, and set `WEIGHTS_URL` to the published `best.pt` so
evaluation and inference use the fully trained model.

A secondary Roboflow download cell is included for the author's workflow.
It is skipped automatically when the keyless dataset is present. Roboflow
holds the uncleaned version 4, which must go through notebook 01b first.

## 5. Reproducibility checklist

- **Dataset:** Release `v1.1`, SHA256 `bdcf62ff…4187164` (cleaned from Release `v1.0`, SHA256 `134e41be…51cd`, Roboflow `facade-materials-3mrrt` version 4)
- **Cleaning:** notebook 01b; `MIN_BOX_PX=4`, `MERGE_GAP=0.005`, `STACK_IOU=0.7`, `MIN_AREA=0.01`, images missing their own class removed
- **Model:** `yolov8s.pt` (COCO-pretrained, fine-tuned)
- **Training:** `epochs=50`, `batch=16`, `imgsz=640`, `seed=0`, `deterministic=True`, optimizer auto (AdamW)
- **Library:** `ultralytics==8.4.163` (pinned in cell 1); versions of torch and Python are written to `results/<run>/run_info.json`
- **Hardware:** Google Colab, NVIDIA T4 GPU
- **Randomness:** fixed seed; the 10 validation images for the evidence pack are chosen with the same seed

## 6. Results

### Baseline: pretrained YOLOv8s (COCO), no training

| True material | Pretrained model detected |
|---|---|
| brick | bicycle (0.88) |
| glass | nothing |
| cladding_panel | nothing |
| stone_cladding | car (0.94), bird (0.25) |

COCO has no material classes, so the pretrained model can only report
objects in front of a facade. This is the reason for a custom model.
Figure: [`results/baseline/baseline_comparison.jpg`](results/baseline/baseline_comparison.jpg).

### Exploration runs on Roboflow (not the final model)

| Run | Model | mAP@50 | Precision | Recall |
|---|---|---|---|---|
| 01 | RF-DETR NAS, ~50 images | 0.304 | 0.356 | 0.182 |
| 02 | YOLO26-S, different version | 0.001 | 0.007 | 0.015 |

Both were trained before the label problems (A1, A2) were found and fixed.
Details: [`docs/project_log.md`](docs/project_log.md) (entry 6).

### Run 03: first full training run (YOLOv8s, 50 epochs)

Notebook version 1: labels cleaned (polygons → boxes, fragments dropped) but
**not merged**; scored on labels as annotated (per pane / per stone).

| Split | Precision | Recall | mAP@50 | mAP@50-95 |
|---|---|---|---|---|
| Validation | 0.372 | 0.159 | 0.143 | 0.093 |
| Test | 0.332 | 0.216 | 0.219 | 0.141 |

Below the prototype target (mAP@50 ≥ 0.50), with recall especially low.
This led to the label measurement in `docs/project_log.md` (entry 11).

### Run 04: YOLOv8s, 50 epochs, cleaned labels (Release v1.1)

| Split | Precision | Recall | mAP@50 | mAP@50-95 |
|---|---|---|---|---|
| Validation | _pending_ | _pending_ | _pending_ | _pending_ |
| Test | _pending_ | _pending_ | _pending_ | _pending_ |

Run 03 and Run 04 are not scored on the same labels (v1.0 vs. v1.1), so the
change in scores mixes two effects: a better-trained model and an easier,
more consistent answer key. The prediction images in the evidence pack show
the practical difference.

Per-class results: `results/yolov8s_clean/metrics.csv` · Curves and
confusion matrices: `results/yolov8s_clean/curves/` · With Run 03:
`results/comparison.csv`

**Key takeaways:** _to be written after Run 04._

**Weights:** `best.pt` — _link to be added (GitHub Release)._

## 7. Reproducibility proof

| Item | Value |
|---|---|
| Last successful run | _pending (date and time)_ |
| Hardware | _pending (from `results/run_info.json`)_ |
| Training time | _pending_ |
| Expected runtime | _pending_ |
| Tested from a clean runtime | _pending_ (Disconnect and delete runtime → Run all) |

## 8. Evidence and error analysis

| Evidence | Location |
|---|---|
| Annotation examples (5) | [`results/evidence/annotations/`](results/evidence/annotations/) |
| Validation predictions (10) | `results/yolov8s_clean/evidence/val_predictions/` (after Run 04) |
| New-image predictions (5) | `results/yolov8s_clean/evidence/new_predictions/` (after Run 04) |
| Label cleaning report | [`results/data_cleaning/`](results/data_cleaning/) |
| Exploration-run screenshots | [`results/evidence/training_runs/`](results/evidence/training_runs/) |
| Error analysis: 3 FP, 3 FN, next data fixes | [`docs/error_analysis.md`](docs/error_analysis.md) |
| Full project log (every action and decision) | [`docs/project_log.md`](docs/project_log.md) |

## 9. Governance and limitations

- **Screening aid, not certification.** The model flags where a facade
  appears to differ from the specification; a qualified person decides. It
  must not be used for fire-safety or life-safety judgements.
- **False negatives cost more than false positives:** a missed material is
  a substitution nobody reviews. Recall is prioritised.
- **Privacy:** only openly licensed Wikimedia Commons images; no client or
  site photos. Incidental faces and plates are to be blurred in the next
  dataset version.

Full checklist: [`docs/governance_checklist.md`](docs/governance_checklist.md).

## 10. Repository structure

```
notebooks/
  00_collect_images.ipynb       image collection from Wikimedia Commons (one-off)
  00b_collect_cladding.ipynb    targeted re-collection for cladding_panel (one-off)
  01_baseline_inference.ipynb   pretrained YOLOv8 vs. our classes
  01b_clean_dataset.ipynb       label cleaning: Release v1.0 -> v1.1
  02_training_eval.ipynb        training, evaluation, evidence
docs/
  02_class_definitions.md       classes and labelling rules
  error_analysis.md             model and data errors, next data fixes
  governance_checklist.md       privacy, limitations, risk, licensing
  project_log.md                every action, problem and decision
results/
  baseline/                     notebook 01 outputs
  data_cleaning/                label-cleaning report (counts, removed images, before/after)
  yolov8s_clean/                Run 04: metrics, curves, predictions
  evidence/                     annotations, predictions, exploration runs
data/                           reference only; the dataset is a Release asset
```

## 11. License

- **Code, notebooks and documentation:** MIT, see [`LICENSE`](LICENSE).
- **Dataset images:** each keeps its source license (CC0, public domain,
  CC BY or CC BY-SA).
- **Trained weights:** AGPL-3.0, because they are produced with Ultralytics
  YOLOv8 (AGPL-3.0).
