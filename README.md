<h1 align="center">DUT-Plus: A Multi-Target and Distractor-Enriched Benchmark for Drone Detection</h1>

<div align="center">

![Images](https://img.shields.io/badge/images-14%2C000-blue)
![Format](https://img.shields.io/badge/format-YOLO%20%2F%20VOC-green)
![BaiduYun](https://img.shields.io/badge/download-BaiduYun-red)

</div>

<p align="center">
  <img src="Assets/dut_plus_overview.jpg" width="900">
</p>

<p align="center"><i>Sample images from DUT-Plus illustrating multi-drone scenes and visually similar distractors over cluttered backgrounds.</i></p>

---

## 1. Overview

DUT-Plus is a vision-based drone detection benchmark constructed on top of the publicly available [DUT Anti-UAV](https://github.com/wangdongdut/DUT-Anti-UAV) dataset. The original DUT Anti-UAV is widely used for single-drone detection. However, the majority of its images contain a single drone target against a relatively clean background. Real-world surveillance scenarios are considerably more challenging. Multiple drones may appear simultaneously, and visually similar objects such as birds frequently appear in the same field of view. Detectors are therefore required to discriminate drone targets from these distractors under cluttered backgrounds.

DUT-Plus is constructed to address this limitation. The dataset contains 14,000 images partitioned into 7,000 training, 4,000 validation, and 3,000 test samples. All images are annotated with bounding boxes for drone targets only. The design has three goals.

1. Provide multi-drone scenes rather than single-target scenes.
2. Inject visually similar distractors such as birds as hard-negative samples for false-positive suppression.
3. Preserve the visual style of the original DUT Anti-UAV dataset for distribution consistency.

---

## 2. Comparison with Existing Benchmarks

DUT-Plus is positioned to address two weaknesses commonly observed in detectors trained on DUT Anti-UAV alone, namely missed detections in multi-drone scenes and false positives triggered by visually similar non-drone targets.

| Benchmark | Original Task | Multi-Drone Scenes | Bird Distractors | Annotation |
|---|---|:---:|:---:|---|
| DUT Anti-UAV | Anti-UAV detection and tracking | △ rare | ✗ | Manual |
| Det-Fly | Air-to-air micro-UAV detection | ✗ | ✗ | Manual |
| Anti-UAV (Jiang et al.) | Anti-UAV tracking (RGB and IR) | ✗ | ✗ | Manual |
| Drone-vs-Bird | Drone detection under bird interference | ✗ | ✓ | Manual |
| Real-World (Pawełczyk et al.) | Multi-model quadcopter detection | ✗ | ✗ | Manual |
| MIDGARD (Walter et al.) | Multi-MAV relative localization | ✗ | ✗ | Auto (UVDAR) |
| **DUT-Plus (ours)** | **Anti-UAV detection** | **✓** | **✓** | **Manual** |

---

## 3. Construction Pipeline

DUT-Plus is built through four stages, namely source image curation, multi-target scene generation, manual quality screening, and manual bounding-box annotation. Each stage is described below.

### 3.1. Source Image Curation

The pipeline starts from the original DUT Anti-UAV detection dataset. Source images are selected to cover the typical scene categories of the original dataset, including sky, urban, vegetation, and mixed terrain. Blurry, mislabeled, and duplicated frames are excluded. The selected images serve as appearance and style references for downstream generation.

### 3.2. Multi-Target Scene Generation

Multi-drone scenes and distractor-rich scenes are generated using FLUX.1, a recent text-to-image diffusion model. For each scene type, a prompt template specifies the number of drones, the spatial arrangement of drones, the type and number of distractors, the background scene, and the lighting condition. The style descriptor in each prompt is aligned with the visual style of the original DUT Anti-UAV imagery to maintain distribution consistency between generated and source samples. Each prompt is sampled with multiple random seeds for diversity.

### 3.3. Manual Quality Screening

Generated images are screened by human annotators before annotation. Two criteria are applied. The first criterion assesses photorealism, where images with visible diffusion artifacts on drones, distractors, or backgrounds are rejected. The second criterion assesses style consistency, where images whose color tone or noise statistics deviate noticeably from the DUT Anti-UAV style are rejected. Only images that pass both criteria proceed to the next stage.

### 3.4. Manual Bounding-Box Annotation

Retained images are annotated with LabelImg under a drone-only protocol. Birds and other distractors are intentionally left unlabeled, so that they provide hard-negative supervision during training rather than serving as additional positive classes. This protocol requires the detector to discriminate drones from visually similar non-drone targets at the feature level. The final annotations are exported in both YOLO and Pascal VOC XML formats.

### 3.5. Mutually Exclusive Train, Validation, and Test Splits

The 14,000 annotated images are partitioned into three mutually exclusive splits. No image or its near-duplicate appears in more than one split. The split index files are released together with the dataset for reproducibility.

| Split | Images | Purpose |
|---|:---:|---|
| Train | 7,000 | Model training |
| Validation | 4,000 | Hyperparameter tuning and model selection |
| Test | 3,000 | Final evaluation |

---

## 4. File Structure and Annotation Format

After unzipping the downloaded archive, the directory layout is as follows.