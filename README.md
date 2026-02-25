# Lightweight CNN-based Drone Detection Framework in Low-SNR RF Environments
 
> **First and Foremost** > This repository contains only publicly shareable materials and code snippets because the associated research paper is currently under submission.  
> Therefore, we provide:
> - Public / hybrid dataset structures  
> - Model architecture configurations and pre-trained weights (Phase 1)  
> - Outputs, evaluation metrics, and demonstrations

**Authors:** 
- Dong Nguyen Khanh Duy - Computer Engineering - K23 HCMUT
- Tran Tan Hai - Computer Engineering - K23 HCMUT

**Advisor:** Vo Tuan Binh

**Note:**
- Our repository (will be made fully public after our paper is accepted): https://github.com/Mouse2204/MicroDoppler
- You can find the model configuration (including SiLU and Focal Loss implementations) in the `/models` folder.
- Evaluation metrics across different SNR levels are located in the `/output` folder.

---

## 1. Overview

![RF signal spectrogram showing drone and Wi-Fi interference](path/to/your/spectrogram_image.png)

This project implements a **Data-Driven RF Drone Detection Framework** designed to distinguish:
- Genuine Unmanned Aerial Vehicles (UAVs) based on RF control signals (AR, Bebop, Phantom).
- Real-world Background RF activities (Wi-Fi, Bluetooth, ambient noise).

The system is motivated by the critical limitation of traditional detection methods: performance degradation in ultra-low Signal-to-Noise Ratio (SNR) environments. Instead of relying on heavy mathematical filters that might erase weak drone signals, this framework utilizes a **Lightweight Convolutional Neural Network (CNN)** optimized for Edge AI, trained to "see through" real-world RF interference.

---

## 2. Problem Motivation

Traditional UAV detection techniques using RF signals often rely on:
- Ideal White Gaussian Noise (AWGN) assumptions.
- Heavy deep learning backbones (ResNet, VGG) that are not suitable for embedded deployment.
- Hard DSP filtering (like CFAR) which can accidentally suppress weak drone signals at low SNRs.

However, in real-world scenarios:
- Background noise consists of bursty interference (Wi-Fi, Bluetooth) rather than smooth mathematical noise.
- Deep learning models tend to suffer from "Catastrophic Forgetting" or "Noise Memorization," where the AI learns to identify the background interference instead of the actual drone signature.

This raises an important question:
> *How can we design an edge-friendly AI model that learns the intrinsic physical features of a drone's RF signature without memorizing the surrounding real-world noise?*

---

## 3. Core Principle

To address these challenges, our framework treats drone detection as a **Robust Feature Extraction problem under Extreme Interference**. 

Real drone signals (Frequency Hopping, OFDM) leave specific spatial patterns on a spectrogram. By employing **Curriculum Learning** and injecting **Real RF Background Noise**, we force the CNN to discard useless background patterns (Information Gain = 0) and focus strictly on the microscopic ridges of the drone's RF signature. 

---

## 4. Pipeline Architecture

The proposed system consists of four sequential stages:

1. **RF to Spectrogram Transformation** 2. **Hybrid Dataset Generation (Alpha Blending)** 3. **Edge-Optimized Lightweight CNN Configuration** 4. **Two-Phase Curriculum Learning (Hybrid Training)** Each stage is engineered to maximize robustness at low SNRs (-15 dB) while maintaining a minimal computational footprint.

---

## 5. Stage Descriptions

### Stage 1 – RF to Spectrogram Transformation
Raw 1D RF signals are highly volatile. 
- We utilize Short-Time Fourier Transform (STFT) to convert 1D signals into 2D time-frequency spectrograms.
- Power is converted to the dB scale to highlight microscopic frequency ridges.
- Result: A grayscale 2D image optimized as input for spatial feature extraction.

### Stage 2 – Hybrid Dataset Generation
Instead of using synthetic AWGN, we utilize actual RF Background activities.
- **Alpha Blending:** Drone signals are mixed with real Wi-Fi/Bluetooth noise across a wide SNR range (-18 dB to +11 dB).
- **Hard-example mining:** We ensure the dataset accurately reflects real-world overlapping interference, making the detection task significantly harder but highly realistic.

### Stage 3 – Edge-Optimized Lightweight CNN

![Lightweight Convolutional Neural Network architecture](path/to/your/cnn_architecture_image.png)

We designed a custom CNN from scratch, avoiding heavy backbones.
- **Architecture:** 4 consecutive blocks of `Conv2D -> BatchNorm2d -> SiLU -> MaxPool2d`.
- **SiLU Activation:** Replaces traditional LeakyReLU to provide smooth non-monotonicity, preventing "dying neurons" and improving gradient flow at low SNRs.
- **Focal Loss:** Replaces Cross-Entropy to dynamically scale down the penalty for easily classified high-SNR samples and aggressively focus the model on hard-to-detect low-SNR samples.
- **Complexity:** Sub-1M parameters, making it highly suitable for Edge/Embedded devices (Jetson Nano, Raspberry Pi).

### Stage 4 – Two-Phase Curriculum Learning
To prevent the CNN from memorizing noise or suffering from catastrophic forgetting, we implement a strategic training loop:
- **Phase 1 (Warm-up):** The model is trained purely on high-SNR and clean drone signals to learn the core spatial representation of the UAVs.
- **Phase 2 (Robust Fine-tuning):** The model weights are transferred and trained on a Multi-SNR Hybrid Dataset. Crucially, **Pure Noise samples (Class 0)** are mixed into every batch. This forces the model to mathematically cancel out Wi-Fi/Bluetooth features and focus only on the drone.

---

## 6. Dataset

This repository utilizes an extended version of the publicly available DroneRF dataset.
- **Total Samples:** > 45,000 hybrid spectrogram images.
- **Class 1 (Drone):** Mixed Drone signals at various SNRs.
- **Class 0 (Pure Noise):** Clean noise and Real RF background activities.
- **Split:** 80% Train / 20% Validation (Stratified).

---

## 7. Experimental Performance (Summary)

Our lightweight framework demonstrates State-of-the-Art robustness against real RF interference:
- **Max Validation Accuracy:** ~89.27% across the entire hybrid dataset.
- **High-SNR Performance (0 dB to +10 dB):** > 91% to 95%.
- **Low-SNR Performance (-15 dB):** Maintains ~79.07% accuracy, significantly outperforming random guessing in an environment where the signal is virtually invisible to the human eye.

By utilizing Real RF noise instead of AWGN, our model promises far fewer false positives in actual deployment scenarios.

---

## 8. Contributions

This work introduces:
- A highly practical Drone Detector validated against **Real Background RF Noise**, moving beyond ideal AWGN assumptions.
- An **Edge-ready Lightweight CNN** (< 1M params) leveraging SiLU activation and Focal loss for ultra-low latency.
- A **Two-Phase Hybrid SNR Training Strategy** that acts as a robust defense against AI noise-memorization and catastrophic forgetting.

---

## 9. Limitations & Future Work

Planned extensions for our research include:
- Establishing a baseline comparison using mathematical AWGN (following the standard $P_n$ addition formula) for direct cross-evaluation with existing literature.
- Conducting hardware deployment metrics (measuring exact FLOPs and inference latency in milliseconds) on NVIDIA Jetson or similar embedded boards.

---

## 10. Citation

If referencing this work, please cite the associated paper  
(currently under submission).
