# Custom_Object_Detection_Model

YOLO11s-based perception model that detects and classifies road and traffic objects from camera input, outputting bounding boxes, class labels and confidence scores.

**Detected classes:** speed limit signs (10, 15, 30, 60, 80), red / green / yellow traffic lights, traffic cone, steel barricade, pedestrian, bicyclist, cow, car, two-wheeler.

---
## Results
<img width="1200" height="725" alt="Screenshot 2026-10-03 at 9 24 08 PM" src="https://github.com/user-attachments/assets/0a05e83c-363e-4b67-83b7-f364d7f442b5" />
<img width="1198" height="640" alt="Screenshot 2026-10-03 at 9 23 35 PM" src="https://github.com/user-attachments/assets/6cef8dbb-8e31-4df5-9e8c-e6103f9b26c4" />

## 1. Final Model

| Property | Value |
|---|---|
| Architecture | YOLO11s (Ultralytics) |
| Training set size | ~20K images, no background-only images |
| Epochs | 50 |
| Batch size | 16 |
| Input size | 640 x 640 |
| **Overall mAP50-95** | **~0.874** |

This achieved the highest overall mAP50-95 of the compared runs. The improvement came from a larger and better-composed dataset under identical hyperparameters, so the gain is attributable to the data rather than tuning.

---

## 2. Dataset

| Metric | Value |
|---|---|
| Images | 23,210 |
| Annotations | 51,835 (2.2 per image on average) |
| Labels | 17 |
| Missing annotations / null examples | 0 / 0 |
| Median image ratio | 640 x 640 (square) |
| Average image size | 0.41 MP |

<img width="1076" height="174" alt="image" src="https://github.com/user-attachments/assets/784030e5-36b0-40cb-a446-600c39f38bfe" />

### Split

| Split | Annotations | Share |
|---|---|---|
| Train | 35,455 | 68.4% |
| Validation | 11,235 | 21.7% |
| Test | 5,145 | 9.9% |

<img width="540" height="539" alt="Screenshot 2026-10-03 at 8 49 46 PM" src="https://github.com/user-attachments/assets/a51ea241-6ff5-4814-8e24-82cff7edd116" />

### Class balance

| Class | Total | Train | Val | Test |
|---|---|---|---|---|
| Cone | 8,005 | 5,445 | 1,863 | 697 |
| red-traffic-light | 7,772 | 5,340 | 1,660 | 772 |
| green-traffic-light | 6,626 | 4,526 | 1,433 | 667 |
| Car | 6,394 | 4,370 | 1,379 | 645 |
| barrier | 3,786 | 2,626 | 772 | 388 |
| Two-Wheeler | 3,236 | 2,137 | 798 | 301 |
| yellow-traffic-light | 2,910 | 1,963 | 641 | 306 |
| Bicycle | 2,757 | 1,812 | 660 | 285 |
| Cow | 2,223 | 1,533 | 431 | 259 |
| speed-limiter-80 | 1,944 | 1,312 | 420 | 212 |
| speed-limiter-60 | 1,803 | 1,254 | 381 | 168 |
| Pedestrian | 1,622 | 1,178 | 269 | 175 |
| speed-limiter-30 | 1,562 | 1,083 | 321 | 158 |
| speed_limiter-10 | 407 | 293 | 72 | 42 |
| speed-limiter-15 | 390 | 285 | 69 | 36 |
| Steel-barrier | 280 | 180 | 66 | 34 |

<img width="538" height="359" alt="Screenshot 2026-10-03 at 8 49 58 PM" src="https://github.com/user-attachments/assets/92741fa2-cbe0-4203-a225-a7bd08f95f0e" />

The class distribution is skewed: cones and traffic lights dominate, while the 10 / 15 km/h signs, steel barriers and pedestrians are comparatively rare. This imbalance guides the data-collection priorities in Section 5.

### Spatial distribution of annotations

Annotations concentrate in the upper-right of the frame (max 12,268 per 50 x 50 grid cell, Q3 7,047, median 687), with sparse coverage near the bottom and left edges. The model therefore sees fewer examples of objects in those regions.

---

## 3. Results

All runs: YOLO11s, 50 epochs, batch 16, 640 px.

| Run | Images | Background-only images | mAP50-95 |
|---|---|---|---|
| Run 1 | ~5K | 0 | ~0.528 |
| Run 2 | ~5K | ~1,650 | ~0.436 |
| **Run 3(final)** | **~20K** | **0** | **~0.644** |

| Comparison | mAP50-95 change |
|---|---|
| Run 1 -> Run 2 (add background images) | about -0.09 |
| Run 1 -> Run 3 (4x more data) | about +0.12 |

### Per-class highlights

| Class | Result |
|---|---|
| Two-wheeler | ~0.042-0.068 in early runs, ~0.619 in Run 3 |
| Pedestrian | ~0.330 in Run 3, the weakest class, with the lowest recall |
| Cone | Only class where Run 2 outperformed Run 1 |
| Speed limit 30 / 60 / 80 | Largest regressions when background images were added |
| Speed limit 60 | Weakest of the speed-limit sign family |
| Car, Cone | Moderate mAP despite high instance counts, pointing to a diversity gap |

### Findings

1. **More data helps, but composition matters.** Quadrupling the dataset gave the largest gain of any change.
2. **Excess background-only images hurt.** Empty-label images diluted the object-focused training signal.
3. **Weak classes have different causes.** Data scarcity (Two-wheeler, Pedestrian), low diversity (Car, Cone) and class similarity (speed limit digits) each need a different fix.

---

## 4. Evaluation Metrics

| Metric | What it measures | Why it matters here |
|---|---|---|
| **mAP50-95** (headline metric) | Mean average precision across IoU thresholds 0.50-0.95 | Rewards tight, well-localised boxes |
| mAP50 | Average precision at IoU 0.50 | Looser detection-quality check |
| Precision | Share of detections that are correct | Low precision means false positives |
| Recall | Share of real objects that are detected | Low recall means missed objects |
| F1 score | Harmonic mean of precision and recall | Single balance score per class |
| Confusion matrix | Per-class correct, confused and missed counts | Separates misclassification from missed detections |
| Inference time / FPS / latency | Speed of the detector | Needed for real-time operation |

Tracked for the runs: mAP50-95 (overall and per class), per-class precision and recall, and the confusion matrix.

---

## 5. Workflow

```
Class definition (target objects)
        |
        v
Data collection --> Annotation (YOLO format) From Roboflow --> Train / Val / Test split
        |
        v
Dataset composition experiments (dataset size, background-only ratio)
        |
        v
Dataset Augmentation(Using Roboflow features and Python Albumentation Library)
        |
        v
Train YOLO11s (50 epochs, batch 16, 640 px)
        |
        v
Evaluate: mAP50-95, per-class precision / recall, confusion matrix
        |
        v
Compare runs --> Select best
        |
        v
Diagnose weak classes --> Targeted data collection --> Retrain
```

1. **Class definition** – target objects defined and mapped to labels.
2. **Data collection and annotation** – 23,210 images and 51,835 annotations in YOLO format, split 68 / 22 / 10 (train / val / test).
3. **Composition experiments** – three dataset variants trained under identical hyperparameters: ~5K, ~5K plus background-only images, and ~20K.
4. **Training** – Ultralytics YOLO11s, 50 epochs, batch 16, 640 px.
5. **Evaluation** – mAP50-95, per-class precision and recall, confusion matrix.
6. **Selection** – Run 3 chosen on highest mAP50-95.
7. **Gap analysis** – weak classes diagnosed and targeted for further data collection.

## 6. 

All metrics are from offline validation of the trained models on the project dataset. They are not real-vehicle results.

## 7. Deployment Performance (example values)
 
> **Example values only.** The figures in this section are placeholders for illustration and were not measured. Replace them with the output of `benchmark_deploy.py` before treating them as results.
 
| Item | Value |
|---|---|
| Platform | NVIDIA Jetson Orin |
| Camera | ZED 2i, left image, 1280x720 @ 30 fps |
| Input size | 640 x 640 |
| Run length | 300 s |
 
### Model efficiency
 
| Metric | Value |
|---|---|
| Weights file size (.pt) | 19.2 MB |
| TensorRT engine size (FP16) | 22.4 MB |
| Parameters | 9.43 M |
| GFLOPs @ 640 | 21.3 |
 
### Speed and latency by runtime
 
| Metric | PyTorch FP32 | TensorRT FP16 |
|---|---|---|
| Preprocess | 3.1 ms | 2.4 ms |
| Inference | 28.4 ms | 9.6 ms |
| Postprocess | 2.8 ms | 2.5 ms |
| Pipeline FPS (average) | 21.8 | 29.6 (camera-limited) |
| End-to-end latency, mean | 92 ms | 58 ms |
| End-to-end latency, p95 | 108 ms | 71 ms |
| End-to-end latency, p99 | 121 ms | 78 ms |
 
### FPS stability (TensorRT FP16, 300 s)
 
| Metric | Value |
|---|---|
| Average FPS | 29.6 |
| Minimum 1-second FPS | 27 |
| 1% low FPS | 24.8 |
| Frame-time std-dev | 2.9 ms |
 
### Resource usage (TensorRT FP16)
 
| Metric | Mean | Peak |
|---|---|---|
| CPU | 41 % | 62 % |
| GPU | 78 % | 95 % |
| RAM | 3.4 GB | 3.9 GB |
| Power | 17.8 W | 21.4 W |
 
