<h1 align="center">DUT-Plus: A Multi-Target and Distractor-Enriched Benchmark for Drone Detection</h1>

<div align="center">

![License](https://img.shields.io/badge/license-Research--Only-yellow)
![Images](https://img.shields.io/badge/images-14%2C000-blue)
![Format](https://img.shields.io/badge/format-YOLO%20%2F%20VOC-green)
![BaiduYun](https://img.shields.io/badge/download-BaiduYun-red)

</div>

<p align="center">
  <img src="Assets/dut_plus_overview.jpg" width="900">
</p>

<p align="center"><i>Sample images from DUT-Plus showing multi-drone formations, visually similar avian distractors, and aircraft-like hard negatives over cluttered backgrounds.</i></p>

---

## 1. Overview

**DUT-Plus** is a vision-based drone detection benchmark constructed on top of the publicly available [DUT Anti-UAV](https://github.com/wangdongdut/DUT-Anti-UAV) dataset. The original DUT Anti-UAV is one of the most widely used resources for single-drone detection, but the majority of its images contain a single isolated drone against a relatively clean background. Real-world surveillance scenarios are far more demanding. Multiple drones may appear in formation, fast-moving birds and small aircraft routinely intrude into the field of view, and detectors must distinguish actual drones from these visually similar objects under cluttered backgrounds.

To bridge this gap, DUT-Plus was built with three explicit design goals.

1. Provide dense **multi-drone formations** rather than single-target scenes.
2. Inject **hard-negative distractors** such as birds and aircraft to stress test false-positive suppression.
3. Preserve the **photorealism** of the original DUT Anti-UAV style so that detectors trained on DUT-Plus generalize to real-world surveillance footage.

DUT-Plus contains **14,000 images in total**, partitioned into 7,000 training, 4,000 validation, and 3,000 test images, all annotated with bounding boxes for drone targets only.

---

## 2. Why DUT-Plus

The table below compares DUT-Plus with its parent dataset and several common drone detection benchmarks.

| Benchmark | Multi-Drone Scenes | Bird Distractors | Aircraft Distractors | Cluttered Backgrounds | Manual Annotation | Total Images |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| DUT Anti-UAV | △ rare | ✗ | ✗ | △ partial | ✓ | ~10,000 |
| Det-Fly | ✗ | ✗ | ✗ | ✓ | ✓ | ~13,000 |
| MIDGARD | ✗ | ✗ | ✗ | △ partial | ✓ | ~6,000 |
| **DUT-Plus (ours)** | **✓** | **✓** | **✓** | **✓** | **✓** | **14,000** |

DUT-Plus is therefore positioned as a stress test specifically aimed at the two weaknesses most often observed in long-range drone detectors trained on DUT Anti-UAV alone, namely missed detections in multi-drone formations and false positives triggered by birds.

---

## 3. Construction Pipeline

The construction of DUT-Plus follows a six-stage pipeline that combines diffusion-based image generation, manual quality control, and manual bounding-box annotation. Every stage is described in detail below so that the dataset can be reproduced or extended by other researchers.

### Stage 1 — Source Image Curation from DUT Anti-UAV

The pipeline starts from a curated subset of the original DUT Anti-UAV detection dataset. Images are selected according to three criteria. First, the source image must contain a clean and unambiguous drone target whose pose, scale, and lighting are well exposed. Second, the background should cover the typical scene categories of the original dataset, including sky, urban skyline, vegetation, and mixed terrain. Third, blurry, mislabeled, or duplicated frames are excluded. The curated subset serves as the appearance prior for downstream generation.

### Stage 2 — Multi-Target Scene Generation with FLUX.1

To populate scenes with multiple drones and visually similar distractors, we use **FLUX.1**, a state-of-the-art text-to-image diffusion model with strong photorealism on small aerial objects. For each target scene type, we design a structured prompt template that specifies the following elements.

- **Subject layout**: number of drones (2 to 6), spatial arrangement (formation, scattered, paired), and inter-drone distance.
- **Distractor specification**: number of birds or aircraft, their species or model class, and their relative position with respect to the drones.
- **Background context**: sky condition, urban or rural scene, lighting time of day, and the level of background clutter.
- **Style anchor**: a fixed style descriptor chosen to match the realism and color tone of DUT Anti-UAV imagery, ensuring distribution alignment between generated and real samples.

Each prompt is sampled with multiple random seeds to enrich diversity. Approximately 25,000 candidate images are generated in this stage, of which roughly 56% survive the next two stages.

### Stage 3 — Hard-Negative Distractor Injection

Hard negatives are central to DUT-Plus. Two distractor categories are produced.

- **Avian distractors**: small to medium-sized birds in flight, including silhouettes and side views that overlap with typical drone aspect ratios. Both single birds and bird flocks are generated.
- **Aircraft distractors**: distant fixed-wing aircraft and small civil aviation models. These appear at sizes that overlap with mid-range drone projections, forcing the detector to rely on shape rather than mere blob size.

Distractors are placed in three configurations: appearing alone in the frame as pure negatives, co-occurring with one or more drones in the same frame, and partially occluding or being occluded by the drone targets. The co-occurrence design is what drives the hard-negative training signal that DUT Anti-UAV alone does not provide.

### Stage 4 — Photorealism Quality Screening

Generated images are not used directly. They first pass through a manual two-pass screening procedure designed to remove artifacts that diffusion models commonly introduce.

- **Pass A — Realism check**: annotators reject images with anatomically implausible birds, deformed propellers, melted aircraft fuselages, or unnatural lighting transitions.
- **Pass B — Distribution check**: annotators reject images whose overall color tone, sensor noise pattern, or compositional style deviates noticeably from the DUT Anti-UAV style. This step prevents the model from learning a shortcut between synthetic style and target identity.

Only images that pass both checks proceed to annotation. The reject rate during screening is approximately 44%.

### Stage 5 — Manual Bounding-Box Annotation

Every retained image is manually annotated using **LabelImg**, with the following labeling protocol.

- Only **drone** is treated as a positive class. Birds and aircraft are intentionally **left unlabeled**, so they act as hard negatives during training rather than as additional positive classes. This design choice forces the detector to learn discriminative features that separate drones from visually similar non-drone targets.
- A bounding box must enclose the entire airframe, including rotors and visible payload, with at most a 2-pixel margin on each side.
- Targets smaller than 4×4 pixels are still annotated as long as they are visible to a human annotator under normal screen viewing.
- Each image is reviewed by at least two annotators. Disagreements are resolved by a third senior annotator.
- The final annotation export is provided in both **YOLO** format and **Pascal VOC XML** format.

### Stage 6 — Mutually Exclusive Train / Val / Test Split

The 14,000 retained images are partitioned into three mutually exclusive splits.

| Split | Images | Purpose |
|---|:---:|---|
| Train | 7,000 | Model training |
| Validation | 4,000 | Hyperparameter tuning, model selection |
| Test | 3,000 | Final evaluation, never seen during training |

Splits are stratified so that each contains a balanced proportion of single-drone, multi-drone, and distractor-co-occurrence scenes. No image or near-duplicate of an image appears in more than one split. Random seeds and the split index files are released with the dataset to ensure reproducibility.

---

## 4. Dataset Statistics

### 4.1 Overall Distribution

| Property | Train | Val | Test | Total |
|---|:---:|:---:|:---:|:---:|
| Images | 7,000 | 4,000 | 3,000 | 14,000 |
| Drone instances | ~17,500 | ~10,000 | ~7,500 | ~35,000 |
| Avg. drones per image | 2.5 | 2.5 | 2.5 | 2.5 |
| Images with $\geq$ 2 drones | ~62% | ~62% | ~62% | ~62% |
| Images containing avian distractors | ~38% | ~38% | ~38% | ~38% |
| Images containing aircraft distractors | ~12% | ~12% | ~12% | ~12% |

### 4.2 Target Scale Distribution

Target scale is defined as $\sqrt{w \cdot h} / \sqrt{W \cdot H}$, where $w \times h$ is the bounding-box size and $W \times H$ is the image size.

| Scale Range | Proportion |
|---|:---:|
| Very small (< 0.02) | ~22% |
| Small (0.02–0.05) | ~41% |
| Medium (0.05–0.15) | ~28% |
| Large ($\geq$ 0.15) | ~9% |

The distribution is intentionally skewed toward small-scale targets, which is the regime where detectors trained on DUT Anti-UAV alone tend to fail.

### 4.3 Background Categories

Backgrounds are sampled from sky-only, urban skyline, vegetation, mountainous, mixed-terrain, and night-or-twilight scenes, in roughly balanced proportions.

---

## 5. File Structure and Format

After unzipping the downloaded archive, the directory layout is as follows.