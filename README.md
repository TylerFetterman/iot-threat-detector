# IoT Threat Detector

Built as a portfolio project for a BS in Cybersecurity at the University of North Dakota.

---

## Project Overview

This project is a risk detection system that uses a trained ML model to predict whether a URL is malicious and a second model to determine if an IoT device is part of a botnet by analyzing its network traffic. Both signals are combined into a single weighted risk score.

---

## Architecture

The system consists of three modules:

1. **IoT Anomaly Detector** — analyzes device network traffic for botnet behavior
2. **Malicious URL Classifier** — predicts whether a URL is legitimate or phishing
3. **Combined Risk Scoring Pipeline** — loads both trained models and produces a unified risk score weighted by signal type

---

## Module 1: IoT Anomaly Detector
`01_iot_anomaly.ipynb`

Uses the **N-BaIoT dataset** (Ben-Gurion University), which contains pre-summarized network traffic statistics captured from a Danmini Doorbell device under both normal and Mirai botnet attack conditions.

**Dataset feature groups:**
- `MI_dir` — packet direction statistics
- `H` — stats about packets from the source IP
- `HH` — stats about the source/destination IP communication channel
- `HH_jit` — jitter (timing variation between packets)
- `HpHp` — stats about a specific source/destination IP and port combination

Each group is measured across multiple lambda decay time windows (L5, L3, L1, L0.1, L0.01), resulting in 115 pre-engineered features. No additional feature engineering was required.

**Results:** 100% precision and recall across five Mirai attack types (ack, syn, tcp, udp, scan). This was initially suspicious, so feature importance was investigated across all attack types. No single dominant feature was found — importance was spread across many features, suggesting Mirai-infected devices produce traffic that differs from normal behavior across many statistical dimensions simultaneously. The high accuracy reflects how distinct volumetric botnet attacks are, not a flaw in the pipeline.

---

## Module 2: Malicious URL Classifier
`02_url_classifier.ipynb`

Uses the **Phishing Site URLs dataset** (Tarun Tiwari, Kaggle) — chosen specifically because it provides raw URLs, allowing for manual feature engineering rather than relying on pre-extracted features.

**Engineered features:**
- `url_length` — total character count
- `num_dots` — number of dots in the URL
- `num_hyphens` — number of hyphens
- `entropy` — randomness of the URL string
- `TLD_grouped` — top-level domain, bucketed into 15 common TLDs + "other", one-hot encoded

**Results:** 83% overall accuracy. Entropy alone accounts for ~40% of feature importance. The model performs better on legitimate URLs (91% recall) than phishing URLs (63% recall), because well-crafted phishing URLs are structurally designed to mimic legitimate ones — the same challenge human users face.

---

## Combined Pipeline
`03_combined_pipeline.ipynb`

Loads both trained models using `joblib` and combines their outputs into a single weighted risk score:
combined_score = (0.65 * traffic_risk) + (0.35 * url_risk)
Traffic is weighted higher (65%) because active attack behavior represents an ongoing compromise. URL risk is weighted lower (35%) because a suspicious URL is a potential threat that may not have been acted on yet.

**Risk thresholds:**
| Score Range | Risk Level |
|---|---|
| 0.0 – 0.3 | Low |
| 0.3 – 0.7 | Medium |
| 0.7 – 1.0 | High |

---

## Limitations & Future Work

**URL Classifier:** Feature engineering is structural rather than content-based, limiting detection of sophisticated phishing sites. Future improvements: analyzing page content, domain registration age, or SSL certificate details.

**IoT Anomaly Detector:** The N-BaIoT dataset represents obvious volumetric attacks that don't attempt to blend in with normal traffic. Results may not generalize to more subtle or evasive intrusion types. Future improvements: testing on datasets with more sophisticated, low-and-slow attack patterns.

---

## Stack

Python · scikit-learn · pandas · Jupyter · joblib · matplotlib · seaborn

---

## Setup / How to Run

**1. Clone this repo:**
```bash
git clone https://github.com/TylerFetterman/iot-threat-detector
```

**2. Download datasets and place all CSV files in the `data/` folder:**
- N-BaIoT (Danmini Doorbell): https://archive.ics.uci.edu/dataset/442/detection+of+iot+botnet+attacks+n+baiot  
  Required: `benign_traffic.csv`, `ack.csv`, `syn.csv`, `tcp.csv`, `udp.csv`, `scan.csv`
- Phishing Site URLs: https://www.kaggle.com/datasets/taruntiwarihp/phishing-site-urls  
  Required: `phishing_site_urls.csv`

**3. Install dependencies:**
```bash
pip install pandas scikit-learn matplotlib seaborn jupyter joblib
```

**4. Run notebooks in order:**
- `01_iot_anomaly.ipynb` — trains and saves the IoT anomaly detection model
- `02_url_classifier.ipynb` — trains and saves the URL classification model
- `03_combined_pipeline.ipynb` — loads both models and runs the combined risk scoring pipeline

> **Note:** Trained model files (`.pkl`) are excluded from this repo due to file size. They are regenerated automatically when you run the notebooks above.
