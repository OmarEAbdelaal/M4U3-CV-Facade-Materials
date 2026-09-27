# Data

The dataset is **not** stored in this repository. The frozen versions are
GitHub Release assets:

| Release | File | SHA256 | Use |
|---|---|---|---|
| [`v1.0`](https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/tag/v1.0) | `facade-materials-v3-yolov8.zip` (Roboflow `facade-materials-3mrrt` version 4, as exported) | `134e41be8c310452bbe8ea33fd55a891c6e29ca863f3c9b8df3827f59bb951cd` | Source; Run 03 |
| [`v1.1`](https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/tag/v1.1) | `facade-materials-v4-clean-yolov8.zip` (v1.0 after `notebooks/01b_clean_dataset.ipynb`) | `bdcf62ff01fd58892ee22dd07cf2b481fa1194d0946dd5f78c6b41dc04187164` | Training from Run 04 on |

The notebooks download a release by URL, check the SHA256 and extract it.
No Roboflow account or API key is needed. Full dataset details are in the
root README, section 3.

This folder is for small reference files only: the per-image source and
license lists from the collection notebooks (`sources.txt`, `sources.csv`),
used for attribution. Raw images, zips and label files are excluded by
`.gitignore`.
