# Results

Populated by `notebooks/02_train_eval.ipynb`. Expected contents once a
training run completes:

- `metrics/` — precision, recall, mAP50, mAP50-95 curves and the raw
  `results.csv` from training
- `confusion_matrix.png`
- `predictions/validation/` — sample predictions on held-out validation
  images
- `predictions/new_images/` — sample predictions on images the model
  never saw during training or validation
- `error_analysis/` — failure-case crops referenced in
  `docs/03_error_analysis.md`

Not added yet — coming after the first training run.
