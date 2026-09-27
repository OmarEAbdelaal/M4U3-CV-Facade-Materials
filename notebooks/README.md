# Notebooks

All notebooks run in Google Colab with no accounts, API keys or local
installs. Use the "Open in Colab" badges in the root README.

| Notebook | Purpose | Needed to reproduce results? |
|---|---|---|
| `00_collect_images.ipynb` | First image collection from Wikimedia Commons, about 50 per class, with source and license log | No (documents where images came from) |
| `00b_collect_cladding.ipynb` | Targeted re-collection for `cladding_panel` after the first batch was unusable; license filter, preview and cull, `sources.csv` | No (documents where images came from) |
| `01_baseline_inference.ipynb` | Keyless dataset download; pretrained COCO YOLOv8s on 4 facade photos; shows it has no material classes | Yes (baseline) |
| `02_training_eval.ipynb` | Keyless dataset download with SHA256 check; label cleanup; YOLOv8s training; metrics; curves; validation and new-image predictions; results package | Yes (main result) |

Notebook 02 needs a GPU (Runtime → Change runtime type → T4 GPU). Without
one, set `QUICK_RUN = True` in cell 2.
