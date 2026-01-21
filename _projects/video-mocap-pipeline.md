---
title: Video-to-Motion Capture Pipeline
description: An end-to-end pipeline converting regular video into game-ready motion capture using SAM3D, SMPL-X, and Blender, with direct export to Unreal Engine.
date: 2025-01-05 12:00:00
label: ML / Computer Vision / 3D
image: '/TimBunkerPortfolio/images/mocap-pipeline-cover.png'
---

# Video-to-Motion Capture Pipeline

## Overview

Transform regular video footage into professional motion capture data compatible with Unreal Engine—no expensive mocap suits or studio setup required.

This pipeline leverages Meta's SAM-3D (Segment Anything Model for 3D bodies) to extract human motion from video, converts it to the industry-standard SMPL-X body model, and exports animation-ready FBX files that work directly with Unreal Engine's Manny skeleton.

---

## Pipeline Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Video     │────▶│   SAM-3D    │────▶│  SMPL-X     │────▶│   Blender   │
│   Input     │     │  Tracking   │     │  Params     │     │   Export    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                          │                   │                    │
                          ▼                   ▼                    ▼
                    track_000.npz      *_smplx.npz           *.fbx
                    (MHR mesh seq)     (body params)      (Unreal-ready)
```

---

## Components

### 1. SAM-3D Body Tracking

Uses Meta's SAM-3D model to extract per-frame body pose from video:
- Outputs MHR (Meta Human Rig) mesh sequences
- Includes global rotations, joint coordinates, and timestamps
- Handles occlusion and complex poses

### 2. MHR → SMPL-X Conversion

Converts Meta's proprietary MHR format to the open SMPL-X body model:

```bash
python sam3d_track_to_smplx.py \
  --track_npz track_000.npz \
  --out_npz track_000_smplx.npz \
  --gender male \
  --batch_size 12
```

**Key features**:
- Uses Meta's official conversion tools
- Supports male/female/neutral body models
- Batch processing for efficiency
- Outputs standard SMPL-X parameters (pose, shape, translation)

### 3. Vertex Cache Generation

For visualization without runtime dependencies:

```bash
python smplx_params_to_vertex_cache.py \
  --smplx_params track_000_smplx.npz \
  --out_cache track_000_vertex_cache.npz
```

Precomputes all mesh vertices for lightweight Blender visualization.

### 4. FBX Export for Unreal Engine

Two export paths optimized for different workflows:

#### Option A: MHR Rig Export
Exports using Meta's original skeleton:
```bash
blender -b -P mhr_track_to_fbx.py -- \
  --track_npz track_000.npz \
  --mhr_rig_fbx lod1.fbx \
  --output_fbx track_000_mhr.fbx
```

#### Option B: Manny-Native Export (Recommended)
Exports directly to Unreal's Manny skeleton—no retargeting needed:
```bash
blender -b -P mhr_track_to_manny_fbx.py -- \
  --track_npz track_000.npz \
  --mhr_rig_fbx lod1.fbx \
  --manny_fbx Manny.fbx \
  --output_fbx track_000_manny.fbx
```

**Advanced options**:
- `--skeleton_only`: Export armature without mesh (smaller files)
- `--apply_global_rot`: Include world-space orientation
- `--max_hand_deg`: Clamp hand rotations to prevent snapping
- `--no_twist`: Disable twist bones for cleaner animation

---

## Technical Challenges Solved

### Bone Mapping Complexity
MHR uses a different skeleton hierarchy than Unreal's Manny. The pipeline implements:
- Automatic bone name mapping
- Rotation space conversion (local vs global)
- Twist bone handling
- Spine chain interpolation

### Preserving Motion Quality
Multiple mapping modes handle different body parts optimally:
- `global_delta`: For root motion and pelvis
- `relative_delta`: For spine and extremities
- Clavicle weight blending for natural shoulder movement

### Unreal Compatibility
FBX export preserves:
- Manny's original hierarchy and units
- Proper axis conventions (-Y forward, Z up)
- Animation curves at 30 FPS (configurable)

---

## Use Case: Fencing Motion Capture

I developed this pipeline specifically to capture fencing movements for game development:

1. **Record** training footage with a standard camera
2. **Process** through SAM-3D for body tracking
3. **Convert** to SMPL-X parameters
4. **Export** to Unreal-ready FBX
5. **Import** directly onto Manny character in UE5

No mocap suit. No expensive hardware. Just video and compute.

---

## Dependencies

### Python Environment
```bash
mamba create -n mhrconv -c conda-forge \
  python=3.12 pymomentum pytorch numpy scipy trimesh tqdm
pip install smplx
```

### Required Assets
- Meta MHR model files (lod*.fbx, compact_v6_1.model)
- SMPL-X body models (SMPLX_MALE.npz, etc.)
- Unreal Engine Manny skeleton export

### Tools
- Blender (for FBX export)
- PyTorch (for body model inference)

---

## What I Learned

- **3D Graphics Pipelines**: Coordinate systems, skeleton hierarchies, quaternion math
- **ML Model Integration**: Working with pretrained models and their data formats
- **Blender Scripting**: Headless automation with Python API
- **Game Engine Workflows**: Understanding Unreal's animation import requirements
- **Complex Data Transformations**: Multiple conversion steps with error propagation
