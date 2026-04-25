# Gestura

> **AI-powered real-time sign language communication — bridging the gap between deaf and hearing individuals during live meetings and calls.**

Gestura uses computer vision and deep learning to recognize hand gestures and translate them into text or speech in real time, while simultaneously converting spoken language back into text — enabling seamless two-way communication.

---

## What Gestura Does

**Sign-to-Text Translation**
Recognizes ASL hand gestures via computer vision and translates them into readable text, displayed as live subtitles on the video feed.

**Speech-to-Text**
Converts spoken language into text using AI-powered speech recognition, giving deaf users a readable transcript of what hearing participants are saying.

**Live Meeting Integration**
Routes the annotated video feed (with subtitles baked in) to a virtual camera device via PyVirtualCamera and OBS Virtual Camera, making it usable directly inside Zoom, Google Meet, and Microsoft Teams.

---

## How It Works

```
Webcam
  ↓
MediaPipe  →  Landmark Extraction (hands + pose)
  ↓
Sequence Buffer  (60 frames sliding window)
  ↓
Hybrid Model  (Conv1D + BiLSTM + Attention)
  ↓
Temporal Smoother  →  Stable Prediction
  ↓
Subtitle Overlay  →  Virtual Camera  →  Meeting App
```

The model watches a 60-frame window of body and hand landmarks and classifies it into one of 16 ASL words. A temporal smoother filters out noisy single-frame predictions before they reach the subtitle display.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Computer Vision | MediaPipe Holistic |
| Deep Learning | TensorFlow / Keras |
| GUI Framework | PyQt6 |
| Virtual Camera | pyvirtualcam |
| Language | Python 3.10+ |

---

## Current Supported Signs

`hello` · `yes` · `no` · `thank you` · `please` · `sorry` · `stop` · `help` · `what` · `how` · `where` · `when` · `eat` · `drink` · `sleep` · `goodbye`

---

## Project Structure

```
gestura/
├── ASL.py                      # MediaPipe helpers, model architecture, data collection
├── main_GUI.py                 # PyQt6 desktop UI and inference pipeline
├── model_training.py           # Model training script
├── model2.keras                # Trained model weights
├── requirements.txt            # Python dependencies
└── README.md
```
---

## Installation

```bash
git clone https://github.com/yourusername/gestura.git
cd gestura
pip install -r requirements.txt
```

Place your trained `model2.keras` weights file in the root `gestura/` directory, then run:

```bash
python main_GUI.py
```

**Requirements:** `opencv-python`, `mediapipe`, `tensorflow`, `PyQt6`, `numpy`, `pyvirtualcam`

---

## Using the Virtual Camera

1. Install [OBS Studio](https://obsproject.com/) and enable the OBS Virtual Camera.
2. On Linux, install `v4l2loopback` instead.
3. In Gestura, tick **"Virtual Camera (Meetings)"** in the control panel before clicking Start.
4. In your meeting app (Zoom, Meet, Teams), select **"OBS Virtual Camera"** or **"pyvirtualcam"** as your camera source.

---

## Current Development — What's Being Improved

Gestura is actively being upgraded from a proof-of-concept into a production-grade desktop application. Here's what's in progress:

### ✅ Phase 1 — Stable Desktop App Pipeline (In Progress)

The original single-threaded loop (capture → inference → render, all blocking each other) has been replaced with a proper multi-threaded architecture:

- **CaptureThread** — dedicated thread that captures frames at full camera FPS without being blocked by model inference.
- **InferenceThread** — runs MediaPipe landmark extraction and model prediction independently. Uses a sliding window (last 60 frames) instead of waiting for fixed 60-frame chunks, making predictions continuous rather than chunked.
- **VirtualCamThread** — isolated thread for virtual camera output. Previously `pyvirtualcam.send()` was a blocking call on the inference thread, causing lag in meetings. Now it runs independently and never slows down inference.
- **TemporalSmoother** — voting-based filter (7 out of 10 consistent predictions required) that prevents the subtitle text from flickering on single noisy frames.
- **Startup validation** — clean error dialogs if the model file is missing or the camera is unavailable, instead of cryptic thread crashes.
- **Camera selector** — switchable camera index from the UI, no code editing required.

---

### 🔄 Phase 2 — Model Upgrade (Planned)

The current model is a **classifier** — it predicts one word per 60-frame window. This means it can't handle continuous, flowing signing. Phase 2 replaces this with a proper sequence model:

- Rewrite model architecture in **PyTorch** for better research-grade control and easier future iteration.
- Introduce **CTC (Connectionist Temporal Classification)** loss — the same technique used in speech recognition — which removes the need to manually segment signs and allows the model to handle variable-speed, continuous signing.
- **Sliding window inference** as an intermediate step before full CTC, improving responsiveness without requiring re-labelling of the dataset.
- Data augmentation pipeline: speed variation, frame dropout, noise injection to improve robustness.

---

### 🔄 Phase 3 — Language Model Integration (Planned)

Raw model output like `"hello yes stop"` will be passed through a language correction layer:

- Lightweight local option: n-gram model for offline use.
- API option: LLM-based correction to reconstruct natural sentences from predicted word sequences.

---

### 🔄 Phase 4 — Expanded Vocabulary & Dataset (Planned)

- Expand from 16 words to full conversational phrases.
- Build a structured data collection pipeline with augmentation.
- Explore publicly available ASL landmark datasets for pre-training.

---

## Roadmap Summary

| Phase | Focus | Status |
|---|---|---|
| Phase 1 | Stable multi-threaded desktop app | Completed |
| Phase 2 | PyTorch model + CTC training pipeline | 🔄 In Progress |
| Phase 3 | Language model correction layer | 🔄 In Progress |
| Phase 4 | Expanded vocabulary + dataset pipeline | 📋 Planned |

---
