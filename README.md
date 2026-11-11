# AI Flood Monitoring & Risk Prediction System

**Developer: Pragati Sarang**  
[GitHub](https://github.com/pragatisarang36-design)

A computer vision-based flood monitoring system designed to analyze video footage, identify water-like regions, estimate water coverage, monitor potential flood zones, and assess flood risk using image processing and object detection techniques.

> This repository documents the flood monitoring application only.

## Project Overview

The application processes a video stream frame by frame. It converts each frame to HSV colour space and combines three colour masks to find blue, dark or reflective, and green-murky water-like regions. Closing and opening morphology, Gaussian blur, and thresholding refine the mask before the program measures coverage and finds external contours.

For each frame, the system calculates total water pixels and the percentage of the frame classified as water. It also measures coverage in the bottom third of the frame. The script accumulates segmented water pixels over time into a map and overlays a colour heatmap to visualize repeated detections.

When YOLOv8 loads successfully, the application runs object detection every third frame and filters detections below 0.35 confidence. It checks whether each remaining bounding-box centre falls inside the water mask. Overlapping detections are counted and highlighted as potential object exposure; the model does not determine damage. If YOLOv8 cannot be loaded, video processing continues without object detections.

The risk score is a heuristic calculated as `min(100, coverage × 1.2 + lower-third coverage × 0.8)`. The displayed category is assigned from coverage thresholds:

| Category | Rule |
| --- | --- |
| **CRITICAL** | Coverage > 60%, or lower-third coverage > 75% |
| **DANGER** | Coverage > 35%, or lower-third coverage > 50% |
| **WARNING** | Coverage > 15%, or lower-third coverage > 25% |
| **SAFE** | None of the thresholds above are met |

The output video includes water overlays and contour outlines, the accumulation heatmap, object boxes when available, and a monitoring HUD.

## Key Features

### Water Detection

- HSV colour segmentation for blue, dark/reflective, and green-murky water-like colours.
- Morphological closing and opening, Gaussian smoothing, thresholding, and contour detection to refine and outline regions.
- Semi-transparent highlighting of pixels classified as water.

### Flood Analysis

- Per-frame water pixel count and water coverage percentage.
- Coverage analysis of the bottom third of the frame.
- Accumulation heatmap built from detections across processed frames.
- Active water-zone count based on external contours in the current frame.

### Object and Infrastructure Monitoring

- Optional YOLOv8 object detection, run every third frame when the model loads.
- Detection confidence threshold of 0.35.
- Bounding-box centre check against the water mask; overlapping detections are counted and highlighted as potential exposure.

### Risk Assessment

- A 0–100 heuristic risk score based on water coverage and bottom-third coverage.
- Four visual categories: **SAFE**, **WARNING**, **DANGER**, and **CRITICAL**, assigned using the thresholds in the overview.
- On-screen visual alert badge using the current category.

### Monitoring Dashboard

The video HUD displays water coverage, water pixel count, an approximate depth estimate, risk score and category, active zone count, object-overlap count, processing FPS, frame progress, estimated remaining time, and CPU/GPU device information.

The depth estimate is computed as `coverage percentage / 20 × 2.5`. It is only a rough display heuristic and is not a physical depth measurement.

## Technologies and Dependencies

The script checks for and attempts to install these Python packages at startup:

| Package / technology | Role in the implementation |
| --- | --- |
| Python | Application runtime and dependency installer |
| OpenCV (`opencv-python`) | Video input/output, image processing, masks, contours, overlays, and HUD |
| NumPy | Masks, pixel measurements, and accumulation map |
| PyTorch (`torch`, `torchvision`) | CUDA availability and YOLO model inference support |
| YOLOv8 (Ultralytics) | Object detection when the model loads successfully |
| SciPy (`scipy`) | Listed in the script's startup dependency checks |
| Pillow (`pillow`) | Listed in the script's startup dependency checks |
| Matplotlib (`matplotlib`) | Listed in the script's startup dependency checks |
| Supervision (`supervision`) | Listed in the script's startup dependency checks |
| MediaPipe (`mediapipe`) | Listed in the script's startup dependency checks |

The current implementation does not import SciPy, Pillow, Matplotlib, Supervision, or MediaPipe for its processing pipeline; the script only checks for and installs them. YOLOv8 is used when available and successfully loaded. A CUDA-capable GPU is used when PyTorch detects one; otherwise processing uses the CPU.

## Project Structure

```text
Computer-Vision-Systems/
├── Flood & risk check/
│   └── FLOOD MONITORING.py
└── README.md
```

This repository contains the flood monitoring application and its supporting documentation.

## Installation and Execution

### Prerequisites

- Windows with Python installed and available as `python` in PowerShell or Command Prompt.
- A webcam for live capture, or a video file for analysis.
- Internet access on the first run so the script can install missing listed Python packages. Ultralytics may also download the `yolov8n.pt` model when it first loads.

### Clone the repository

Replace the URL below with the HTTPS URL of this repository on GitHub:

```powershell
git clone <repository-url>
cd Computer-Vision-Systems
cd "Flood & risk check"
```

### Run the application

From the application directory, pass a video file path as the first argument:

```powershell
python "FLOOD MONITORING.py" video.mp4
```

To use the default webcam source (device `0`), omit the argument:

```powershell
python "FLOOD MONITORING.py"
```

The script checks for and automatically installs its listed Python dependencies when they are missing. It writes the processed video to `output_flood_monitoring_ai.mp4` in the current working directory. The output uses the input video's dimensions and frame rate.

## Important Technical Limitations

- Water detection relies on HSV colour ranges. Other blue, dark, reflective, or green areas may be classified as water, and water with different colours may be missed.
- The displayed depth estimate is a rough heuristic derived from image coverage, not a physical water-depth measurement.
- Risk scoring and categories are rule-based heuristics. The system does not use a trained flood forecasting model.
- YOLOv8 detects objects; a box centre falling in a segmented region does not establish infrastructure damage or even confirm that the object is infrastructure.
- The system does not use live weather, rainfall, river-level, geographic, or other sensor data.
- Results depend on the camera view, lighting, video quality, and object detector availability. They are not scientifically validated flood predictions.

## Disclaimer

This application is an experimental computer vision monitoring tool intended for educational and research use. Its visual detections and heuristic risk indicators are not a substitute for official flood warnings, emergency response systems, or assessments by qualified hydrology professionals. Follow local authorities and emergency services for decisions involving public safety.

