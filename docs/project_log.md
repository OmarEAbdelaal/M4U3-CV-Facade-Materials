# Project Log — Facade Material Detection (M4U3)

One running record of every action, problem and decision in this assignment,
in date order. Each entry says what was done, what went wrong (if anything),
how it was resolved, and what was learned. Failures are kept on purpose: they
explain why the final setup looks the way it does.

Related documents:
- Class list and labelling rules: [`02_class_definitions.md`](02_class_definitions.md)
- Error analysis required by the brief: [`error_analysis.md`](error_analysis.md)
- Training-run screenshots: [`../results/evidence/training_runs/`](../results/evidence/training_runs/)

All times are UAE time (UTC+4).

---

## Summary of decisions

| # | Decision | Why |
|---|---|---|
| D1 | Detect facade **materials**, 6 classes | Fits a two-day deadline and links to the Final Master's Project (value engineering) |
| D2 | Collect images from **Wikimedia Commons** only for the published dataset | Every image carries a clear free license, so the dataset can be republished in a GitHub Release |
| D3 | **Bounding boxes**, one box per continuous material area | The brief requires YOLOv8 detection; per-panel boxes would teach "panel shape" instead of "material" |
| D4 | SAM masks allowed as an annotation shortcut; polygons converted to boxes **in the training notebook** | Roboflow's YOLOv8 export kept the polygons (see entry 8), so the conversion is done in code |
| D5 | Keep the git repository **out of live Google Drive sync** where possible | Drive wrote `desktop.ini` files into `.git` and corrupted it |
| D6 | Re-collect `cladding_panel` with a targeted search | The first batch contained almost no real cladding facades |
| D7 | Final model is **YOLOv8s trained in Colab** | Brief requires a reproducible notebook; Roboflow-hosted runs are exploration only |
| D8 | Dataset frozen as a **GitHub Release** with SHA256 | Course reproducibility standard: keyless download, verified file |
| D9 | Drop boxes smaller than 4 px during training | About 3,500 boxes were SAM fragments, not real objects |

---

## 1. Scope and problem framing — 26 Sep 2026

**Action.** Chose a Computer Vision use case that connects to the Final
Master's Project on continuous value engineering with AI and BIM.

**Reasoning.** Value-engineering decisions are made during design
(pre-concept to IFC), where there is nothing yet to photograph. Computer Vision
fits later, as the verification step: checking on site whether the material
decided during design is the material actually installed.

**Scope cut.** With one to two days available, the task was narrowed to a
single capability: detect which facade material is visible in an image.

**Problem statement.** Detect and classify the visible facade material in an
image, to support checks between the design-stage material specification and
what is later observed.

---

## 2. Classes — 26–27 Sep 2026

**Action.** Extended an existing Roboflow project (4–5 material classes) with
one new class, `stone_cladding`, giving 6 classes:

`glass`, `painted_render`, `exposed_concrete`, `brick`, `cladding_panel`, `stone_cladding`

**Action.** Wrote a Class Definition Contract for all 6 classes in
`02_class_definitions.md`: positive definition, boundary cases, box rule,
minimum size, decision supported, and which error type costs more.

**Reflection.** The hardest boundaries are glass vs. reflective panels,
exposed concrete vs. grey render in shadow, and stone vs. stone-look composite
panels. These are written down before annotation so labels stay consistent.

---

## 3. First image collection — 27 Sep 2026 (early morning)

**Action.** Wrote `notebooks/00_collect_images.ipynb`, which queries the
Wikimedia Commons API with one keyword phrase per class, downloads about 50
images per class, and logs source URL, license and author to `sources.txt`.

**Why Wikimedia Commons.** Publishing the dataset as a GitHub Release is
redistribution. Commons images carry explicit CC0 / CC BY / CC BY-SA licenses.
Unsplash and Pexels were not used for the published dataset because their
licenses restrict repackaging photos into a redistributed collection.

**Result.** Usable for most classes, but see entry 7 for `cladding_panel`.

---

## 4. Repository setup and git problems — 27 Sep 2026, 03:37–04:00

The repository was first created inside a Google Drive folder
(`G:\My Drive\...\M4U3-CV-Assignment-Facade-Materials`). Four problems came up
in sequence.

| Problem | Error seen | Cause | Fix |
|---|---|---|---|
| P1 | `Author identity unknown` | git had no name/email on this computer | Set `git config --global user.name` and `user.email` |
| P2 | `src refspec main does not match any` | The commit in P1 had failed, so no branch existed | Committed again after P1 |
| P3 | `rejected ... (fetch first)` | The GitHub repo was created with its own initial commit | Tried `git pull --allow-unrelated-histories` |
| P4 | `fatal: bad object refs/desktop.ini` | Google Drive sync wrote a `desktop.ini` file inside `.git/refs`, which git read as a broken branch reference | Moved away from the Drive copy (see below) |

**Resolution.** Cloned the GitHub repository fresh, copied the project files
into the clone while excluding the old `.git` folder and all `desktop.ini`
files (`robocopy ... /XD .git /XF desktop.ini`), then committed and pushed.
Because the clone already contained GitHub's initial commit, the push was a
normal fast-forward. `desktop.ini` was also added to `.gitignore`.

**Reflection.**
- A live cloud-sync folder and a git repository both try to manage the same
  files. Sync clients inject metadata files and can lock files mid-write.
- **Open risk:** the working repository (`M4U3-repo`) currently sits on the
  Google Drive path again. If a `desktop.ini` error reappears, move the repo to
  a local folder (for example `C:\Projects`) and keep Drive for backups only.

---

## 5. Annotation approach — 27 Sep 2026, around 12:20

**Question.** Draw bounding boxes, or masks; and one large box around a whole
material area, or one box per panel?

**Decision.**
- **Bounding boxes**, because the brief requires YOLOv8 object detection.
  Masks would need a segmentation model and 3–5 times the annotation effort.
- **One box per continuous material area**, not per panel, so the model
  learns the material rather than the shape of individual panels. L-shaped or
  wrapping areas are split into 2–3 rectangles.

**What actually happened.** Some images were annotated with SAM masks and
others with hand-drawn boxes. At the time it was assumed that Roboflow's
YOLOv8 export converts masks to boxes. **That assumption turned out to be
wrong** (see entry 8): the export kept the polygons, and the conversion is now
done in the training notebook.

**Reflection.** Materials are surfaces, not objects, so a box always includes
some neighbouring material. This is a known limitation of using detection for
this task; segmentation is noted as a future improvement.

---

## 6. Training runs on Roboflow — 27 Sep 2026

Metrics are Roboflow test-set values at a 50% confidence threshold.

| Run | Model | Dataset version | mAP@50 | Precision | Recall | F1 | Outcome |
|---|---|---|---|---|---|---|---|
| 01 | RF-DETR NAS | `2026-09-27 5:08am` | 30.4% | 35.6% | 18.2% | 24.1% | Weak baseline |
| 02 | YOLO26 Object Detection (Small) | `2026-09-27 12:50pm` | 0.1% | 0.7% | 1.5% | 1.0% | Failed to learn |
| 03 | YOLOv8s, 50 epochs (Colab, notebook v1: labels cleaned, not merged) | `2026-09-27 4:52pm` (v4, Release `v1.0`) | 0.143 (test 0.219) | 0.372 (test 0.332) | 0.159 (test 0.216) | — | Below target; low recall. Scored on as-annotated labels |
| 04 | YOLOv8s, 50 epochs (Colab), cleaned labels | Release `v1.1` (v4 cleaned) | _TBD_ | _TBD_ | _TBD_ | _TBD_ | See entry 12 |

### Run 01 — RF-DETR NAS

- Dataset: about 50 images or fewer across 6 classes. Latency 67.9 ms per image.
- On the sample image (red-brick facade with three windows, screenshot
  `run01_rfdetr_nas_roboflow.png`):
  - detected the three windows as `glass` at 55–59% confidence;
  - **missed the brick wall entirely**, the dominant material in the image;
  - the `glass` boxes included the boarded-up panels below each window pane.
- Diagnosis: recall of 18% means about 4 in 5 material regions are missed. The
  main cause is too little data, roughly 8 images per class.
- Decision: keep as a baseline only.

### Run 02 — YOLO26 Small (failed)

- Dataset: a **different** version (12:50pm) from Run 01. Training used only
  0.18 credits. Latency 5.5 ms per image. Model license: AGPL-3.0.
- On the same sample image (`run02_yolo26s_roboflow.png`): **no predictions
  at all**.
- Likely causes, most likely first:
  1. very short training (few epochs); YOLO needs more data and epochs than
     RF-DETR on a small dataset;
  2. a different dataset version, so not comparable with Run 01;
  3. possible Grayscale preprocessing (test thumbnails look black-and-white;
     **to confirm**), which removes colour, a key cue for materials.
- Decision: treat as failed; only compare runs trained on the same version.

### Targets set after these runs

| Level | mAP@50 | Recall | Precision |
|---|---|---|---|
| Assignment target (prototype) | ≥ 50% | ≥ 50% | ≥ 50% |
| Practical screening tool | ≥ 70% | ≥ 75% on `glass`, `cladding_panel`, `stone_cladding`, `exposed_concrete` | ≥ 65% |

Recall is weighted higher than precision because a missed material means a
value-engineering substitution could go unflagged.

---

## 7. `cladding_panel` images unusable — re-collection — 27 Sep 2026, 14:14

**Problem.** On review, the `cladding_panel` folder from the first collection
(entry 3) contained almost no buildings with cladding panels.

**Cause.** The first script used one plain keyword phrase
(`metal cladding facade building`). Commons keyword search matches any file
whose title or description mentions the words, so it returned unrelated
buildings, details and non-facade images.

**Action.** Wrote `notebooks/00b_collect_cladding.ipynb`, which:
- searches Commons categories first (for example "Aluminium facades",
  "Metal cladding", "Rainscreen cladding"), then targeted phrases such as
  "aluminium cladding", "composite panel facade", "perforated metal facade";
- excludes interiors, diagrams, drawings, maps and texture close-ups;
- keeps only CC0, public domain, CC BY and CC BY-SA images at least 800 px wide;
- shows a numbered preview grid so unsuitable images are deleted by hand;
- writes `sources.csv` (file, title, license, author, source page, query) for
  attribution.

**Result.** _TBD — record how many images were downloaded, how many were
deleted in the preview, and how many were uploaded to Roboflow._

**Reflection.** Keyword search is not enough for a visual class. Checking the
downloaded images by eye before annotation catches bad data early; this
should be done for every class, not only `cladding_panel`.

---

## 8. Dataset frozen and published — 27 Sep 2026, 16:52–17:05

**Action.** Finished annotation in Roboflow, generated a dataset version,
exported it in YOLOv8 format, and published the zip as a GitHub Release.

| Item | Value |
|---|---|
| Roboflow project | `omar-el-sayed-pzb69/facade-materials-3mrrt` |
| Roboflow version | **4** (generated 2026-09-27 4:52pm) |
| Release | tag `v1.0` |
| File | `facade-materials-v3-yolov8.zip` (37,025,500 bytes) |
| URL | https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/download/v1.0/facade-materials-v3-yolov8.zip |
| SHA256 | `134e41be8c310452bbe8ea33fd55a891c6e29ca863f3c9b8df3827f59bb951cd` |
| Preprocessing | Auto-orient, resize 640×640 (stretch). No Grayscale. |
| Augmentation | 2 versions per training image: 50% horizontal flip, rotation ±10°, brightness ±15% |
| Images | 430 in total: 354 train (177 source × 2), 51 valid, 25 test |
| Split of source images | 177 / 51 / 25 = **70 / 20 / 10** |

Checked on 27 Sep: the URL downloads without any login and the SHA256 above
matches the downloaded file.

**Images and boxes per class** (after the label cleanup described below):

| Class | Train images | Train boxes | Valid images | Valid boxes | Test images | Test boxes |
|---|---|---|---|---|---|---|
| brick | 142 | 1,985 | 16 | 323 | 8 | 88 |
| cladding_panel | 58 | 220 | 12 | 55 | 4 | 17 |
| exposed_concrete | 78 | 274 | 8 | 70 | 4 | 10 |
| glass | 188 | 8,089 | 29 | 1,369 | 13 | 364 |
| painted_render | 90 | 223 | 13 | 26 | 6 | 6 |
| stone_cladding | 87 | 2,979 | 16 | 344 | 6 | 279 |

**Problems found when the released zip was inspected**

| Problem | What was found | Consequence | Resolution |
|---|---|---|---|
| P5 | Roboflow's YOLOv8 export **kept SAM annotations as polygons**. 105 label files are polygon-only and **181 mix polygons with plain boxes** | In a mixed file the YOLOv8 detection loader reads every line as a polygon, so the plain boxes become corrupted | Training notebook converts every polygon to its tightest box before training (11,963 polygons converted) |
| P6 | **3,507 boxes under 4 px** wide or tall at 640 px, mostly on `glass` (1,342) and `stone_cladding` (1,816) | Fragments from SAM clicks, not real objects; they add noise | Dropped in the training notebook (the notebook counts 3,563, because it applies the size test before rounding coordinates) |
| P7 | `glass` and `stone_cladding` were annotated **per pane / per stone** (about 43 and 34 boxes per image) instead of one box per continuous area (rule D3) | Thousands of small objects; these classes behave very differently from the others | Not relabelled for lack of time. Recorded as a known inconsistency and a first item for the next dataset version |
| P8 | Strong imbalance: `glass` has 8,089 training boxes, `cladding_panel` 220 and `painted_render` 223 (only 26 validation boxes) | Metrics for the small classes are unstable | Recorded; top up small classes in the next version |
| P9 | The file is named `v3` but contains Roboflow **version 4** | Naming confusion only; the content is fixed by the checksum | Documented here and in the README |
| P10 | Split is 70/20/10, not the 80/20 in the brief | A test split exists in addition to validation | Kept: the training notebook uses the test split for an extra independent check |
| P11 | Roboflow shows the dataset license as **CC BY 4.0** | If any source image is CC BY-SA, the dataset must be CC BY-SA 4.0 | _To do: check `sources.csv`; change the license if needed_ |

**Correction.** Entry 5 assumed Roboflow's YOLOv8 export converts masks to
boxes. P5 shows it does not. The earlier Roboflow training runs (entry 6) may
also have been affected by mixed polygon/box labels.

**Reflection.** Opening the exported zip and counting the labels took a few
minutes and caught three problems that would have silently lowered the final
scores. Always inspect the exported data, not only the tool's interface.

---

## 9. Notebooks — 27 Sep 2026, 17:15–17:30

### Notebook 01 — baseline (`notebooks/01_baseline_inference.ipynb`)

**Question.** Does an off-the-shelf model already detect facade materials?

**Method.** YOLOv8s pretrained on COCO (80 object classes), run at 25%
confidence on one validation image each of `brick`, `glass`,
`cladding_panel` and `stone_cladding`, taken from the released dataset.

**Result** (run on 27 Sep; outputs in `results/baseline/`):

| True material | What the pretrained model detected |
|---|---|
| brick | bicycle (0.88) |
| glass | nothing |
| cladding_panel | nothing |
| stone_cladding | car (0.94), bird (0.25) |

None of the 6 material classes exists in COCO, so the model can only report
objects in front of the facade, never the surface itself.

**Reflection.** Same lesson as Session 1: a generic model can be technically
good and still not answer the business question. This justifies collecting
and labelling our own data. The pretrained weights are still used as the
starting point for training (transfer learning).

### Notebook 02 — training and evaluation (`notebooks/02_training_eval.ipynb`)

**Action.** Wrote the training notebook, following the course's
reproducibility standard.

- Downloads the dataset from the GitHub Release with the course's keyless cell
  and checks the SHA256.
- Cleans labels (P5, P6) on the extracted copy only; the released zip is
  unchanged.
- Trains `yolov8s.pt` with `epochs=50`, `imgsz=640`, `batch=16`, `seed=0`.
- Ultralytics pinned to **8.4.163**.
- Writes the metrics table (overall and per class, validation and test),
  curves, confusion matrices, 10 validation predictions, new-image predictions
  and `run_info.json` (versions, GPU, time) to `/content/results/`.
- `QUICK_RUN` option for a 5-epoch verification run when no GPU is available.

- Includes the optional Roboflow download cell from the course email: skipped
  automatically when the keyless dataset is present, asks for a key instead of
  crashing when none is set, and pinned to version 4.

**Check.** Both notebooks were run end to end in a CPU environment. Notebook
02 used 1 epoch and 10% of the training images, only to catch code errors;
all cells completed. (Those smoke-test metrics are meaningless and are not
reported.)

### Compliance with the course reproducibility standard (instructor email)

| # | Requirement | Status |
|---|---|---|
| 1 | Export the exact version trained on, file name shows the version | Done: Roboflow version 4. File is named `v3` (P9); format YOLOv8, which has the same label format as the YOLOv11 export named in the email |
| 2 | SHA256 checksum | Done: `134e41be…51cd` |
| 3 | Public repo, dataset as a Release asset, zip not committed | Done: tag `v1.0`; `*.zip` is in `.gitignore` |
| 4 | Primary keyless data cell used as given | Done in notebooks 01 and 02; only `DATA_URL` and `SHA256` changed |
| 5 | Optional Roboflow cell: skip on Run all, no crash without a key, pinned version | Done in notebook 02 (cell 3b), version 4 |
| 6 | README Dataset section: Universe link + version, classes and counts per split, preprocessing and augmentation, Release URL + SHA256, license | Done (README section 3); license label still to confirm (P11) |
| 7 | "Open in Colab" badge; test with "Disconnect and delete runtime" → Run all | Badge in both notebooks. _To do: badge in README and the clean-runtime test in Colab_ |
| — | No API key in any committed cell, output or git history | No key in the notebooks. _To confirm: no key was ever pushed_ |
| — | Only images you have the rights to share; no client or project photos | Sources are Wikimedia Commons (free licenses). _To do: license check P11_ |

---

## 10. Repository organisation, evidence and governance — 27 Sep 2026, 17:40

**Actions**
- Rewrote the root `README.md` in the order required by the brief: problem
  and success criteria, classes, dataset (all items from the instructor's
  point 6), reproduce steps with Colab badges, reproducibility checklist,
  results, reproducibility proof, evidence, governance, structure, license.
  Results that depend on Run 03 are marked _pending_.
- Rewrote the four folder READMEs (`data/`, `docs/`, `notebooks/`,
  `results/`); they still listed planned file names that were never used.
- Added `docs/governance_checklist.md`: privacy, data minimisation,
  limitations (screening aid, not certification), false-negative vs.
  false-positive risk, licensing.
- Replaced `LICENSE`: it was the unfilled Apache 2.0 template; now MIT
  (2026, Omar Elsayed) with a scope note for dataset images (source licenses)
  and weights (AGPL-3.0, because they are produced with Ultralytics).
- Added 5 annotation examples to `results/evidence/annotations/`, one per
  class except `brick` (shown in the baseline figure).
- Removed 8 `desktop.ini` files that Google Drive had put into the
  repository, and added `desktop.ini` and `*.pt` to `.gitignore`.

**Found while preparing the annotation examples** (added to
`error_analysis.md` as A4 and A5)
- `stone_cladding` example: outlines appear to follow the roofs, and the
  facade looks like render, which is a likely mislabel.
- A `painted_render` example is a black-and-white historic photo.

**Reflection.** Looking at the labels drawn on the images found two data
problems that the label counts alone did not show. Visual review should be a
standard step before every dataset version.

---

## 11. Label granularity: measured and a merge option added — 27 Sep 2026, 17:55–18:15

**Question.** Which data problem, if fixed, would most improve the model?

**Measurement** (254 source images, after polygon conversion and fragment
removal):

| Folder class | Images | Median share of image labelled | Images < 25% labelled | Images with no label of their own class | Median boxes per image |
|---|---|---|---|---|---|
| brick | 43 | 59% | 1 | 3 | 38 |
| cladding_panel | 34 | 68% | 3 | 0 | 10 |
| exposed_concrete | 44 | 78% | 9 | 4 | 3 |
| glass | 42 | 57% | 3 | 0 | 101 |
| painted_render | 39 | 39% | 13 | 14 | 6 |
| stone_cladding | 50 | 58% | 2 | 3 | 76 |

**Findings, largest first**
1. **Inconsistent granularity (A3).** `glass`, `stone_cladding` and `brick`
   were labelled per pane / per stone; the other classes per area. `glass`
   alone is about 60% of all training boxes. The model is effectively asked
   to learn two different tasks.
2. **Missing labels (A6).** 14 of the 39 `painted_render` images have no
   render label, and 31 images are less than a quarter labelled.
3. **Wrong labels (A4).** Some `stone_cladding` labels are on roofs or on
   surfaces that do not look like stone; some render boxes are nested inside
   other render boxes.

**Action.** Problem 1 can be fixed in code; 2 and 3 need relabelling. Added
to `02_training_eval.ipynb`:
- `MERGE_ELEMENTS` setting: same-class `glass`, `stone_cladding` and `brick`
  boxes that touch or are closer than 1.5% of the image width are merged into
  one box per connected area. Each merged box is the union of the original
  boxes, so it never extends beyond them. Training boxes: glass 8,075 → 1,047,
  stone 2,962 → 199, brick 1,980 → 382.
- Each run builds its own working copy (`/content/work/<RUN_NAME>/`) and
  writes to its own results folder, so two runs can be compared in one
  session. Cell 12 shows them side by side.
- **Validation and test labels are always merged**, so both runs are scored
  against the same answer key, the one that follows the class-definition
  rule. Without this, the two runs would be graded on different tasks and
  their scores could not be compared.
- Planned runs: Run 04 `yolov8s_merged` and Run 05 `yolov8s_asis`.
  **Superseded by entry 12:** the cleaning was moved out of the training
  notebook into a separate cleaned release, so these two runs were not made.

**Problem found and fixed while testing.** The first version of the merge
produced boxes with negative width for thin boxes touching the image edge
(the padding was clipped at the border, then removed again). Rewritten to
take each area's extent from its original boxes; checked on all training
labels: 0 invalid boxes.

**Check.** Both settings were run end to end on CPU (1 epoch, 10% of the
training images): all 13 code cells completed for both runs, each run used
its own training labels (2,341 vs. 13,730 boxes), and the comparison table
was produced. The two smoke-test runs gave identical scores because 5 batches
of 8 images are fewer than the 64 images YOLOv8 accumulates before its first
weight update; the real run (354 images, batch 16) updates every 4 batches.

**Reflection.** Measuring label coverage and boxes per image took minutes
and pointed to the most likely cause of weak results. Rules written in the
class definitions are only useful if the labels are checked against them.

---

## 12. Label cleaning: Release v1.1 — 27 Sep 2026, 19:50–20:05

**Request.** Fix the wrong and stacked labels on the images by removing most
of the small boxes, in the dataset itself rather than only at training time.

**Measured first** (253 source images, after polygon conversion, fragment
removal and per-area merging):
- 29% of boxes cover less than 0.5% of the image; 45% less than 1%.
- Boxes nested inside a larger box of the same class: 144 of 216
  `exposed_concrete` boxes, and 7–37 per other class.
- Two different classes on the same area (IoU > 0.7): 14 pairs in 13
  images (21 label files, counting augmented copies).

**Rules chosen** (`notebooks/01b_clean_dataset.ipynb`):

| Step | Rule | Why this setting |
|---|---|---|
| 1 | Polygons → boxes | A1 |
| 2 | Drop fragments under 4 px | A2 |
| 3 | Merge same-class boxes that overlap or are within 0.5% of the image width, **all classes** | One box per area (A3) and no nested duplicates (A7). A 1.5% gap was tried first and joined separate windows and neighbouring buildings; 0% and 0.5% were compared visually and 0.5% kept windows separate while still joining curtain-wall panes |
| 4 | Stacked classes (IoU > 0.7): keep the photo's own class, otherwise the larger box | A7 |
| 5 | Drop boxes under 1% of the image area (about 64 × 64 px) | Compared 0.5%, 1% and 2%: 905, 701 and 552 boxes kept. 1% removes most of the small boxes as requested; the removal never takes away an image's only label of its own material |
| 6 | Remove images with no label of their own material | 24 source images (14 `painted_render`, 4 `exposed_concrete`, 3 `brick`, 3 `stone_cladding`); their labels were already missing before cleaning (A6). Keeping them would teach the model that those walls are background |

**Result**

| | Images (train · valid · test) | Boxes (train · valid · test) |
|---|---|---|
| Release v1.0 | 354 · 51 · 25 | 13,730 · 2,179 · 756 |
| Release v1.1 | 322 · 45 · 23 | 968 · 149 · 78 |

- File `facade-materials-v4-clean-yolov8.zip`, 31,714,195 bytes, SHA256
  `bdcf62ff01fd58892ee22dd07cf2b481fa1194d0946dd5f78c6b41dc04187164`.
- Written deterministically; running the notebook twice gave the same SHA256.
- Report in `results/data_cleaning/`.

**Training notebook updated.** `02_training_eval.ipynb` now downloads
Release `v1.1`; cell 4 only checks the labels; the per-run merge option from
entry 11 is removed; cell 12 compares with Run 03. Tested end to end (1
epoch, 10% of images, CPU): all 13 code cells completed.

**Known limits**
- Merging and the 1% rule enforce the rules mechanically. Wrong labels
  (A4: roofs labelled as stone) and partly labelled images remain.
- The test split is now 23 images (3 for some classes): per-class test
  scores will be noisy.
- Roboflow still holds the uncleaned version 4; v1.1 exists only as a
  GitHub Release, documented and reproducible from v1.0.

**Reflection.** Measuring before changing made each threshold defensible,
and comparing gap sizes on real images caught an over-merging problem that
the counts alone would not have shown.

---

## 13. Next steps

- [ ] Run `01_baseline_inference.ipynb` in Colab so the outputs are saved in the notebook.
- [x] Run 03 (notebook v1) trained in Colab: validation mAP@50 0.143, recall 0.159 (README section 6).
- [ ] Publish Release `v1.1` with `facade-materials-v4-clean-yolov8.zip`.
- [ ] Run `02_training_eval.ipynb` in Colab on a T4 GPU (Run 04 on v1.1); fill in the README results table.
- [ ] Clean-runtime test: Runtime → Disconnect and delete runtime → Run all, from the Colab badge.
- [x] Write the README (entry 10). _Fill in the Run 04 results, takeaways and reproducibility proof after training._
- [ ] Add 5 new-image URLs (not in the dataset) to `NEW_IMAGE_URLS`.
- [ ] Attach `best.pt` to a GitHub Release and link it from the README.
- [ ] Complete the false-positive / false-negative tables in `error_analysis.md`.
- [ ] Check dataset license (P11) and update the Roboflow and Release notes.
- [ ] Review `stone_cladding` labels for roofs (A4); blur faces and plates in the next dataset version.
- [ ] Commit the source lists (`sources.txt`, `sources.csv`) to `data/` for attribution.
- [ ] Build the PDF pack (6–8 slides, 2-page report) once Run 04 results exist.
- [ ] Record the `cladding_panel` re-collection result in entry 7.
- [ ] Next dataset version: relabel in Roboflow to the class rules (A3), starting with the 24 removed images (A6) and the `stone_cladding` roofs (A4); more `cladding_panel` and `painted_render` images (P8).
