# RRPIPs: Respiratory Waveform Reconstruction using Persistent Independent Particles Tracking from Video

This is the official code release for the RRPIPs framework accepted in CHASE 2025, "RRPIPs: Respiratory Waveform Reconstruction using Persistent Independent Particles tracking from video".

---

## Table of Contents

- [Overview](#overview)
- [Motivation & Background](#motivation--background)
- [Requirements](#requirements)
- [Methodology](#methodology)
  - [Stage 1: Respiratory Region Localization](#stage-1-respiratory-region-localization)
  - [Stage 2: Quality Respiratory Motion Localization](#stage-2-quality-respiratory-motion-localization)
  - [Stage 3: Fine-Scale Tracking & Waveform Extraction](#stage-3-fine-scale-tracking--waveform-extraction)
  - [RRPIPs Model Development](#rrpips-model-development)
- [Datasets & Data Preparation](#datasets--data-preparation)
- [Experiments & Results](#experiments--results)
  - [Comparative Analysis](#comparative-analysis)
- [Discussion & Future Directions](#discussion--future-directions)
- [Conclusion](#conclusion)
- [References](#references)

---

## Overview

RRPIPs is a novel framework designed to accurately estimate respiratory waveforms from video modalities (RGB, NIR, IR) without contact. By treating video-based respiratory rate estimation as a *Tracking-All-Points (TAP)* problem, our coarse-to-fine pipeline is able to:

- **Detect and localize subtle respiratory motions** using video magnification and optical flow.
- **Refine tracking with a Signal Quality Index (SQI)** to pinpoint high-quality respiratory signals.
- **Extract robust respiratory waveforms** that are clinically relevant for assessing breathing depth, timing, and consistency.
![Figure 1: An overview of the RRPIPs pipeline showing the coarse-to-fine stages—from motion magnification and ROI extraction to SQI-based point selection and fine-scale tracking.](figures/1.pdf)

---

## Motivation & Background

### Respiratory Rate (RR) Importance
- RR is a critical vital sign used for detecting conditions such as apnea, respiratory depression, and stress-induced changes.

### Current Challenges
- Existing methods mainly focus on estimating RR without reconstructing the full waveform, leaving a gap in capturing detailed respiratory parameters such as breathing depth and consistency.

### Our Approach
- We reformulate the problem as tracking every point that exhibits respiratory-induced motion across frames—addressing multi-modal inputs and low signal-to-noise challenges.

![Figure 1: Respiratory rate mechanism during inhale and exhale phases; video cameras capture subtle respiration-induced movements.](figures/2.pdf)

---

## Requirements

The lines below should set up a fresh environment with everything you need:

```bash
conda create -n rrpips python=3.8
conda activate rrpips
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
pip install -r requirements.txt
```

Additional dependencies:

- OpenCV (for video processing)
- SciPy (for signal processing)
- DeepMag (for motion magnification)
- RAFT (for optical flow estimation)

---

## Methodology

### Stage 1: Respiratory Region Localization

**Motion Magnification:**  
- Amplifies subtle movements using techniques like DeepMag to enhance detection for the subsequent RAFT optical flow estimation.

**Optical Flow Estimation:**  
- Uses the RAFT model to compute dense pixel displacements and generates heatmaps to isolate high-motion regions (ROIs).

![Figure 3: Optical flow magnification results showing the 1st frame, 2nd frame, and the computed optical flow magnitude between original and magnified frames.](figures/3.pdf)

---

### Stage 2: Quality Respiratory Motion Localization

**Coarse-Scale Tracking:**  
- Selected ROIs are upscaled and tracked using the RRPIPs model to obtain initial point trajectories.

**Signal Quality Index (SQI):**  
- Evaluates the SNR of these trajectories by computing the Power Spectral Density (PSD) and ranks points based on signal energy in the respiratory frequency band.

**Outcome:**  
- Selection of the best points for final waveform extraction.

![Figure 4: Signal Quality Index (SQI) framework for point selection based on respiratory signal quality, showing waveform extraction, PSD computation, and ranking.](figures/4.pdf)

---

### Stage 3: Fine-Scale Tracking & Waveform Extraction

**Region Upscaling:**  
- Windows around the selected points are cropped and upscaled by a factor of eight to enhance resolution.

**Fine-Scale Point Tracking:**  
- Multi-frame tracking is performed on these upscaled regions to capture fine, non-rigid respiratory motions.

**Respiratory Waveform Extraction:**  
- Point trajectories are filtered (using a bandpass filter and PCA) to isolate and extract the dominant respiratory waveform.

![Figure 5: Steps to extract the respiratory waveform from point trajectories, illustrating the fine-scale tracking and subsequent waveform extraction process.](figures/5.pdf)

---

### RRPIPs Model Development

**Adaptation of PIPs++:**  
- The model refines the PIPs++ architecture for multimodal data (RGB, NIR, IR) by integrating multi-scale tracking mechanisms.

**Training & Tracking Operation:**  
- Fine-tuning is performed on a curated dataset using full and component-specific strategies, with iterative processing of consecutive frame sequences to maintain tracking continuity.

**Key Advantages:**  
- Robust handling of non-rigid motion and low-texture regions, resulting in high-fidelity waveform reconstruction with low MAE and RMSE.

![Schematic: A diagram of the RRPIPs model architecture highlighting its multi-frame, multi-scale tracking operations.](figures/6.pdf)

---

## Datasets & Data Preparation

**In-House Dataset:**  
- 38 video sessions (3–5 minutes each) from 29 adult volunteers, capturing multiple views (shoulders, abdomen, chest) under diverse conditions and synchronized with ground truth from a Vernier Go Direct Respiration Belt.

**Public Datasets:**  
- **AIR-125:** Infant RGB videos with RR annotations.  
- **Sleep Database:** Dual-mode (NIR and IR) videos of adult subjects.

**Data Annotation:**  
- A semi-supervised, human-in-the-loop process using RAFT and optical flow-based techniques ensures precise point tracking and trajectory annotation.

![Figure 8: Cropped sample frames from our in-house dataset illustrating the diversity of subjects, camera angles, and regions of interest.](figures/7.1.pdf)

![Figure 9:  (a) Diversity in RR ranges (b) Diversity in Subjects and Camera POV and exposed RR induced organs](figures/7.2.pdf)

![Figure 9: A sample data Collection Setting. The camera sensor captures the respiratory movements and the Vernier Belt provides ground truth respiration pressure. The imaginary white line acts as a visualization reference to highlight a subtle abdomen movement.](figures/7.3.pdf)

---

## Experiments & Results

### Comparative Analysis

We evaluated RRPIPs against several baseline methods and state-of-the-art approaches. Key metrics include:

| **Method**            | **MAE (breaths/min)** | **RMSE (breaths/min)** |
|-----------------------|-----------------------|------------------------|
| Intensity-Based RR    | 3.25                  | 5.12                   |
| Optical Flow (RAFT)   | 2.42                  | 3.89                   |
| Edge Shift Tracking   | 2.52                  | 5.53                   |
| PIPs++                | 1.62                  | 2.92                   |
| **RRPIPs (Proposed)** | **1.01**              | **1.80**               |

**Performance Highlights:**
- RRPIPs achieves state-of-the-art accuracy with a lower MAE and RMSE.
- It shows consistent performance across RGB, NIR, and IR modalities.
- Extensive ablation studies confirm the effectiveness of each pipeline component.

---

## Discussion & Future Directions

**Interpretability:**  
- Visual tracking from ROI localization to final waveform extraction provides enhanced clinical interpretability.

**Robustness:**  
- The framework efficiently handles low-SNR conditions and varying modalities.

**Future Work:**  
- Extending to dynamic scenarios (e.g., subject or camera motion, speech artifacts).
- Adaptation for deployment on edge devices.
- Enhancing motion separation with global motion analysis (GMA).

![Figure 16: Movement pattern similarity in time and frequency domains for points over the RR pixels, demonstrating the robustness of the tracking.](figures/9.pdf)

---

## Conclusion

**Summary:**  
- Our work bridges the gap in video-based RR estimation by reconstructing detailed respiratory waveforms.
- The RRPIPs framework employs a robust coarse-to-fine pipeline that significantly outperforms existing methods.

**Impact:**  
- This approach lays the groundwork for advanced contactless respiratory monitoring systems with applications in clinical, sports, and home-care settings.

![Figure 17: Sample tracking in original video resolutions, showing the final selected points for respiratory waveform extraction. Top: Subject 1 tracking points. Bottom: Subject 2 tracking points.](figures/10.1.pdf)

![Figure 17: Sample tracking in original video resolutions, showing the final selected points for respiratory waveform extraction. Top: Subject 1 tracking points. Bottom: Subject 2 tracking points.](figures/10.2.pdf)

## Citation
If you use this code for your research, please cite:
```
@inproceedings{rrpips2025,
    author = {Zahid Hasan and Masud Ahmed and Shadman Sakib and Snehalraj Chugh and Md Azim Khan and Abu Zaher MD Faridee and Nirmalya Roy},
    title = {RRPIPs: Respiratory Waveform Reconstruction using Persistent Independent Particles tracking from video},
    booktitle = {CHASE},
    year = {2025}
}
```
