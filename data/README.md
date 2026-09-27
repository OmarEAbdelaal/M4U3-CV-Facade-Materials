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

## Files in this folder

| File | Contents |
|---|---|
| [`attribution.csv`](attribution.csv) | **One row per source image in the released dataset** (253 in v1.0; 229 kept in v1.1): class folder, whether it is in v1.1, license, author, Wikimedia Commons page |
| [`sources_collection_01.csv`](sources_collection_01.csv) | Full log of the first collection (`notebooks/00_collect_images.ipynb`): 300 images, 50 per class. Its `cladding_panel` images were replaced (project log, entry 7) |
| [`sources_cladding_recollection.csv`](sources_cladding_recollection.csv) | Log of the `cladding_panel` re-collection (`notebooks/00b_collect_cladding.ipynb`): 48 images kept after the preview check, with the search query used |
| [`new_images.csv`](new_images.csv) | The 5 images **not in the dataset** used for the new-image predictions in notebook 02 (license, author, source page) |

## License

184 of the 253 released source images are CC BY-SA (2.0, 3.0 or 4.0); the
rest are CC0, public domain or CC BY. The dataset as a whole is therefore
released as **CC BY-SA 4.0**, and each image keeps its own license and
attribution as listed in `attribution.csv`.

| License | Images (v1.0) |
|---|---|
| CC BY-SA 4.0 | 97 |
| CC BY-SA 2.0 | 69 |
| Public domain | 29 |
| CC0 | 26 |
| CC BY-SA 3.0 | 18 |
| CC BY 4.0 | 7 |
| CC BY 2.0 | 5 |
| CC BY 3.0 | 2 |

Raw images, zips and label files are excluded by `.gitignore`.
