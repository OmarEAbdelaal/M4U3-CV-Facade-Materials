# Results

Outputs of the notebooks. Notebook 02 writes to
`/content/results/yolov8s_clean/` in Colab; unzip
`results_yolov8s_clean.zip` into a folder of the same name here after a run.

| Path | Contents | Produced by |
|---|---|---|
| `baseline/` | Pretrained COCO model on 4 facade photos: comparison figure and detections table | Notebook 01 |
| `data_cleaning/` | Boxes per class before/after, removed images (for relabelling), stacked labels resolved, before/after figure, SHA256 of Release v1.1 | Notebook 01b |
| `yolov8s_clean/` | Run 04: YOLOv8s trained on Release v1.1 | Notebook 02 |
| `comparison.csv` | Run 04 next to Run 03 (overall precision, recall, mAP50, mAP50-95) | Notebook 02, cell 12 |
| `evidence/annotations/` | 5 annotation examples from Release v1.0, one per class (boxes and SAM polygons) | Drawn from the released dataset |
| `evidence/training_runs/` | Screenshots of the exploration runs on Roboflow (runs 01–02) | Manual |

Inside `yolov8s_clean/`:

| Path | Contents |
|---|---|
| `curves/` | `results.png` (loss and metric curves), `results.csv`, PR / F1 / P / R curves, confusion matrices for validation and test |
| `metrics.csv` | Precision, recall, mAP50, mAP50-95: overall and per class, validation and test |
| `dataset_counts.csv` | Images and boxes per class and split |
| `run_info.json` | Date, library versions, GPU, settings, training time |
| `evidence/val_predictions/` | 10 validation predictions (fixed seed) |
| `evidence/new_predictions/` | 5 predictions on images not in the dataset |

Model weights (`best.pt`) are not committed; they are attached to a GitHub
Release.
