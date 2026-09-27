# Notebooks

This folder holds the Colab-ready notebooks for this project. Each one is
self-contained: opening it via the "Open in Colab" badge in the root README
and choosing Runtime → Run all should reproduce the results in `/results/`
with no accounts, API keys, or local installs.

Notebooks:

- `00_collect_images.ipynb` — one-off data collection: pulls candidate
  photos per class from Wikimedia Commons and writes `sources.csv`
  (license + attribution per image). Not needed to reproduce results;
  kept here to document where the raw images came from.
- `01_baseline.ipynb` — environment setup, keyless dataset download
  (from a GitHub Release), a quick look at the data, and baseline inference
  with a pretrained YOLO model (sanity check before training).
- `02_train_eval.ipynb` — full training run, validation metrics, curves,
  and inference on both validation images and new/unseen images.

`01` and `02` are not added yet — coming in a later step.
