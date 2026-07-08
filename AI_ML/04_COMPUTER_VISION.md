# Computer Vision — Interview Prep

> **Tag: Theory** — CNNs are the core; convolution/pooling mechanics, the task taxonomy, and transfer learning are what get asked.

## Images as Data

Image = H×W×3 tensor (RGB, 0–255). A 224×224 image = 150k values — a dense layer on raw pixels = millions of parameters and no spatial awareness. CNNs fix both.

## Convolution (the mechanism to explain)

A small **filter/kernel** (e.g., 3×3 weights) slides across the image; at each position: elementwise multiply + sum → one output value → an output **feature map** showing *where that pattern occurs*.

```
Input (5×5)      Kernel (3×3)      Feature map (3×3)
[..pixels..]  ⊛  [edge detector] = [activations where edges exist]
```

Why convolutions beat dense layers for images:
- **Local connectivity:** pixels relate to neighbors — a 3×3 filter matches that structure.
- **Weight sharing:** one filter scans everywhere → detects the pattern anywhere (**translation invariance**) with tiny parameter count (3×3×C vs millions).

Vocabulary: stride (step size), padding (preserve size), channels (each layer learns many filters). Output size = (N − K + 2P)/S + 1 — the one formula worth memorizing (e.g., 32×32, 5×5 kernel, no pad, stride 1 → 28×28).

**Pooling:** downsample feature maps (max-pool 2×2 keeps the strongest activation per window) → smaller, more robust to small shifts, larger effective receptive field.

**The hierarchy story (tell this):** early layers learn edges → mid layers combine into textures/parts (eyes, wheels) → deep layers represent objects — learned feature engineering, which is exactly what classical CV did by hand (SIFT/HOG) before 2012.

## Classic Architecture Pattern & Milestones

`[Conv → ReLU → (Conv → ReLU) → Pool] × N → flatten/GAP → dense → softmax`

- **AlexNet (2012):** deep learning's ImageNet moment.
- **VGG:** stacked small 3×3 convs.
- **ResNet (2015):** **skip connections** — layers learn residuals; solved vanishing gradients; enabled 100+ layers. If you know one architecture deeply, make it ResNet ("why do skip connections help?" = gradient highway + easy identity default).
- **Vision Transformers (ViT):** image → 16×16 patches → tokens → standard Transformer; matches/exceeds CNNs with enough data — the convergence of CV and NLP (nice modern close).

## Task Taxonomy (know the ladder)

| Task | Output | Example models |
|---|---|---|
| Classification | One label per image | ResNet |
| Classification + localization | Label + one box | — |
| **Object detection** | Boxes + labels for ALL objects | YOLO (single-shot, fast), Faster R-CNN (two-stage, accurate) |
| **Semantic segmentation** | Class per pixel | U-Net, DeepLab |
| Instance segmentation | Per-pixel + separates individuals | Mask R-CNN |
| Face recognition | Identity via embedding similarity | Siamese/triplet nets |

Detection extras worth one line: **IoU** (box-overlap metric), **NMS** (drop duplicate boxes), **mAP** (the benchmark metric).

## Transfer Learning (the applied answer)

Nobody trains from scratch on small data: take an ImageNet-pretrained backbone → replace the final layer → either **freeze + train head** (tiny data) or **fine-tune deeper layers** (more data). Early features (edges/textures) are universal. Plus **data augmentation** (flips, crops, color jitter — free data, built-in invariances) and regularization (dropout, batch norm). This paragraph answers "how would you build a classifier with 2,000 images?" — a near-guaranteed applied question.

## Most-Asked Interview Questions

1. **How does a CNN work / why not dense layers?** → local connectivity + weight sharing + hierarchy story.
2. **Explain convolution, stride, padding, pooling.** → mechanics + the output-size formula.
3. **Why ReLU?** Cheap, non-saturating gradient → deep nets train; know dying-ReLU caveat (leaky variant).
4. **What do skip connections solve?** Vanishing gradients / degradation; identity shortcut makes deeper-than-shallower always possible.
5. **Classification vs detection vs segmentation?** → the ladder table.
6. **YOLO vs two-stage detectors?** Single pass over a grid (real-time) vs propose-then-classify (accuracy) — speed/accuracy tradeoff.
7. **Small dataset — approach?** Transfer learning + augmentation + freeze/fine-tune decision (→ the paragraph above).
8. **Overfitting signs and fixes in CV?** Train/val gap; augment, dropout, early stopping, more data, smaller model.
9. **CNN vs ViT?** Built-in spatial priors (data-efficient) vs learned-from-scratch attention (scales better with data).
10. **How does face recognition work?** Embedding networks trained with triplet/contrastive loss → identity = nearest-neighbor in embedding space (same cosine-similarity math as NLP embeddings — cross-ref).
