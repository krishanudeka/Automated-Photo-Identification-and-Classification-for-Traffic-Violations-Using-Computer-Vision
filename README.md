# Automated-Photo-Identification-and-Classification-for-Traffic-Violations-Using-Computer-Vision

# 🚦 Sentinel-BLR v4.2
### Evidence-Quality Traffic Enforcement Framework for Smart Cities

![Status](https://img.shields.io/badge/Status-Framework%20Proposal-blue)
![Hackathon](https://img.shields.io/badge/Flipkart%20Gridlock-2.0-orange)
![Theme](https://img.shields.io/badge/Theme-3-success)
![Domain](https://img.shields.io/badge/Computer%20Vision-Traffic%20Enforcement-red)

---

## Overview

**Sentinel-BLR** is an AI-powered **Evidence Quality Framework** designed for automated traffic enforcement in urban environments.

Unlike traditional computer vision pipelines that stop after detecting a violation, Sentinel-BLR introduces an evidence validation layer that determines whether the detected violation is reliable enough for legal enforcement.

The framework combines modern object detection, multi-frame reasoning, intelligent policy routing, auto-calibration, OCR fusion, and evidence packaging into a single end-to-end architecture.

Developed as a proposal for **Flipkart Gridlock 2.0 – Theme 3**, the system focuses on making AI-based traffic enforcement more reliable, scalable, and legally defensible.

---

# Why Sentinel-BLR?

Conventional traffic violation systems generally follow:

```
Camera
   ↓
Vehicle Detection
   ↓
Violation Classification
   ↓
Fine Generated
```

This works well in ideal conditions but fails under real-world deployment due to:

- Camera drift
- Motion blur
- Rain and fog
- Dirty or partially visible license plates
- Dense traffic
- False positives
- Lack of explainability
- Poor legal reliability

Sentinel-BLR addresses these challenges through an evidence-centric architecture.

---

# Key Features

## Auto-Calibration Heartbeat

- Detects camera drift every 15 minutes
- Uses SIFT/ORB feature matching
- Computes homography transformation
- Automatically re-projects all Regions of Interest (ROIs)
- Eliminates manual recalibration

---

## Adaptive Temporal Striding (ATS)

Instead of disabling expensive models during heavy workloads, Sentinel-BLR intelligently reduces inference frequency while maintaining detection quality.

Benefits:

- Real-time performance
- Lower GPU utilization
- Consistent detection coverage
- Graceful degradation under adverse conditions

---

## Temporal OCR Fusion

Instead of selecting the best OCR result from a single frame:

- Stores character probabilities across multiple frames
- Performs probability fusion
- Removes corrupted OCR frames
- Produces more reliable license plate recognition

---

## Confidence-Gated Enforcement

Violation detection and enforcement are intentionally separated.

Every detected violation is routed through:

- Confidence thresholds
- Multi-frame consensus
- OCR verification
- Calibration health
- Policy rules

before any enforcement decision is made.

---

## Evidence Reliability Score (ERS)

Every evidence packet receives a composite score (0–100) computed using:

- Detection confidence
- OCR confidence
- Multi-frame agreement
- Calibration health
- Image preprocessing quality

ERS enables transparent review and prioritization without introducing another machine learning model.

---

## Hybrid Evidence Payloads

Instead of transmitting raw video streams:

- Metadata
- Cropped violation image
- GPS
- Timestamp
- Vehicle information
- OCR result
- Evidence score

are transmitted together as a lightweight evidence packet.

---

# System Architecture

```
Camera Feed
      │
      ▼
Environmental Preprocessing
      │
      ▼
Auto Calibration
      │
      ▼
Vehicle Detection
      │
      ▼
Inference Budget Controller
      │
      ▼
Violation Detection
      │
      ▼
Confidence Validation
      │
      ▼
Policy Engine
      │
      ▼
License Plate OCR
      │
      ▼
Evidence Generation
      │
      ▼
Strategic Intelligence
      │
      ▼
Analytics Dashboard
```

---

# Complete Pipeline

The framework consists of **15 logical layers** grouped into **7 functional modules**.

| Group | Responsibility |
|--------|---------------|
| Ingestion | Camera feeds and streaming |
| Preprocessing | Image enhancement and calibration |
| Detection | Vehicle detection and tracking |
| Classification | Violation detection |
| Decision | Confidence validation and policy engine |
| Evidence | OCR, explainability, evidence packaging |
| Intelligence | Analytics and deployment planning |

---

# Traffic Violations Supported

- Helmet Violation
- Seatbelt Violation
- Triple Riding
- Wrong Side Driving
- Stop Line Crossing
- Red Light Violation
- Illegal Parking

---

# Technology Stack

| Component | Technology |
|------------|------------|
| Object Detection | YOLOv10 |
| Tracking | ByteTrack |
| OCR | PaddleOCR |
| Pose Estimation | MediaPipe Pose Lite |
| Feature Matching | SIFT / ORB |
| Image Processing | OpenCV |
| Deep Learning | PyTorch |
| Inference | TensorRT INT8 |
| Backend | FastAPI |
| Database | PostgreSQL + TimescaleDB |
| Analytics | DuckDB |
| Dashboard | Streamlit |
| Streaming | Apache Kafka |
| Deployment | Docker |

---

# Proposed Repository Structure

```
Sentinel-BLR/

├── docs/
│   ├── Project_Proposal.pdf
│
├── diagrams/
│   |__ architecture.png
│
├── datasets/
│
|__ README.md

```

---

# Deployment Roadmap

### Phase 1

- Pilot deployment
- High-priority junctions
- Validate detection accuracy
- Measure latency

---

### Phase 2

- Replace synthetic priors with live data
- Expand deployment
- Improve OCR
- Optimize models

---

### Phase 3

- City-scale deployment
- Multi-camera support
- Strategic traffic analytics
- Continuous model refinement

---

# Performance Targets

| Metric | Target |
|---------|---------|
| Detection mAP | ≥ 0.85 |
| Auto-fine Precision | ≥ 95% |
| Triple Riding Recall | ≥ 90% |
| OCR Accuracy | ≥ 95% |
| End-to-End Latency | ≤ 38 ms (Design Target) |

---

# Datasets

The proposal utilizes:

- Flipkart Gridlock Dataset A = ASTRAM Event Dataset
- Flipkart Gridlock Dataset B = Police Violations Dataset
- Public Karnataka road statistics (synthetic priors)
- OpenStreetMap
- IMD Weather Data
- BMTC GTFS

---

# Novel Contributions

Sentinel-BLR introduces several architectural ideas beyond conventional traffic violation systems:

- Auto-calibrating camera geometry
- Adaptive Temporal Striding
- Temporal character-level OCR fusion
- Confidence-gated enforcement
- Policy engine driven by configuration instead of model retraining
- Evidence Reliability Score (ERS)
- Hybrid evidence payload architecture
- Closed-loop strategic deployment intelligence

---

# Current Status

This repository accompanies the **Flipkart Gridlock 2.0 Phase 2 submission**.

At this stage it contains:

- System architecture
- Technical proposal
- Design methodology
- Deployment strategy
- Evaluation framework

A production implementation is planned in future development phases.

---

# Future Work

- Real-time implementation
- Edge deployment on NVIDIA Jetson
- Multi-camera fusion
- Automatic traffic signal integration
- Live dashboard
- Active learning pipeline
- Explainable AI improvements

---

# Hackathon

**Competition:** Flipkart Gridlock 2.0

**Phase:** Phase 2

**Theme:** Theme 3 – Automated Photo Identification and Classification for Traffic Violations

---

# Team_Rocket

**Project:** Sentinel-BLR 

Evidence-Quality Traffic Enforcement Framework

Developed for Flipkart Gridlock 2.0.

---
---

> **Note:** Sentinel-BLR is currently a research and system-design proposal intended for the Flipkart Gridlock 2.0 hackathon. The repository focuses on architecture, methodology, and deployment strategy rather than a production-ready implementation.
