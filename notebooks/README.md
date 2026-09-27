# Notebooks

All notebooks run in Google Colab with no accounts, API keys or local
installs. Use the "Open in Colab" badges in the root README.

| Notebook | Purpose | Needed to reproduce results? |
|---|---|---|
| `00_collect_images.ipynb` | First image collection from Wikimedia Commons, about 50 per class, with source and license log | No (documents where images came from) |
| `00b_collect_cladding.ipynb` | Targeted re-collection for `cladding_panel` after the first batch was unusable; license filter, preview and cull, `sources.csv` | No (documents where images came from) |
| `01_baseline_inference.ipynb` | Keyless download of Release v1.0; pretrained COCO YOLOv8s on 4 facade photos; shows it has no material classes | Yes (baseline) |
| `01b_clean_dataset.ipynb` | Label cleaning: Release v1.0 → v1.1 (polygons to boxes, fragments, one box per area, stacked labels, small boxes, images missing their own class). Writes the zip deterministically and prints its SHA256 | Yes (to rebuild v1.1); no GPU needed |
| `02_training_eval.ipynb` | Keyless download of Release v1.1 with SHA256 check; label check; YOLOv8s training; metrics; curves; validation and new-image predictions; results package; comparison with Run 03 | Yes (main result) |

Notebook 02 needs a GPU (Runtime → Change runtime type → T4 GPU). Without
one, set `QUICK_RUN = True` in cell 2.
