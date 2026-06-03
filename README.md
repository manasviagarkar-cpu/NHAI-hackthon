# NHAI FaceRec — 100% Offline Facial Recognition & Liveness Detection

Submission for **NHAI Innovation Hackathon 7.0**. An enterprise-grade, offline-first attendance verification application built with React Native, TFLite on-device inference, and advanced safety features.

---

## 📱 App Visuals & UI

| Home Screen | Facial Recognition | Liveness Detection |
| :---: | :---: | :---: |
| <img width="793" height="1600" alt="image" src="https://github.com/user-attachments/assets/0c8c9db9-bdb8-43d2-910f-27c45f26faed" />   |  <img width="864" height="1536" alt="WhatsApp Image 2026-06-02 at 2 34 39 PM" src="https://github.com/user-attachments/assets/95580575-1824-42c6-9b2f-804fa7db5c35" />   |  <img width="778" height="1600" alt="image" src="https://github.com/user-attachments/assets/8ce43537-1f66-400b-a2ad-506c68604b1d" />   |

---

## 🎥 Video Demonstration
**Watch the App in Action:** [NHAI FaceRec Demo](https://youtube.com/shorts/KHF1hko_ra0)

---

## 🌟 Advanced Features 

Beyond the mandatory requirements, NHAI FaceRec includes several high-impact features demonstrated in our video:

### 🛡️ Multi-Stage Liveness & Anti-Spoofing
Our liveness flow goes beyond simple blinks. It includes:
- **Sensory Challenges:** Align → Blink → Smile → Turn Head.
- **3D Face Mesh:** Real-time 468-landmark tracking to ensure 3D presence.
- **Passive Liveness:** Integrated texture analysis to block high-res screen replays.

### 👷 PPE Safety Compliance (Integrated AI)
Utilizing a custom **YOLOv8-Nano** model, the app automatically verifies:
- **Helmet Detection:** Ensuring workers are wearing safety headgear.
- **High-Visibility Vest:** Verifying safety vest compliance during attendance.

### 🌐 Connectivity-Independent Sync
- **BLE Mesh Sync:** In zero-network zones, workers' devices can sync attendance logs via Bluetooth Low Energy (BLE), creating a decentralized ledger.
- **DataLake Secure Ledger:** A transparent, audit-ready log of all verifications, including similarity scores and geofencing coordinates.

### 🔒 Privacy & Security
- **Local Differential Privacy (LDP):** Adds mathematical noise to embeddings to protect identity while maintaining accuracy.
- **Secure Enclave Storage:** All biometric data is encrypted using AES-256 and stored in the device's hardware-backed secure storage.

---

## 📊 Performance Benchmarks
- **Detection Latency:** ~15ms
- **Embedding Extraction:** ~110ms
- **PPE Safety Check:** ~45ms
- **Total E2E Pipeline:** ~170ms (Target: <1000ms)

---

## 🏗 System Architecture & Datalake 3.0 Integration

## Architecture Diagram 
<img width="1600" height="453" alt="WhatsApp Image 2026-06-02 at 2 55 03 PM" src="https://github.com/user-attachments/assets/b2c2574a-5695-4927-af4d-953206a3336c" />  

NHAI FaceRec is architected as a modular plug-in for the **Datalake 3.0** ecosystem, ensuring seamless integration and data migration.

---

## ⚖️ License
Licensed under the Apache License, Version 2.0. Models owned by sirius-ai and Google MediaPipe.
