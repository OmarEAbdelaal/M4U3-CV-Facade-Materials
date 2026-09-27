# Results

Outputs of the notebooks. Notebook 02 writes to `/content/results/` in
Colab with the same layout; copy the contents of `results_package.zip`
here after a run.

| Path | Contents | Produced by |
|---|---|---|
| `baseline/` | Pretrained COCO model on 4 facade photos: comparison figure and detections table | Notebook 01 |
| `curves/` | `results.png` (loss and metric curves), `results.csv`, PR / F1 / P / R curves, confusion matrices for validation and test | Notebook 02 |
| `metrics.csv` | Precision, recall, mAP50, mAP50-95: overall and per class, validation and test | Notebook 02 |
| `dataset_counts.csv` | Images and boxes per class and split after label cleanup | Notebook 02 |
| `run_info.json` | Date, library versions, GPU, settings, training time | Notebook 02 |
| `evidence/annotations/` | 5 annotation examples, one per class (boxes and SAM polygons) | Drawn from the released dataset |
| `evidence/val_predictions/` | 10 validation predictions | Notebook 02 |
| `evidence/new_predictions/` | 5 predictions on images not in the dataset | Notebook 02 |
| `evidence/training_runs/` | Screenshots of the exploration runs on Roboflow (runs 01–02) | Manual |

Model weights (`best.pt`) are not committed; they are attached to a GitHub
Release.
