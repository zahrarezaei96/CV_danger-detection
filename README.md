# Autonomous Driving Danger Detection System
### Multi-Camera Perception | nuScenes Dataset

A real-time danger detection pipeline for autonomous vehicles using 6-camera surround perception. Detects, tracks, and assesses risk from surrounding objects, producing annotated video output with Bird's-Eye View (BEV) map.

---

## Models & Components

| Module | Model | Paper | Year |
|--------|-------|-------|------|
| Object Detection | **YOLO11x** | Ultralytics | 2024 |
| Depth Estimation | **Depth Anything V2 Metric Outdoor Large** | Yang et al., NeurIPS | 2024 |
| Multi-Object Tracking | **StrongSORT** (via BoxMOT) | Du et al., IEEE TMM | 2023 |
| Optical Flow | **RAFT** / Farneback fallback | Teed & Deng, ECCV | 2020 |
| Risk Scoring | TTC + Multi-factor + Ego dynamics | — | — |
| Cross-Camera Fusion | Edge-aware merge + World NMS | — | — |
| Dataset | **nuScenes mini** (v1.0-mini) | Caesar et al. | 2020 |

---

## Results Summary

All metrics evaluated on nuScenes mini, 20 frames, 6 cameras. Best results at 7m matching threshold.

| Version | Precision | Recall | F1 | Depth MAE |
|---------|-----------|--------|----|-----------|
| V1 — Baseline | 7.14% | 12.03% | 8.96% | 3.40m |
| V2 — All-camera tracking | 26.3% | 21.5% | 23.6% | 3.83m |
| V3 — Reduced FP | 27.4% | 21.2% | 23.9% | 3.80m |
| V4 — Multi-scale + Depth calibration | 55.9% | 45.3% | 50.0% | 3.37m |
| V5 — Smart fusion | 59.9% | 45.9% | 52.0% | 3.30m |
| **V5.1 — Visual BEV (best)** | **59.9%** | **45.9%** | **52.0%** | **3.30m** |

### Per-class Recall @ 7m (V5.1)

| Class | Recall |
|-------|--------|
| car | 81% |
| truck | 80% |
| person | 39% |
| motorcycle | 7% |
| bicycle | 0% |

---

## Repository Structure

```
CV_danger-detection/
├── danger_detection_v1.ipynb   # Baseline pipeline
├── danger_detection_v2.ipynb   # All-camera tracking + cross-camera fusion
├── danger_detection_v3.ipynb   # Reduced FP, conf=0.25, bbox area filter
├── danger_detection_v4.ipynb   # Multi-scale YOLO + depth calibration
├── danger_detection_v5.ipynb   # Smart edge fusion + dual-pass person detection
├── danger_detection_v5_1.ipynb # BEV heading arrows + speed display
└── README.md
```

---

## Version History

### V1 — Baseline
- First end-to-end pipeline: YOLO11x + Depth Anything V2 + StrongSORT
- Tracking on front 3 cameras only
- Simple world-space NMS (fixed 3.0m threshold)
- Risk scoring: depth(50%) + speed(30%) + approach angle(20%)

### V2 — All-Camera Tracking + Smart Fusion
- **Fix:** COCO↔nuScenes category mapping (adult→person)
- **Fix:** StrongSORT tracking extended to all 6 cameras
- **New:** Cross-camera edge-aware fusion (EDGE_MARGIN=30px)
- **New:** Per-class NMS thresholds + ego speed in risk scoring
- **New:** Multiple evaluation thresholds (3m, 5m, 7m)

### V3 — Reduced False Positives
- Confidence threshold raised to 0.25 (uniform)
- Minimum bbox area filter (400px²) to remove noise detections
- BEV object orientation corrected

### V4 — Multi-Scale + Depth Calibration *(biggest jump: +26 F1)*
- **New:** Multi-scale YOLO inference (640 + 1280px)
- **New:** Per-camera depth calibration using nuScenes LiDAR GT
- **New:** Per-class confidence thresholds (person=0.15, vehicles=0.25)
- **New:** Aggressive cross-camera fusion (EDGE_MARGIN=60px)
- **New:** Depth-aware NMS (threshold scales with distance)

### V5 — Smart Fusion Refinement
- EDGE_MARGIN reduced 60→40px (V4 was too aggressive)
- Smart edge detection: aspect-ratio check to confirm truncation
- Dual-pass person detection: Pass 2 is person-only at 1280px, conf=0.10
- SAHI removed (was conflicting with fusion logic)

### V5.1 — Visual BEV Enhancement *(metrics unchanged from V5)*
- Heading estimation from optical flow → world velocity
- BEV heading arrows per object
- BEV vehicle rotation along estimated heading
- Speed display in km/h on BEV
- Visual deduplication within 15px on BEV canvas

---

## Setup & Usage

### Requirements
```bash
pip install "numpy<2.0"
pip install nuscenes-devkit==1.2.0
pip install ultralytics==8.3.40
pip install pyquaternion==0.9.9
pip install "imageio[ffmpeg]" tqdm
pip install "transformers>=4.50.0"
pip install boxmot
git clone https://github.com/princeton-vl/RAFT.git /content/RAFT
```

### Dataset
1. Download [nuScenes mini](https://www.nuscenes.org/nuscenes#download) (`v1.0-mini.tgz`)
2. Download [nuScenes map expansion](https://www.nuscenes.org/nuscenes#download) (`nuScenes-map-expansion-v1.3.zip`)
3. Place both in `MyDrive/datasets/nuscenes/` on Google Drive

### Running (Google Colab)
Open any notebook and run cells sequentially. Cell 1 installs dependencies — restart runtime after Cell 1, then continue from Cell 2.

> **Note:** After Cell 1, go to **Runtime → Restart Runtime**, then run from Cell 2.

---

## Hardware

Tested on Google Colab (T4 GPU). Processing ~20 frames takes approximately 10–15 minutes depending on the version.

---

## References

- **YOLO11x:** Ultralytics (2024). https://github.com/ultralytics/ultralytics
- **Depth Anything V2:** Yang et al., NeurIPS 2024. https://depth-anything-v2.github.io
- **StrongSORT:** Du et al., IEEE TMM 2023. https://github.com/mikel-brostrom/boxmot
- **RAFT:** Teed & Deng, ECCV 2020. https://github.com/princeton-vl/RAFT
- **nuScenes:** Caesar et al., CVPR 2020. https://www.nuscenes.org
