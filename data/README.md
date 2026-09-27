# Data

The dataset is **not** stored in this repository. The frozen version that
all results depend on is a GitHub Release asset:

| Item | Value |
|---|---|
| Release | [`v1.0`](https://github.com/OmarEAbdelaal/M4U3-CV-Facade-Materials/releases/tag/v1.0) |
| File | `facade-materials-v3-yolov8.zip` (content: Roboflow `facade-materials-3mrrt` version 4) |
| SHA256 | `134e41be8c310452bbe8ea33fd55a891c6e29ca863f3c9b8df3827f59bb951cd` |

The notebooks download it by URL, check the SHA256 and extract it. No
Roboflow account or API key is needed. Full dataset details are in the root
README, section 3.

This folder is for small reference files only: the per-image source and
license lists from the collection notebooks (`sources.txt`, `sources.csv`),
used for attribution. Raw images, zips and label files are excluded by
`.gitignore`.
