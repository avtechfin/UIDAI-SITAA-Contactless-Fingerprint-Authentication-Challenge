# Fingerprint Image Processing API (Flask + OpenCV)

## Overview

This project implements a **server-side fingerprint image preprocessing and ridge extraction pipeline** exposed via a RESTful Flask API. The application ingests a color fingerprint image (typically captured via a mobile camera), performs **skin-based finger segmentation**, **contrast normalization**, **ridge enhancement**, **adaptive binarization**, and **morphological thinning**, and outputs a **DPI-normalized binary fingerprint image** suitable for downstream biometric analysis.

The system is designed for **contactless fingerprint acquisition scenarios**, where illumination is uncontrolled and images suffer from non-uniform lighting, motion blur, and background clutter.

---

## Key Features

* HSV + YCrCb based **skin-tone finger segmentation**
* Robust finger **ROI localization and cropping**
* Flash-like grayscale normalization using **CLAHE and bilateral filtering**
* Ridge enhancement using **Black-Hat morphology**
* Adaptive local thresholding for ridge binarization
* Skeletonization via **iterative morphological thinning**
* DPI-aware image export (PNG / TIFF / JPEG)
* Stateless REST API suitable for containerized deployment

---

## Technology Stack

| Component              | Description                     |
| ---------------------- | ------------------------------- |
| Python 3.11+            | Core runtime                    |
| Flask                  | REST API framework              |
| OpenCV (cv2)           | Image processing and morphology |
| NumPy                  | Numerical operations            |
| scikit-image           | Skeletonization (thinning)      |
| Pillow (PIL)           | Image export with DPI metadata  |
| Gunicorn (recommended) | Production WSGI server          |

---

## System Architecture

```
Client (Mobile / Browser)
        |
        |  HTTP POST (multipart/form-data)
        v
Flask REST API
        |
        v
Fingerprint Processing Pipeline
        |
        v
Binary Ridge Image (500 DPI)
```

---

## Image Processing Pipeline

### 1. Finger Segmentation (HSV + YCrCb Fusion)

* Converts input image from **BGR → HSV** and **BGR → YCrCb**
* Applies empirically tuned skin-color thresholds
* Combines masks using logical AND / OR fallback
* Morphological cleanup using:

  * Median filtering
  * Morphological closing (hole filling)
  * Morphological opening (noise removal)
* Extracts the **largest connected contour** as the finger region
* Computes bounding box for ROI extraction

> This dual-color-space approach improves robustness across different skin tones and illumination conditions.

---

### 2. ROI Cropping with Margin

* Expands bounding box by a configurable margin
* Prevents ridge truncation near finger boundaries
* Ensures spatial consistency during resizing

---

### 3. Flash-Like Grayscale Normalization

Simulates the effect of a controlled flash sensor:

* Grayscale conversion
* Low-frequency background estimation using Gaussian blur
* High-frequency emphasis via unsharp masking
* Contrast Limited Adaptive Histogram Equalization (CLAHE)
* Edge-preserving denoising using bilateral filtering

This stage enhances ridge–valley contrast while suppressing illumination gradients.

---

### 4. Ridge Enhancement (Black-Hat Morphology)

* Applies **Black-Hat morphological operation** using elliptical structuring elements
* Extracts dark ridge structures against lighter background
* Robust percentile-based contrast stretching (2%–98%)
* Secondary CLAHE for local ridge contrast boosting
* Mild Gaussian smoothing to suppress high-frequency noise

---

### 5. Adaptive Ridge Binarization

* Uses **adaptive Gaussian thresholding**
* Operates on local neighborhoods to handle non-uniform illumination
* Optional masking with finger segmentation map
* Morphological post-processing:

  * Closing to connect broken ridges
  * Opening to remove salt noise

Output is a clean binary ridge map.

---

### 6. Skeletonization (Thinning)

* Implements **iterative morphological thinning**
* Produces single-pixel-wide ridge skeletons
* Preserves ridge topology and connectivity
* Suitable for minutiae extraction algorithms

---

### 7. DPI-Aware Export

* Images are exported with embedded **DPI metadata** (default: 500 DPI)
* Supported formats:

  * PNG
  * TIFF (LZW compression)
  * JPEG (lossy, optional)
* Ensures interoperability with AFIS and biometric SDKs
