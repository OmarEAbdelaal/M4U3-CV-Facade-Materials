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

Main rules: one box per continuous material area; label every material that
covers at least 5% of the image; boundary cases (glass vs. reflective panels,
concrete vs. grey render, stone vs. stone-look panels) resolved as written.
Full definitions: [`docs/02_class_definitions.md`](docs/02_class_definitions.md).

> Known gap: in the released version, `glass` and `stone_cladding` were
> mostly labelled per pane / per stone rather than per area. See
> [`docs/error_analysis.md`](docs/error_analysis.md) (A3).

## 3. Dataset

| Item | Value |
|---|---|
| Roboflow Universe | [facade-materials-3mrrt, **version 4**](https://universe.roboflow.com/omar-el-sayed-pzb69/facade-materials-3mrrt/dataset/4) |
| Frozen copy (used by the notebooks) | GitHub Release [`v1.0`](https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/tag/v1.0) → [`facade-materials-v3-yolov8.zip`](https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/download/v1.0/facade-materials-v3-yolov8.zip) |
| SHA256 | `134e41be8c310452bbe8ea33fd55a891c6e29ca863f3c9b8df3827f59bb951cd` |
| Format | YOLOv8 (same label format as YOLOv11) |
| Images | 430: 354 train · 51 valid · 25 test |
| Split of source images | 177 · 51 · 25 (70 / 20 / 10); training images doubled by augmentation |
| Preprocessing | Auto-orient; resize to 640 × 640 (stretch) |
| Augmentation (train only) | 2 outputs per image: 50% horizontal flip, rotation ±10°, brightness ±15% |
| Sources | Wikimedia Commons (CC0, public domain, CC BY, CC BY-SA) |
| License | Currently labelled CC BY 4.0 on Roboflow; see the note below |

The file name says `v3` but the content is Roboflow version 4; the SHA256
fixes the exact file.

**Images per class and split** (an image with several materials counts
once for each):

| Class | Train | Valid | Test |
|---|---|---|---|
| brick | 142 | 16 | 8 |
| cladding_panel | 58 | 12 | 4 |
| exposed_concrete | 78 | 8 | 4 |
| glass | 188 | 29 | 13 |
| painted_render | 94 | 13 | 6 |
| stone_cladding | 87 | 16 | 6 |

**Label cleanup at training time.** The export mixes SAM polygons with boxes
and contains about 3,500 sub-4-px fragments. `02_training_eval.ipynb`
converts polygons to boxes and drops the fragments on the extracted copy;
the released zip is unchanged. Box counts after cleanup are in
[`docs/project_log.md`](docs/project_log.md) (entry 8).

**License note.** Every image keeps its own source license. If any image is
CC BY-SA, the dataset as a whole must be CC BY-SA 4.0; this is being checked
(see [`docs/governance_checklist.md`](docs/governance_checklist.md)).

## 4. How to reproduce (Google Colab)

No accounts, API keys or local installs are needed.

1. Click a **Open in Colab** badge above.
2. For notebook 02: **Runtime → Change runtime type → T4 GPU**.
3. **Runtime → Disconnect and delete runtime**, then **Runtime → Run all**.
4. The notebook downloads the dataset from the GitHub Release, checks the
   SHA256, cleans the labels, trains, evaluates, and writes everything to
   `/content/results/` (same layout as this repo's `results/` folder). The
   last cell downloads `results_package.zip`.

Without a GPU: set `QUICK_RUN = True` in cell 2 for a 5-epoch verification
run, and set `WEIGHTS_URL` to the published `best.pt` so evaluation and
inference use the fully trained model.

A secondary Roboflow download cell is included for the author's workflow.
It is skipped automatically when the keyless dataset is present.

## 5. Reproducibility checklist

- **Dataset:** Roboflow `facade-materials-3mrrt` version 4 → Release `v1.0`, SHA256 `134e41be…51cd`
- **Model:** `yolov8s.pt` (COCO-pretrained, fine-tuned)
- **Training:** `epochs=50`, `batch=16`, `imgsz=640`, `seed=0`, `deterministic=True`, optimizer auto (AdamW)
- **Label cleanup:** polygons → boxes; boxes under 4 px dropped
- **Library:** `ultralytics==8.4.163` (pinned in cell 1); versions of torch and Python are written to `results/run_info.json`
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

### Final model: YOLOv8s, 50 epochs (Run 03)

| Split | Precision | Recall | mAP@50 | mAP@50-95 |
|---|---|---|---|---|
| Validation | 0.372 | 0.159 | 0.143 | 0.093 |
| Test | 0.332 | 0.216 | 0.219 | 0.141 |

Per-class results: `results/metrics.csv` · Curves and confusion matrices:
`results/curves/`

**Key takeaways:** _to be written after Run 03._

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
| Validation predictions (10) | `results/evidence/val_predictions/` (after Run 03) |
| New-image predictions (5) | `results/evidence/new_predictions/` (after Run 03) |
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
  02_training_eval.ipynb        training, evaluation, evidence
docs/
  02_class_definitions.md       classes and labelling rules
  error_analysis.md             model and data errors, next data fixes
  governance_checklist.md       privacy, limitations, risk, licensing
  project_log.md                every action, problem and decision
results/
  baseline/                     notebook 01 outputs
  curves/                       training curves, confusion matrices (Run 03)
  evidence/                     annotations, predictions, exploration runs
data/                           reference only; the dataset is a Release asset
```

## 11. License

- **Code, notebooks and documentation:** MIT, see [`LICENSE`](LICENSE).
- **Dataset images:** each keeps its source license (CC0, public domain,
  CC BY or CC BY-SA).
- **Trained weights:** AGPL-3.0, because they are produced with Ultralytics
  YOLOv8 (AGPL-3.0).
