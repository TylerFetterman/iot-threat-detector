# IoT Threat Detector

A machine learning project combining IoT device anomaly detection and malicious URL classification to identify compromised smart devices and their C2 callback traffic.

## Modules
- **Module 1**: IoT traffic anomaly detection using the N-BaIoT dataset
- **Module 2**: Phishing/malicious URL classification
- **Module 3**: Combined risk scoring pipeline

## Stack
Python · scikit-learn · pandas · Jupyter

## Status
In progress - summer 2026

## Limitations
        **IoT Anomaly Detector (Module 1):** Achieved near-perfect accuracy (100%) across multiple Mirai botnet attack types (scan, tcp, syn, ack, udp). Investigation into feature importance showed no single dominant feature explains this -- importance is spread across many features, suggesting Mirai-infected devices produce traffic that differs from normal usage across many statistical dimensions simultaneously. This likely reflects a genuine property of this dataset (volumetric botnet attacks are highly distinct) rather than a flw in the pipeline, but also means this result may not generalize to more subtle or evasive attack types not represented in N-BaIoT.
    **URL Classifier (Module 2):** Performs well overall (83% accuracy) but struggles to catch sophosticated phishing URLs (63% recall on "bad" class), since structural features like length and entropy can't detect phishing sites designed to closly mimic legitmate URL patterns.

