# Automated-Photo-Identification-and-Classification-for-Traffic-Violations-Using-Computer-Vision

# Sentinel-BLR v6.1

### Evidence-Quality Traffic Enforcement Framework for Smart Cities

[![Status](https://img.shields.io/badge/Status-Framework%20Proposal-blue)](https://img.shields.io/badge/Status-Framework%20Proposal-blue)
[![Hackathon](https://img.shields.io/badge/Flipkart%20Gridlock-2.0-orange)](https://img.shields.io/badge/Flipkart%20Gridlock-2.0-orange)
[![Theme](https://img.shields.io/badge/Theme-3-success)](https://img.shields.io/badge/Theme-3-success)
[![Domain](https://img.shields.io/badge/Computer%20Vision-Traffic%20Enforcement-red)](https://img.shields.io/badge/Computer%20Vision-Traffic%20Enforcement-red)

---

## Quick Links

| Resource | Link |
|---|---|
| Technical Proposal (PDF) | [`docs/Sentinel_BLR_v6.1.pdf`](docs/Sentinel_BLR_v6.1.pdf) |
| System Architecture Diagram | [`diagrams/Sentinel-BLR architecture.png`](diagrams/Sentinel-BLR%20architecture.png) |
| Pipeline Flow Diagram | [`diagrams/Sentinel-BLR pipeline.png`](diagrams/Sentinel-BLR%20pipeline.png) |
| Pitch Deck | [`docs/Sentinel_BLR_Pitch.pptx`](docs/Sentinel_BLR_Pitch.pptx) |

---

## Overview

**Sentinel-BLR** is an AI-powered **Evidence Quality Framework** designed for automated traffic enforcement in urban environments.

Unlike traditional computer vision pipelines that stop after detecting a violation, Sentinel-BLR introduces an evidence validation layer that determines whether the detected violation is reliable enough for legal enforcement.

The framework combines modern object detection, multi-frame reasoning, intelligent policy routing, auto-calibration, OCR fusion, and evidence packaging into a single end-to-end architecture.

Developed as a proposal for **Flipkart Gridlock 2.0 – Theme 3**, the system focuses on making AI-based traffic enforcement more reliable, scalable, and legally defensible.

---

## Why Sentinel-BLR?

Conventional traffic violation systems generally follow:

```
Camera → Vehicle Detection → Violation Classification → Fine Generated
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

Sentinel-BLR addresses these challenges through an evidence-centric architecture, using a single composite **Evidence Reliability Score (ERS)** to gate every enforcement decision.

---

## Key Features

### Auto-Calibration Heartbeat

- Detects camera drift every 15 minutes using SIFT/ORB feature matching
- Computes homography transformation and automatically re-projects all Regions of Interest (ROIs)
- Eliminates manual recalibration; calibration health feeds directly into the ERS

---

### Adaptive Temporal Striding (ATS)

Instead of disabling expensive models during heavy workloads, Sentinel-BLR intelligently reduces inference frequency while maintaining detection quality.

- Real-time performance under ≤ 28 ms standard / ≤ 38 ms adverse-condition compute budget
- Lower GPU utilisation with graceful degradation under load

---

### Temporal OCR Fusion

Instead of selecting the best OCR result from a single frame:

- Stores character-level probabilities across **5 frames**
- Applies per-position softmax fusion across the window
- Removes corrupted frames via Levenshtein outlier filtering
- Produces more reliable license plate recognition, especially on partial or dirty plates

---

### Confidence-Gated Enforcement

Violation detection and enforcement are intentionally separated.

Every detected violation is routed through:

- Per-violation-type confidence thresholds
- Multi-frame consensus
- OCR verification (OCR confidence < 90% → human review, never auto-fine)
- Calibration health check
- YAML-based policy rules

before any enforcement decision is made.

---

### Evidence Reliability Score (ERS)

Every evidence packet receives a composite score (0–100):

```
ERS = 0.30·Cdet + 0.25·Cocr + 0.20·Rcons + 0.15·Hcal + 0.10·Qpre
```

| Component | Weight | What it measures |
|---|---|---|
| C_det | 0.30 | Detection confidence |
| C_ocr | 0.25 | OCR / plate-read confidence |
| R_cons | 0.20 | Multi-frame agreement |
| H_cal | 0.15 | Calibration health |
| Q_pre | 0.10 | Image preprocessing quality |

| ERS | Decision |
|---|---|
| ≥ 85 | Auto-Fine |
| 60 – 84 | Human Review |
| < 60 | Discard — no record |

ERS enables transparent review and prioritisation without introducing an additional machine learning model. Weak evidence structurally cannot become a fine.

The 85 auto-fine threshold is a logically-motivated prior, not an asserted number. Phase 1 shadow mode calibrates it using officer-adjudicated ground truth, and a conformal risk control layer then sets per-violation-type cutoffs carrying a provable ≤ 1% wrongful-fine guarantee at 95% confidence — the threshold is set by data, not by engineers.

---

### Conformal ERS Calibration

The ERS gate is not hand-tuned and left fixed. After shadow-mode operation supplies officer-labelled outcomes, Sentinel-BLR applies **Mondrian conformal risk control** — a distribution-free statistical method that sets a per-violation-type auto-fine threshold from the calibration data directly. The result is a provable guarantee: auto-fines are issued only inside a region whose wrongful-fine rate is ≤ 1% at 95% confidence. Violation types with harder error profiles (wrong-side, stop-line) automatically receive a higher threshold; easier categories (helmet, parking) may receive a lower one. The system does not ask BTP to trust a hand-set number.

---

### Citizen Evidence Portal

Every issued challan carries a QR code linking to an access-controlled page visible only to the registered owner. It shows the annotated frame, violation type and MV Act section, ERS and all five sub-scores, and a GradCAM saliency overlay — the same view the officer saw. Owners can file a structured dispute (plate misread, not a violation, exempt vehicle, duplicate challan) that provisionally stays the fine pending human review. Every overturned dispute feeds directly into the conformal calibration set and the active-learning loop — citizen disputes do not just correct individual errors, they continuously tighten the system that produced them.

---

### Hybrid Evidence Payloads

Instead of transmitting raw video streams, every confirmed violation generates a tamper-evident packet containing: annotated frame + GradCAM overlay + UTC timestamp + GPS + junction ID + plate text + violation type + vehicle class + SHA-256 hash + ERS score + MV Act citation. Each hash is anchored with an RFC 3161 trusted-timestamp token so the record proves both integrity and time of capture — a bare hash without a time anchor is insufficient for a contested challan.

---

## System Architecture

```
Camera Feed (RTSP/ONVIF)
      │
      ▼
Environmental Preprocessing (CLAHE · Retinex · DCP · Wiener)
      │
      ▼
Auto-Calibration Heartbeat (SIFT/ORB · every 15 min)
      │
      ▼
Vehicle Detection + Tracking (YOLOv10-N TensorRT INT8 · ByteTrack)
      │
      ▼
Inference Budget Controller + ATS (per-Track-ID gating)
      │
      ▼
Violation Detection Engine (5 parallel triggered paths)
      │
      ▼
Confidence Gate + ERS (0–100 reliability score)
      │
      ├── ERS ≥ 85 → AUTO-FINE
      ├── ERS 60–84 → HUMAN REVIEW
      └── ERS < 60 → DISCARD
      │
      ▼
Enforcement Policy Engine (YAML · BTP-editable · < 1 min to update)
      │
      ▼
License Plate OCR (PaddleOCR PP-OCRv4 · 5-frame temporal fusion)
      │
      ▼
Evidence Packet Generator (SHA-256 tamper-evident · RFC 3161 timestamp)
      │
      ▼
Strategic Intelligence (Junction Urgency Score · LoRA fine-tuning loop)
      │
      ▼
Analytics Dashboard (GIS heatmap · Streamlit · Folium · Plotly)
```

---

## Complete Pipeline

The framework consists of **15 logical layers** grouped into **7 functional modules**.

| Group | Responsibility |
|---|---|
| Ingestion | Camera feeds and streaming (Kafka) |
| Preprocessing | Image enhancement and auto-calibration |
| Detection | Vehicle detection (YOLOv10-N) and tracking (ByteTrack) |
| Classification | 5-path violation detection engine |
| Decision | ERS confidence gate + YAML enforcement policy |
| Evidence | Temporal OCR fusion, GradCAM explainability, tamper-evident packaging |
| Intelligence | Junction urgency analytics and weekly LoRA adaptation |

---

## Traffic Violations Supported

| Violation | Phase 1 Auto-Fine Eligible | Notes |
|---|---|---|
| Helmet non-compliance | ✅ Yes | 3-frame consensus; §129 MV Act 2019 |
| Seatbelt non-compliance | ❌ Review only | Promoted to auto-fine in Phase 2 upon ≥ 95% windshield-specific validation |
| Triple riding | ❌ Review only | Never auto-fines regardless of confidence; crowd index threshold pending Phase 1 validation; §128 / §194C MV Act 2019 |
| Wrong-side driving | ⚠️ Conditional | Human review only at junctions with contested OSM directionality; §184(e) MV Act 2019 |
| Stop-line / red-light crossing | ✅ Yes | 3-frame consensus; signal-state confirmed |
| Illegal parking (1st offence) | ⚠️ Warning only | Auto-fine on 2nd offence within 30 days |
| Illegal parking (repeat) | ✅ Yes | 180 s duration threshold |

---

## Technology Stack

| Component | Technology |
|---|---|
| Object Detection | YOLOv10-N (TensorRT INT8) |
| Tracking | ByteTrack |
| Safety Classification | MobileNetV3 (LoRA fine-tuned) |
| Pose Estimation | MediaPipe Pose Lite |
| OCR | PaddleOCR PP-OCRv4 |
| Feature Matching | SIFT / ORB |
| Image Processing | OpenCV (CLAHE, Retinex, DCP, Wiener) |
| Deep Learning | PyTorch |
| Inference Runtime | TensorRT INT8 |
| Backend | FastAPI |
| Message Streaming | Apache Kafka |
| Database | PostgreSQL + TimescaleDB |
| Analytics | DuckDB |
| Dashboard | Streamlit + Folium + Plotly |
| Edge Hardware | NVIDIA Jetson Orin NX |
| Deployment | Docker |

---

## Repository Structure

```
Sentinel-BLR/
│
├── docs/
|   ├── Sentinel_BLR.pdf
│   ├── Sentinel_BLR_Code.tex        # Full technical proposal (LaTeX source)
|   └── Sentinel_BLR_Pitch_v2.pptx
│
├── diagrams/
│   ├── Sentinel-BLR architecture.png
│   └── Sentinel-BLR pipeline.png
│
└── README.md
```

> **Note on datasets:** Training datasets (Police Violations Dataset — 298,450 records; ASTRAM — 8,173 records) are not included in this repository due to size constraints. See the Datasets section below for sources.

---

## Deployment Roadmap

### Phase 1 — Pilot (Shadow Mode First)

- 10 high-urgency junctions (top-10 by enforcement load from Police Violations Dataset: Safina Plaza, KR Market, Elite Junction, Sagar Theatre, Central Street, Subbanna, Modi Bridge, Hosahalli Metro, Anand Rao, NR Road/SP Road — real rows, auditable)
- **Stage 0 — Shadow Mode:** Full pipeline runs live. Zero fines issued. Officers adjudicate normally; their decisions become ground-truth labels that calibrate the system. No junction advances to Stage 1 until per-type auto-fine precision clears **99%** on officer-adjudicated outcomes.
- **Stage 1 — Live:** Auto-fine region only; human review and citizen portal active; continuous drift and overturn monitoring.
- Go/no-go criteria: mAP@0.5 ≥ 0.85 AND ERS-gated auto-fine precision ≥ **99%** on Phase 1 officer-adjudicated data

### Phase 2 — Expand

- Replace synthetic priors with live field data (fully phased out within 30 days of Phase 1 start)
- Expand to 50 junctions selected by live Sentinel-BLR urgency scores, not manual survey
- Promote seatbelt detection to auto-fine if Phase 1 windshield set clears ≥ 95% precision
- Improve OCR on degraded and Kannada-character plates

### Phase 3 — City Scale

- City-scale multi-camera deployment
- Integrate with BTP's existing 250 ANPR + 80 RLVD ITMS cameras as an ERS overlay layer
- Live strategic traffic analytics and automatic traffic signal integration
- Gate: infrastructure cost grows linearly or sub-linearly with junction count

---

## Performance Targets

| Metric | Target | Scope |
|---|---|---|
| Detection mAP@0.5 | ≥ 0.85 | YOLOv10-N on held-out Phase 1 footage |
| Detector Precision | ≥ 95% | Before ERS gate |
| **ERS-gated Auto-Fine Precision** | **≥ 99%** | After ERS ≥ 85 filter — the enforcement bar |
| Triple Riding Recall | ≥ 90% | Human-review path; recall prioritised over precision |
| OCR Accuracy (clean plates) | ≥ 95% | Post temporal fusion |
| OCR Accuracy (degraded plates) | ≥ 80% | Post temporal fusion |
| End-to-End Latency (standard) | ≤ 28 ms | Jetson Orin NX, clear conditions, no adverse preprocessing |
| End-to-End Latency (adverse) | ≤ 38 ms sustained cadence | Inter-frame CUDA pipelining; per-frame p95 latency reported separately |

> Detector precision (95%) and ERS-gated auto-fine precision (99%) are intentionally separate targets. A bare 95% detector precision would produce a wrongful-fine rate higher than the system this proposal critiques — Hyderabad's reported 26,000-challan episode amounted to roughly 0.2% of challans issued, and that sub-1% rate was still large enough to damage public trust. The ERS gate exists precisely to lift the auto-fine subset to ≥ 99%, routing everything it is not confident about to human review rather than to a fine.

> The ≤ 38 ms adverse-peak figure is a **sustained throughput target** (one fully processed frame emitted every ≤ 38 ms via inter-frame CUDA stream pipelining), not a single-frame end-to-end latency claim. Per-frame p95 latency is reported separately in Phase 1 validation; the go/no-go gate is the sustained cadence.

---

## Datasets

| Dataset | Size | Source |
|---|---|---|
| Police Violations Dataset (Flipkart Gridlock Dataset B) | 298,450 records | Flipkart Gridlock 2.0 |
| ASTRAM Event Dataset (Flipkart Gridlock Dataset A) | 8,173 records | Flipkart Gridlock 2.0 |
| IDD / ACFR / Roboflow Helmet | Pre-trained weights | Public |
| OpenStreetMap | Road geometry, one-way directionality | OSM |
| IMD Weather API | Adverse-condition routing | Government of India |
| BMTC GTFS | Bus-stop no-parking polygons | BMTC |

> **Note on safety violation data scarcity:** The Police Violations Dataset contains only 2 helmet records and 0 triple-riding records across 298,450 rows (97.3% parking offences). Sentinel-BLR addresses the cold-start gap via a disciplined synthetic bootstrap (CARLA-class simulation matched to Phase 1 junction coordinates + diffusion image-to-image restyling into Bengaluru lighting distributions + cut-paste compositing on real CCTV backgrounds). Synthetic data is used only for pre-Phase-1 classifier training, carries < 20% weight from day 1, and is fully replaced by live shadow-mode labels within 30 days. Synthetic accuracy is never reported as deployment accuracy.

---

## Novel Contributions

Sentinel-BLR introduces several architectural ideas beyond conventional YOLO+OCR traffic violation systems:

1. **Evidence Reliability Score (ERS)** — composite 0–100 enforcement gate replacing five independent confidence checks; weights derived from failure-consequence hierarchy and re-estimated via ridge-regularised least-squares after 500 labelled Phase 1 outcomes
2. **Auto-Calibration Heartbeat** — SIFT/ORB homography drift correction every 15 minutes, automatic ROI re-projection, no manual intervention
3. **Adaptive Temporal Striding (ATS)** — compute-budget-aware inference frequency reduction without disabling detection or dropping violation coverage
4. **Temporal Character-Level OCR Fusion** — per-position softmax across 5 frames, Levenshtein outlier removal, character-template alignment before summation
5. **Confidence-Gated Enforcement** — structural separation of detection confidence from enforcement decision; OCR gate and calibration gate enforced by policy engine, not model thresholds
6. **YAML Policy Engine** — enforcement rules as editable config, not model retraining; updates live in < 1 min; BTP-operable without engineer involvement
7. **Hybrid Evidence Payload Architecture** — lightweight tamper-evident packet (JSON + < 50 KB ROI JPEG) vs. raw video stream; SHA-256 + RFC 3161 trusted-timestamp anchoring
8. **Closed-Loop Strategic Deployment Intelligence** — junction urgency scoring with 0.95^days recency decay and weekly LoRA adaptation of MobileNetV3 classification head
9. **Conformal ERS Calibration** — Mondrian conformal risk control sets per-violation-type auto-fine thresholds from shadow-mode labels, carrying a provable ≤ 1% wrongful-fine guarantee at 95% confidence; threshold set by data, re-validated against drift
10. **Citizen Evidence Portal** — per-challan annotated frame + ERS sub-scores + GradCAM overlay exposed to the registered owner via QR code; structured dispute workflow with provisional fine stay; every overturn feeds directly into conformal recalibration

---

## Legal & Compliance Framework

- **Motor Vehicles Act 2019 citations:**
  - §129 — helmet headgear requirement
  - §128 — pillion rider limit (driver + one pillion); penalty under §194C
  - §184(e) — wrong-side / dangerous driving *(not §185, which covers drunken driving)*
  - §194C — penalty for triple riding / stop-line violations
  - §194D — enhanced penalty on repeat offence
- **DPDP Act 2023:** Face-blurring before storage; raw frames deleted within 24 hours; only violation-confirmed frames retained at full resolution; BTP's city-wide deployment plausibly qualifies as a Significant Data Fiduciary under §10 — flagged as a Phase 1 compliance review item
- **Tamper evidence:** SHA-256 hash + RFC 3161 trusted-timestamp anchoring on every evidence packet; hash alone without time anchor is insufficient for a contested challan
- **Structural safeguards:** Triple riding and seatbelt violations are architecturally blocked from auto-fining in Phase 1 regardless of confidence; no junction issues auto-fines before clearing 99% precision in shadow mode

---

## Current Status

This repository accompanies the **Flipkart Gridlock 2.0 Phase 2 submission**.

At this stage it contains the system architecture, full technical proposal (v6.1), design methodology, deployment strategy, and evaluation framework. A production implementation is planned in future development phases.

---

## Future Work

- Real-time implementation and edge deployment on NVIDIA Jetson Orin NX
- Multi-camera junction fusion
- Automatic traffic signal integration
- Seatbelt detection promotion to auto-fine (Phase 2, subject to windshield validation)
- Active learning pipeline with reviewer-labelled data
- EigenCAM explainability (forward-pass compatible with TensorRT INT8)
- OCR extension for Kannada-script state identifier characters on legacy plates
- §210A fine multiplier exposed as a configurable parameter in Layer 4A policy engine

---

## Hackathon

**Competition:** Flipkart Gridlock 2.0
**Phase:** Phase 2
**Theme:** Theme 3 – Automated Photo Identification and Classification for Traffic Violations

---

## Team Rocket

| Member | Institution |
|---|---|
| Krishanu Deka | NIT Silchar (ECE) |
| Bedagya Bordoloi | VIT AP (CSE) |

**Project:** Sentinel-BLR — Evidence-Quality Traffic Enforcement Framework

---

> **Note:** Sentinel-BLR is currently a research and system-design proposal for Flipkart Gridlock 2.0. The repository focuses on architecture, methodology, and deployment strategy rather than a production-ready implementation.
