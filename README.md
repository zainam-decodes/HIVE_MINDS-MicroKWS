# MicroKWS – Lightweight Edge Voice Activation with On-Demand Remote ASR

> **Listen locally → Decide locally → Stream only when activated → Recognize remotely.**

MicroKWS is a lightweight edge voice-activation system designed for **resource-constrained IoT and embedded devices**.

Instead of continuously sending microphone audio to a remote server or running a heavy speech-recognition model on a microcontroller, MicroKWS performs **custom wake-word detection locally on an ESP32-S3** and activates remote speech recognition only when the wake word is detected.

---

## 🚀 Project Overview

Cloud-first voice systems can continuously transmit microphone audio, resulting in unnecessary network usage, increased latency, power consumption and privacy concerns.

At the other extreme, running complete speech recognition directly on a low-power microcontroller is difficult because of limited RAM and processing capability.

MicroKWS addresses this gap with a **hybrid edge-to-server architecture**:

```text
Microphone
    ↓
ESP32-S3
    ↓
Audio Features
(Log-Mel / MFCC)
    ↓
INT8 DS-CNN
    ↓
Wake Word Detected?
    │
    ├── NO → Continue Local Listening
    │
    └── YES
          ↓
    Pre-Roll + New Audio
          ↓
       Wi-Fi
          ↓
   Remote Open-Source ASR
          ↓
      Text / Command
