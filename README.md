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

The ESP32-S3 acts as an intelligent audio gatekeeper: it listens locally and decides when audio needs to leave the device.

🎯 Key Objectives
Perform custom wake-word detection locally on an MCU.
Keep the edge model lightweight using INT8 quantization.
Avoid continuous audio transmission during idle listening.
Stream audio only after local wake-word activation.
Preserve the beginning of commands using a 300–500 ms circular pre-roll buffer.
Offload computationally heavy speech recognition to a remote ASR server.
Use an open-source software stack wherever possible.
Design around the problem statement constraints of:
< 256 KB RAM
< 10% idle CPU utilization
Validate the system through actual hardware profiling rather than theoretical estimates.
🧠 Core Architecture
1. Local Audio Capture

An INMP441 digital MEMS microphone captures audio and provides digital data to the ESP32-S3 through the I2S interface.

2. Audio Feature Extraction

Raw audio is converted into lightweight acoustic features such as:

Log-Mel Spectrograms
MFCCs

These features provide a compact representation of the audio for keyword spotting.

3. Custom Wake-Word Detection

A custom Depthwise-Separable Convolutional Neural Network (DS-CNN) is used for keyword spotting.

The model is:

Trained using TensorFlow
Optimized using INT8 quantization
Converted for embedded deployment
Executed using TensorFlow Lite for Microcontrollers
4. Local Decision

The ESP32-S3 continuously evaluates the incoming audio locally.

Wake Word?
   │
   ├── No → Continue Listening
   │
   └── Yes → Activate Streaming

No continuous audio stream is sent to the remote ASR system during normal listening.

5. Pre-Roll Audio Buffer

A 300–500 ms circular buffer continuously retains recent audio.

When the wake word is detected:

Pre-Roll Audio + Newly Captured Audio
                    ↓
              Audio Stream

This helps reduce the possibility of losing the beginning of the user's command during the transition to network streaming.

6. Remote Speech Recognition

After activation, audio is streamed over Wi-Fi using a lightweight communication mechanism such as:

WebSocket
Socket-based communication

The remote server performs the computationally intensive speech-to-text operation using a self-hosted open-source ASR system, such as an open-source Whisper implementation.

🛠️ Technology Stack
Component	Technology	Purpose
Edge MCU	ESP32-S3	Local processing and wake-word detection
Microphone	INMP441	Digital audio capture
Audio Interface	I2S	Microphone-to-MCU communication
Features	Log-Mel / MFCC	Lightweight audio representation
KWS Model	DS-CNN	Custom wake-word detection
Training	TensorFlow	Model development
Optimization	INT8 Quantization	Reduce model size and computation
Edge Runtime	TensorFlow Lite Micro	MCU model inference
Firmware	C/C++	Embedded implementation
Network	Wi-Fi	Activated audio transmission
Communication	WebSocket / Socket	Audio streaming
Remote ASR	Open-source Whisper-based ASR	Speech-to-text
📊 Resource-Constrained Design

MicroKWS is designed around strict embedded resource limitations.

Target Constraints
Metric	Target
RAM	< 256 KB
Idle CPU	< 10%
Wake-word model	INT8 optimized
Audio transmission during idle	No continuous ASR streaming
Pre-roll buffer	300–500 ms

Important: These are design targets. Final values will be reported only after physical implementation and profiling on the target hardware.

🎙️ Custom Dataset Strategy

The wake-word model will use a custom dataset designed for real-world robustness.

Positive Samples

Collected across:

Multiple speakers
Different pronunciations
Different speaking distances
Different speaking volumes
Negative Samples

Including:

General speech
Background noise
Similar-sounding words
Non-wake-word conversations
Robustness Techniques
Noise augmentation
Hard-negative mining
Confidence threshold tuning
Temporal confidence smoothing
Real-world acoustic testing

The system will report false activations per hour instead of claiming zero false activations without measurement.

🔬 Implementation Methodology
1. Custom Dataset Creation
             ↓
2. Audio Preprocessing
             ↓
3. DS-CNN Training
             ↓
4. INT8 Quantization
             ↓
5. TFLM Conversion
             ↓
6. ESP32-S3 Deployment
             ↓
7. INMP441 + I2S Integration
             ↓
8. Remote ASR Integration
             ↓
9. Physical Validation
             ↓
10. Optimization & Retesting

The development cycle follows:

TARGET → IMPLEMENT → MEASURE → OPTIMIZE → VALIDATE

📏 Validation Metrics

The final implementation will be evaluated using measurable parameters:

RAM usage
Flash/model footprint
Idle CPU utilization
Wake-word detection performance
False activations per hour
Background-noise robustness
Speaker variation
Distance and volume variation
Wake-word-to-ASR-server latency
Latency Measurement

The primary latency metric is:

Wake-Word Completion
        ↓
Wake-Word Detection
        ↓
Streaming Starts
        ↓
First Audio Packet
        ↓
ASR Server Receives First Audio

The main reported value will be measured from:

Completion of the wake word → Receipt of the first audio data by the remote ASR server

Additional timestamps will help separate local processing, network setup and transmission delays.

🔐 Privacy-Aware Architecture

MicroKWS follows a local-first processing approach.

During normal listening:

Microphone → Local Wake-Word Detection

After activation:

Wake Word → Audio Streaming → Remote ASR

This avoids continuously sending ambient microphone audio to the remote speech-recognition server.

MicroKWS is therefore designed to reduce unnecessary audio transmission, rather than claiming absolute privacy.

⚡ Why This Architecture?
Why not run Whisper locally?

Full speech-recognition models are considerably heavier than a small wake-word model and are not suitable for the intended MCU resource constraints.

MicroKWS separates the tasks:

ESP32-S3
Lightweight KWS
        +
Remote Server
Heavy ASR

This keeps the edge device lightweight while still enabling full speech recognition after activation.

Why DS-CNN?

Depthwise-separable convolutions can reduce the computation and parameter requirements compared with conventional convolutional networks, making the architecture suitable for lightweight keyword spotting.

Why INT8?

INT8 quantization is intended to reduce model size and computational requirements, helping the model fit within embedded resource constraints.

💡 What Makes MicroKWS Different?

The novelty is not DS-CNN alone.

The project combines:

Custom local wake-word detection
INT8 MCU deployment
Resource-aware memory design
300–500 ms pre-roll buffering
Event-driven audio streaming
Remote open-source ASR
Physical resource and latency validation

into a single edge-to-server voice architecture designed specifically for constrained hardware.

The Core Idea

The device does not continuously send everything it hears.

It listens locally, decides locally, and streams only when activated.

🌍 Potential Applications

MicroKWS can be adapted for:

Smart-home devices
Low-power voice interfaces
Wearable devices
Industrial IoT
Safety-oriented embedded systems
Resource-constrained voice-controlled devices
📁 Planned Repository Structure
MicroKWS/
│
├── README.md
│
├── dataset/
│   ├── positive/
│   ├── negative/
│   └── augmented/
│
├── training/
│   ├── preprocessing/
│   ├── train.py
│   ├── evaluate.py
│   └── quantize.py
│
├── model/
│   ├── ds_cnn/
│   ├── tflite/
│   └── int8/
│
├── firmware/
│   └── esp32s3/
│       ├── audio/
│       ├── kws/
│       ├── buffer/
│       ├── network/
│       └── main.cpp
│
├── server/
│   └── asr/
│
├── evaluation/
│   ├── ram/
│   ├── cpu/
│   ├── latency/
│   └── kws/
│
├── docs/
│   ├── architecture/
│   └── methodology/
│
└── demo/
🚧 Current Project Status

Status: In Development

The current repository documents the proposed architecture, methodology and implementation plan.

The following components are planned for implementation and physical validation:

 Custom wake-word dataset
 DS-CNN training
 INT8 quantization
 TFLM deployment
 ESP32-S3 firmware
 INMP441 I2S integration
 Circular pre-roll buffer
 Wi-Fi audio streaming
 Remote open-source ASR
 Physical RAM/CPU profiling
 Latency measurement
 Real-world noise testing

No performance metric is considered achieved until it is measured on the physical implementation.

👥 Team

Team: Hive Minds
SIH 2026 Team ID: 174835
Problem Statement: 26172
Organization: ISRO – Department of Space
Theme: Smart Automation
Category: Hardware

📜 License

This project is intended to use an open-source technology stack. Individual dependencies and models remain subject to their respective licenses.

🔑 MicroKWS in One Line

A tiny edge gatekeeper that detects the wake word locally and streams speech to remote ASR only when the user activates it.


**One important GitHub point:** don't put fake benchmark numbers, fake screenshots, or “working successfully” claims in the repo yet. Since you're submitting the GitHub link alongside the SIH PPT, this README makes the **proposed architecture + implementation status** very clear and defensible.
