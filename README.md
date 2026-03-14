# 🔬 Real-Time SPR Baseline Prediction System

> A hybrid machine learning system for real-time baseline drift correction in fiber-optic Surface Plasmon Resonance (SPR) biosensors.

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://python.org)
[![Scikit-learn](https://img.shields.io/badge/Scikit--learn-F7931E?logo=scikit-learn&logoColor=white)](https://scikit-learn.org)
[![Prophet](https://img.shields.io/badge/Facebook_Prophet-0467DF?logo=meta&logoColor=white)](https://facebook.github.io/prophet/)
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen)]()

---

## Problem

Fiber-optic SPR biosensors produce continuous optical data streams during drug injection experiments. The raw signal contains **instrumental baseline drift** that must be subtracted to isolate the true biochemical response.

The previous workflow relied on **manual, repetitive calculations** to correct for this drift — a process that was:
- ⏱️ Time-consuming (manual correction for each fiber channel)
- ❌ Highly vulnerable to user-related errors
- 📉 Unable to handle non-linear drift patterns

## Solution

A **hybrid ML pipeline** that automatically predicts the underlying baseline in real-time, enabling precise and error-free baseline difference calculation during live experiments.

### Architecture Overview

```
Raw Sensor Data (500ms intervals)
        │
        ▼
┌─────────────────────────┐
│  Savitzky-Golay Filter  │  ← Noise reduction (window=51, poly=3)
│  (Preprocessing)        │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│         4000s Learning Window           │  ← Captures initial trend
│    (Train-first, predict-then)          │     before drug injection
└──────────┬──────────────────────────────┘
           │
     ┌─────┴─────┐
     ▼           ▼
┌──────────┐ ┌──────────────┐
│Penalized │ │  Facebook    │
│Exponential│ │  Prophet     │
│Decay +   │ │  (Residual   │
│Linear Reg│ │  Correction) │
└────┬─────┘ └──────┬───────┘
     │               │
     ▼               ▼
┌─────────────────────────┐
│  Ensemble Blending      │  ← α = 0.80 (80% decay, 20% Prophet)
│  (Weighted Combination) │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  Real-Time GUI          │  ← 3 panels, 500ms refresh
│  + Excel Logging        │
└─────────────────────────┘
```

## Methodology

### Preprocessing
- **Savitzky-Golay Smoothing Filter** (Window=51, Polynomial Order=3) removes high-frequency noise from the continuous optical data stream, revealing the true wavelength trend for model training.

### Hybrid Model
The prediction engine is an ensemble of two complementary components:

| Component | Role | Weight |
|-----------|------|--------|
| **Penalized Exponential Decay + Linear Regression** | Captures rapid initial decrease and slow long-term residual trends | 80% (α=0.80) |
| **Facebook Prophet (Residual Correction)** | Bayesian time-series model corrects remaining non-linear residuals | 20% |

### Training Strategy
- **Data Ingestion Phase**: A 4000-second (~66.7 min) learning window captures the initial wavelength trend *without* making predictions
- **Prediction Phase**: After the learning window, the model predicts the baseline in real-time as the drug injection experiment proceeds

## Results

### Performance Comparison (Mean Absolute Error)

| Approach | File 1 MAE | File 10 MAE | Notes |
|----------|-----------|-------------|-------|
| 2nd-Order Exponential Smoothing | 2.57 nm | 15.52 nm | Rigid, linear — fails on non-linear curves |
| Facebook Prophet (standalone) | 2.10 nm | 6.80 nm | Better, but struggles without seasonal patterns |
| **Hybrid Model (Ours)** | **0.089 nm** | **0.010 nm** | **99.6% error reduction vs legacy** |

### Key Achievement
- **File 10**: Achieved MAE of **0.0104 nm** — near-perfect baseline prediction enabling precise biochemical response calculation

## Tech Stack

- **Language**: Python 3.10+
- **ML/Stats**: Scikit-learn, Facebook Prophet, SciPy (Savitzky-Golay)
- **Data**: Pandas, NumPy
- **Visualization**: Matplotlib, Tkinter (real-time GUI)
- **Logging**: OpenPyXL (Excel export)

## Real-Time GUI

The system features a live monitoring interface with:
- **3 dynamic panels**: Raw signal, smoothed + predicted baseline, and baseline difference
- **500ms refresh rate** for real-time experiment monitoring
- **Automatic Excel logging** of raw, smoothed, predicted, and difference signals

> *Screenshots and demo will be added upon approval.*

## Previous Approaches Evaluated

1. **2nd-Order Exponential Smoothing**: Learns level and velocity of drift, but produced rigid linear predictions that couldn't adapt to non-linear baseline curves
2. **Facebook Prophet (standalone)**: Bayesian time-series model, but relies on seasonal patterns which don't exist in trend-driven biosensor data
3. **Hybrid Model (final)**: Combines the physical intuition of exponential decay with Prophet's residual correction — best of both worlds

## Future Directions

- **Adaptive Learning Windows**: Replace the hardcoded 4000s ingestion phase with dynamic training that monitors signal derivatives to autonomously determine when drift has stabilized
- **Algorithmic Event Triggering**: Replace fixed temporal thresholds with an event-driven "Persistence Gate" that detects drug injection events from the data stream itself

## Team

| Name | Role |
|------|------|
| **Jetra Vyas** | ML Pipeline Development, Hybrid Model Architecture, GUI Application |



---

## License

This project was developed as part of a collaborative research initiative. Code and proprietary data are not publicly available. This repository contains documentation, methodology, and results only.

> *Additional materials (code samples, demo videos) will be added upon obtaining necessary permissions.*

---

*Built at SRH Heidelberg University, 2025*
