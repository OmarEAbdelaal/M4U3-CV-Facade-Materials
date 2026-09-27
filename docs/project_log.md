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
| D4 | SAM masks allowed as an annotation shortcut | Roboflow converts masks to boxes on YOLOv8 export |
| D5 | Keep the git repository **out of live Google Drive sync** where possible | Drive wrote `desktop.ini` files into `.git` and corrupted it |
| D6 | Re-collect `cladding_panel` with a targeted search | The first batch contained almost no real cladding facades |
| D7 | Final model is **YOLOv8s trained in Colab** | Brief requires a reproducible notebook; Roboflow-hosted runs are exploration only |

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
others with hand-drawn boxes. On YOLOv8 export Roboflow converts every mask to
its tightest box, so no relabelling was needed. Hand-drawn boxes can be looser
than mask-derived ones, so a sample was flagged for re-checking.

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
| 03 | YOLOv8s (Colab) | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ | Final run for the assignment |

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

## 8. Next steps

- [ ] Finish the `cladding_panel` re-collection and fill in the result in entry 7.
- [ ] Top up weak classes to 40–60 images each, starting with `brick`.
- [ ] Check class balance in Roboflow Health Check.
- [ ] Generate a new dataset version: no Grayscale, resize 640, flip, ±15% brightness, small rotation.
- [ ] Export as YOLOv8, compute SHA256, publish as a GitHub Release.
- [ ] Train YOLOv8s in Colab (`epochs=50`, `imgsz=640`, `batch=16`) and fill in Run 03.
- [ ] Complete the false-positive / false-negative tables in `error_analysis.md`.
