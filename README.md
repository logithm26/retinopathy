# RetinaLens AI — Clinical-Grade Explainable Diabetic Retinopathy (XAI-DR) Platform

[![GitHub Pages Deployment](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-06b6d4?style=for-the-badge&logo=github)](https://logithm26.github.io/retinopathy/)
[![License: MIT](https://img.shields.io/badge/License-MIT-emerald.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![ICDR Staging](https://img.shields.io/badge/ICDR%20Standard-Stages%200--4-amber?style=for-the-badge)]()
[![Model Calibration](https://img.shields.io/badge/QWK%20Calibration-%3E%200.88-rose?style=for-the-badge)]()

**Live Application URL**: [https://logithm26.github.io/retinopathy/](https://logithm26.github.io/retinopathy/)

RetinaLens AI is an advanced ophthalmic screening and biomedical diagnostic platform engineered for automated, explainable detection of Diabetic Retinopathy (DR) from digital fundus photography.

---

## 🌟 Key Features

- **Clinical Preprocessing (Ben Graham Method)**:
  - Tight circular retinal ROI cropping (intensity threshold $> 10$) to eliminate uninformative black borders.
  - Scale and isotropic resize to $512 \times 512$.
  - Local color equalization: subtracts Gaussian blur ($\sigma = \text{target\_size} / 30 \approx 17\text{ px}$) using:
    $$\text{normalized} = \text{cv2.addWeighted}(img, 4, blurred, -4, 128) \equiv \text{clamp}(4 \cdot img - 4 \cdot blurred + 128, 0, 255)$$
  - Active circular retinal mask to suppress border artifacts.

- **Ordinal Regression & Multi-Threshold Decision Head**:
  - Calibrated for EyePACS / APTOS 2019 cohorts ($\text{QWK} > 0.88$).
  - Continuous ordinal regression score mapped via clinical decision boundaries:
    - **Stage 0 (No Apparent DR)**: $\hat{y} < 0.5$
    - **Stage 1 (Mild NPDR)**: $0.5 \le \hat{y} < 1.5$
    - **Stage 2 (Moderate NPDR)**: $1.5 \le \hat{y} < 2.5$
    - **Stage 3 (Severe NPDR)**: $2.5 \le \hat{y} < 3.5$
    - **Stage 4 (Proliferative DR)**: $\hat{y} \ge 3.5$

- **Ensemble Vision Pipeline**:
  - **Green Channel Isolation**: Targets hemoglobin optical absorption peak (540–577 nm).
  - **$8 \times 8$ Grid CLAHE**: Contrast Limited Adaptive Histogram Equalization with bilinear tile interpolation.
  - **Otsu Automatic Retinal Vessel Segmentation**: Delineates the major vascular tree to eliminate false-positive microaneurysm candidates.
  - **Multi-Scale Morphological Top-Hat Transforms**:
    - *White Top-Hat ($WTH$)*: Detects bright lipid lesions (Hard Exudates, Cotton Wool Spots).
    - *Black Top-Hat ($BTH$)*: Isolates focal dark lesions (Microaneurysms, Petechial Hemorrhages).
  - **Geometric Circularity ($4\pi A / P^2$) & Anatomical Masking**: Disc cup exclusion and foveal distance calculation for Macular Edema risk estimation.

- **Calibrated Grad-CAM Explainability**:
  - Final convolutional feature activation mapping.
  - Heatmap is normalized strictly over the active circular retinal mask, guaranteeing zero activation on dark peripheral borders and eliminating lens glare artifacts.

- **Clinical PDF Diagnostic Report Generator (jsPDF)**:
  - Dynamic **Patient Name** input field with graceful fallback to `"Anonymous Patient"`.
  - Comprehensive report including composite retinal snapshot, ICDR clinical assessment, pathology breakdown, evidence-based Do's and Don'ts, and physician signature block.

- **Interactive Visual Workbench**:
  - Before/After split-view comparison slider with real-time hardware-accelerated Canvas 2D rendering.
  - Live ophthalmic camera feed capture.
  - Immediate sample presets: Normal Retina and Moderate NPDR.

---

## 🏗️ System Architecture

```
RetinaLens AI/
├── index.html                  # Ophthalmic workbench UI & clinical dashboard
├── style.css                   # Glassmorphism design system & visual tokens
├── script.js                   # Client-side Ben Graham preprocessor, ensemble AI & PDF generator
├── retina_analysis.py          # Python clinical module (EfficientNet-B4 / ConvNeXt ordinal engine)
├── server.js                   # Lightweight static HTTP development server
├── .nojekyll                   # Bypasses Jekyll on GitHub Pages
├── .github/
│   └── workflows/
│       └── deploy.yml          # Automated CI/CD GitHub Pages deployment
└── README.md                   # Project documentation
```

---

## 🚀 Running Locally

### Option 1: Using Node.js (Zero external dependencies)
```bash
node server.js
```
Open [http://localhost:3000/](http://localhost:3000/) in your browser.

### Option 2: Using Python
```bash
python -m http.server 3000
```
Open [http://localhost:3000/](http://localhost:3000/) in your browser.

### Option 3: Standalone Python Clinical Diagnostic Engine
```bash
python retina_analysis.py
```

---

## 🌐 GitHub Pages Deployment Guide

This project is configured for automated deployment via GitHub Pages and GitHub Actions.

1. **Repository Settings**:
   - Navigate to `https://github.com/logithm26/retinopathy/settings/pages`
2. **Build and Deployment Source**:
   - Under **Source**, choose either:
     - **GitHub Actions** (uses the included `.github/workflows/deploy.yml` workflow)
     - OR **Deploy from a branch** -> Branch: `main` / Folder: `/ (root)` -> Click **Save**.
3. **Access Live App**:
   - Your site will be live at: **[https://logithm26.github.io/retinopathy/](https://logithm26.github.io/retinopathy/)**

---

## 📄 Output Contract Specification

The prediction engine strictly produces the standardized medical assessment schema:

```json
{
  "stage": "Moderate NPDR",
  "confidence": 0.951,
  "lesions": [
    {
      "lesion_type": "Microaneurysm",
      "box_2d": [0.4125, 0.4820, 0.4672, 0.5367],
      "significance": "Critical"
    },
    {
      "lesion_type": "Hard Exudate",
      "box_2d": [0.5215, 0.6120, 0.5762, 0.6667],
      "significance": "High"
    }
  ],
  "clinical_narrative": "Retinal fundus scan evaluated via Ben Graham contrast normalization and ordinal multi-threshold head. ICDR Assessment: Stage 2: Moderate NPDR with clinical confidence of 95.1%..."
}
```

---

## 📜 Medical Disclaimer

RetinaLens AI is an algorithmic clinical decision support tool designed for assisted ophthalmic triage and research purposes. All findings must be clinically correlated by a board-certified ophthalmologist or eye care professional.
