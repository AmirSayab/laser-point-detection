# Laser Point Detection

> An interpretable video pipeline for detecting transient laser contacts using classical computer vision, with formal evaluation and a 64-test suite.

![Python](https://img.shields.io/badge/Python-3.10+-blue) ![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green) ![Pytest](https://img.shields.io/badge/Tests-64_passing-brightgreen) ![License](https://img.shields.io/badge/License-Private-gray)

## Architecture

```
┌──────────────┐     ┌──────────────────────────────────────────┐
│  Dahua IPC   │────▶│           DETECTION PIPELINE              │
│  (.dav video)│     │                                           │
└──────────────┘     │  ┌────────────────────────────────────┐  │
                     │  │  Background model (exp α=0.01)     │  │
                     │  ├────────────────────────────────────┤  │
                     │  │  Brightness filter                 │  │
                     │  ├────────────────────────────────────┤  │
                     │  │  Temporal difference               │  │
                     │  ├────────────────────────────────────┤  │
                     │  │  Local contrast (integral image)   │  │
                     │  ├────────────────────────────────────┤  │
                     │  │  Color / HSV evidence              │  │
                     │  ├────────────────────────────────────┤  │
                     │  │  Size + aspect ratio filter        │  │
                     │  └────────────────────────────────────┘  │
                     └──────────────────┬───────────────────────┘
                                        │
                                        ▼
                     ┌──────────────────────────────────────────┐
                     │          SCORING + TRACKING               │
                     │  Weighted confidence │ Heat-map suppress  │
                     │  Event tracker       │ Re-arm logic       │
                     │  (50px / 3-frame)    │ (5-frame cooldown) │
                     └──────────────────┬───────────────────────┘
                                        │
                        ┌───────────────┼───────────────┐
                        ▼               ▼               ▼
               ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
               │ Event CSV /  │ │ Annotated    │ │ Contact      │
               │ JSON         │ │ frames       │ │ sheets       │
               └──────┬───────┘ └──────────────┘ └──────────────┘
                      │
                      ▼
               ┌──────────────────────────────────────────┐
               │             EVALUATION                    │
               │  Ground truth CSV │ P/R/F1 engine        │
               │  V1 vs V2 audit   │ Pytest suite (64)    │
               └──────────────────────────────────────────┘
```

## Problem

Detect very brief laser contacts on static target boards in low-frame-rate IPC footage and return event timestamps and pixel locations. No labeled dataset existed for training a learned model — so I started with interpretable signals and a review loop.

## Approach

**No ML training** — entirely classical computer vision:

1. **Multi-signal scoring**: brightness, temporal difference, local contrast (integral images), color/HSV evidence, component size/aspect ratio
2. **Exponential background model** (α=0.01) suppresses persistent scene content
3. **Heat-map suppression** and exclusion zones reduce repeated false positives
4. **Event tracking** associates nearby detections across frames with gap bridging

## Evidence

| Metric | Value | Context |
|--------|-------|---------|
| Grayscale F1 | **0.947** | Two labeled clips (provisional ground truth) |
| Precision | **0.90** | On labeled grayscale clips |
| Recall | **1.00** | On labeled grayscale clips |
| Localization error | **0.32–2.45 px** | Per labeled clip |
| Test suite | **64 tests** | Synthetic and regression tests |

> Ground truth was partly detector-assisted; figures do not generalize to the color camera.

### V1 → V2 Improvements

- Board-aware filtering and integral-image contrast
- ~75% reduction in color-camera false positives
- ~3× throughput improvement
- Early component rejection

## Tech Stack

- **Vision**: OpenCV 4.x, NumPy, FFmpeg
- **Testing**: Pytest (64 tests), synthetic test fixtures
- **Config**: YAML-driven parameters
- **Output**: CSV/JSON events, annotated frames, contact sheets, comparison reports

## CLI Modes

```bash
python detect.py inspect   # Frame-by-frame visual inspection
python detect.py analyze   # Full detection run
python detect.py review    # Interactive result review
python detect.py evaluate  # Formal P/R/F1 evaluation
python detect.py live      # Real-time detection experiment
```

## What I Would Improve

- Create independently labeled clips across lighting conditions and camera types
- Measure event-level precision/recall with confidence intervals
- Test a lightweight learned classifier only after the classical baseline and failure slices are stable

## Disclosure

Raw camera footage and site identifiers are excluded. Metrics are provisional because ground truth was partly detector-assisted.

---

**Portfolio**: [amir-sayab-portfolio.vercel.app](https://amir-sayab-portfolio.vercel.app/work/laser-point-detection/) · **LinkedIn**: [linkedin.com/in/amir-sayab](https://www.linkedin.com/in/amir-sayab/)
