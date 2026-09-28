# OnionGuard AI – AI-Based Onion Quality Assessment and Grading System

[![React](https://img.shields.io/badge/Frontend-React%2019%20%2B%20TypeScript-blue)](https://react.dev/)
[![Vite](https://img.shields.io/badge/Build%20Tool-Vite-646CFF)](https://vitejs.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Styling-Tailwind%20CSS%20v4-38B2AC)](https://tailwindcss.com/)
[![License](https://img.shields.io/badge/License-MIT-emerald.svg)](LICENSE)

> **OnionGuard AI** is a modern, field-ready agricultural computer vision web application engineered to assess onion lot quality objectively. It features an isolated **Two-Stage Computer Vision Pipeline** that enforces strict non-onion screening (guaranteeing that human faces, fruits, and random objects are strictly rejected) before performing granular defect segmentation and AGMARK commercial grading.

---

## 📌 Problem Statement

Quality assessment and grading of onions is often subjective, manual, and varies widely across procurement centers (APMC mandis), causing severe payment disputes, transit wastage, and unfair deductions for farmers.

## 💡 Expected Solution & Key Features

1. **Two-Stage AI Validation Gate**:
   - **Stage 1 (Presence Gatekeeper)**: Confirms whether the image contains an onion bulb with confidence $\ge 70\%$. Rejects human faces, fruits, animals, and manufactured objects.
   - **Stage 2 (Defect Segmentation)**: Quantifies healthy, damaged, rotten (Aspergillus black mold), sprouted, diseased (purple blotch), and undersized onions.
2. **Zero False Positives on Human Faces**:
   - Strictly halts execution if a human face or non-onion is detected. Never generates onion quality metrics on rejected images.
3. **Automated Commercial Grading Engine**:
   - Calculates **Grade A Yield %**, **URS (Under-Ripe / Standard) %**, Defect Rate, and Marketability Category per AGMARK / APMC rules.
4. **Digital Quality Assessment Certificate**:
   - Generates tamper-evident digital reports complete with QR verification code, cryptographic hash, lot metadata, and one-click PDF export.
5. **Zero Database Requirement**:
   - Runs seamlessly on mobile and desktop web browsers without requiring MongoDB, PostgreSQL, or Firebase.
6. **Modular AI Model Connectors**:
   - Supports Demo AI mode, external REST API / YOLOv8 endpoints, Roboflow API, and ONNX Runtime.

---

## 🛠️ Technology Stack

- **Framework**: React 19 + TypeScript
- **Bundler & Dev Server**: Vite
- **Styling & Design System**: Tailwind CSS v4 + Plus Jakarta Sans Typography
- **Icons**: Lucide React
- **Data Visualizations**: Recharts (Donut Pie Charts, Defect Bar Charts)
- **Routing**: React Router v7 (`react-router-dom`)
- **PDF Generation**: `jspdf` + `html2canvas` (client-side export)

---

## 🚀 Quick Start Guide

### Prerequisites
- Node.js (v18.0 or higher)
- npm or yarn

### Installation
```bash
# Clone or navigate to the project directory
cd "onion hackkkk"

# Install all dependencies
npm install
```

### Running Locally (Development)
```bash
npm run dev
```
Open your browser and navigate to `http://localhost:5173`.

### Production Build & Preview
```bash
# Build production bundle
npm run build

# Preview production build locally
npm run preview
```

---

## ⚙️ Environment Variables

Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```

Configuration variables available in `.env`:
```ini
# Optional External Computer Vision API Endpoint
VITE_AI_API_URL=https://api.roboflow.com/onion-quality-v1
VITE_AI_API_KEY=your_secure_api_key_here

# Default Stage 1 Confidence Gate Threshold (0.0 to 1.0)
VITE_DEFAULT_ONION_CONFIDENCE=0.70

# Commercial Standard Version
VITE_GRADING_STANDARD=AGMARK_2026_COMMERCIAL
```

---

## 🧠 AI Model Integration Guide

All AI model services are cleanly decoupled from the UI inside `src/services/ai/`:

```
src/services/ai/
  ├── imageValidator.ts   # Checks file type, resolution, and Laplacian sharpness
  ├── onionDetector.ts    # STAGE 1: Gatekeeper (Onion vs Face/Fruit/Object)
  ├── qualityAnalyzer.ts  # STAGE 2: Defect segmentation & bounding box instances
  └── aiService.ts        # Unified pipeline runner orchestrator
```

### 1. Connecting an External Python / YOLOv8 REST API
In the app UI, click the **Settings icon** (top right) or configure `src/services/ai/onionDetector.ts` to point to your FastAPI / Flask endpoint:

```json
POST http://localhost:8000/detect
Body: { "image": "<base64_data_or_url>", "threshold": 0.70 }
Response: {
  "isOnion": true,
  "confidence": 0.94,
  "primaryObject": "Red Onion",
  "detectedObjects": [...]
}
```

### 2. Connecting Roboflow Hosted Models
Set `modelType: 'roboflow'` in Settings and enter your Roboflow Model ID and Private API Key.

---

## 🔧 Key Configuration Questions

### 1. How to change the Onion Confidence Threshold?
- **In UI**: Click the **Settings icon** (top-right) and drag the **Stage 1 Confidence Threshold** slider (40% – 95%).
- **In Code**: Edit `DEFAULT_ONION_CONFIDENCE_THRESHOLD` in `src/data/gradingRules.ts`:
  ```ts
  export const DEFAULT_ONION_CONFIDENCE_THRESHOLD = 0.70; // 70% baseline
  ```

### 2. How to change Grading Rules?
- Edit `DEFAULT_GRADING_THRESHOLDS` in `src/data/gradingRules.ts` or adjust parameters dynamically in `src/services/grading/gradingEngine.ts`:
  ```ts
  export const DEFAULT_GRADING_THRESHOLDS = {
    minGradeAHealthyPercent: 70, // Min healthy % for Grade A
    maxGradeARotPercent: 4,      // Max rot % for Grade A
    maxGradeADamagePercent: 12,  // Max mechanical damage % for Grade A
    maxGradeASproutPercent: 6,   // Max sprout % for Grade A
    minGradeBHealthyPercent: 50, // Min healthy % for Grade B
    maxGradeBRotPercent: 12,     // Max rot % for Grade B
    ursMaxDefectPercent: 35,     // Max defects for URS classification
  };
  ```

### 3. How to test Human-Face & Non-Onion Rejection?
1. Open the **Analyzer** page (`/analyzer`).
2. Click **Demo Samples** or switch to the **Developer Test Bench** (`/testbench`).
3. Select **Sample 7: Human Face (Strict Rejection Test)** or **Sample 8: Apple Fruit**.
4. Observe the Stage 1 Gatekeeper intercept the image:
   ```
   ❌ This image does not appear to contain an onion.
   Detected: Human Face (99.2% confidence).
   Stage 2 quality analysis halted.
   ```
5. Confirm that **zero onion quality metrics** are produced for rejected images.

---

## 🧪 Standard Test Bench Matrix (`/testbench`)

| Test # | Input Image Type | Expected Outcome | Verified By |
| :--- | :--- | :--- | :--- |
| **TEST 1** | Fresh Single Onion | Onion Detected $\rightarrow$ Grade A | ✅ Stage 1 Pass + Stage 2 Quality |
| **TEST 2** | Human Face Portrait | **NOT ONION** $\rightarrow$ Rejection | ⛔ Strict Stage 1 Rejection Gate |
| **TEST 3** | Red Delicious Apple | **NOT ONION** $\rightarrow$ Rejection | ⛔ Strict Stage 1 Rejection Gate |
| **TEST 4** | Blurry / Low-res Image | Validation Error | ⛔ Laplacian Pre-validator |
| **TEST 5** | Multiple Onion Tray (7 units) | Batch Analyzed $\rightarrow$ Defect breakdown | ✅ Multi-instance segmentation |
| **TEST 6** | Damaged / Cut Onion | Damaged Classification | ✅ Stage 2 Mechanical Defect |
| **TEST 7** | Sprouted Onion Bulb | Sprouted Classification | ✅ Stage 2 Vegetative Defect |
| **TEST 8** | Rotten Black Mold Onion | Rotten Classification (Unmarketable) | ✅ Stage 2 Pathogen Defect |
| **TEST 9** | Computer Keyboard / Inanimate | **NOT ONION** $\rightarrow$ Rejection | ⛔ Strict Stage 1 Rejection Gate |

---

## 📂 Project Architecture

```
src/
├── components/
│   ├── common/
│   │   ├── Header.tsx            # Navigation, branding & demo mode indicator
│   │   ├── Footer.tsx            # Compliance disclaimer & footer links
│   │   ├── ConfidenceBadge.tsx   # Color-coded AI confidence badge
│   │   ├── ErrorMessage.tsx      # Stage 1 rejection gate banner
│   │   ├── LoadingState.tsx      # Laser scanning progress tracker
│   │   └── Modal.tsx             # Accessible backdrop dialog
│   ├── analyzer/
│   │   ├── ImageUploader.tsx     # Drag & drop file selector
│   │   ├── CameraCapture.tsx     # Live WebRTC camera snapshot
│   │   ├── ImagePreview.tsx      # Sharpness diagnostics & analyze CTA
│   │   ├── SamplePresetPicker.tsx# 9 sample presets (including negative tests)
│   │   └── ModelConfigDrawer.tsx # Threshold & model configuration modal
│   ├── results/
│   │   ├── ResultsDashboard.tsx  # Unified results dashboard
│   │   ├── DetectionOverlay.tsx  # SVG Bounding boxes & defect labels
│   │   ├── QualityCard.tsx       # KPI Grade cards
│   │   ├── QualityChart.tsx      # Recharts Donut & Bar charts
│   │   ├── DefectBreakdownTable.tsx # AGMARK tolerance matrix
│   │   └── OnionDetailModal.tsx  # Single onion crop inspection
│   └── reports/
│       ├── OfficialCertificate.tsx # Printable official certificate
│       └── ReportGenerator.tsx   # Metadata editor & PDF export
├── pages/
│   ├── Home.tsx                  # Agricultural AI landing page
│   ├── Analyzer.tsx              # Optical scanner & live execution
│   ├── Results.tsx               # Quality dashboard page
│   ├── Report.tsx                # Official digital report page
│   ├── About.tsx                 # Architecture & computer vision specs
│   ├── Help.tsx                  # Photography rules & defect taxonomy
│   └── TestBench.tsx             # Automated QA test bench
├── services/
│   ├── ai/
│   │   ├── imageValidator.ts     # Pre-inference blur & format validator
│   │   ├── onionDetector.ts      # Stage 1 presence gate
│   │   ├── qualityAnalyzer.ts    # Stage 2 defect segmentation
│   │   └── aiService.ts          # Unified orchestrator
│   ├── grading/
│   │   └── gradingEngine.ts      # Configurable AGMARK grading engine
│   └── reports/
│       └── reportGenerator.ts    # jsPDF + html2canvas PDF export
├── context/
│   └── AnalysisContext.tsx       # React state & localStorage persistence
├── data/
│   ├── demoData.ts               # Offline SVG presets & mock cases
│   └── gradingRules.ts           # Standard constants & thresholds
└── types/
    ├── aiTypes.ts                # TypeScript AI interfaces
    ├── gradingTypes.ts           # Commercial grading types
    └── reportTypes.ts            # Official report & metadata types
```

---

## 📜 Compliance & Agronomic Standards

Grading rules implemented in OnionGuard AI follow standard commercial parameters modeled after:
- **AGMARK Commercial Onion Grading Rules** (Directorate of Marketing & Inspection, Ministry of Agriculture & Farmers Welfare, Govt. of India).
- **APMC Mandi Lot Inward Classification** (Grade A Export, Grade B Domestic, URS Secondary Utility).

---

## 👨‍💻 Developer & Attribution

Developed for the **Smart Onion Quality Assessment & Grading System** initiative to empower farmers and procurement centers with transparent, unbiased computer vision technology.
