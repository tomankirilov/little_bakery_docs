# Little Bakery

Welcome to the documentation for **Little Bakery** — a Blender add-on built for baking textures for **real-time rendering** workflows.

---

## Introduction

Little Bakery helps you bake common game-ready texture maps quickly and consistently using the **active UV map** (no UDIM support). It’s designed around a simple batch workflow:

- Configure global bake settings and output
    
- Define one or more **Texture Sets** (similar to a material set)
    
- Add **Targets** (the meshes you bake _onto_)
    
- Optionally add **Sources** (the meshes you bake _from_)
    
- Hit **BAKE**
    

---

## Features

- Built for **real-time** texture baking
    
- Bake multiple map types per set (Normal, AO, Curvature, Thickness, Position, etc.)
    
- **Texture Sets** for batching outputs (one image per bake target, per set)
    
- Per-set overrides for resolution, anti-aliasing, and padding
    
- Projection controls (cage extrusion and max ray distance), with per-target overrides and optional custom cages
    
- Fast post-process **padding** with multiple algorithms
    
- Output naming based on `texture_set_name + separator + bake_target_name`
    
- Export formats: **PNG** (8/16-bit) and **TGA**
    

---

## Installation

Little Bakery is distributed on GitHub:

- GitHub repository: [https://github.com/tomankirilov/little_bakery](https://github.com/tomankirilov/little_bakery)
    

General steps:

1. Download the add-on from GitHub (ZIP).
    
2. In Blender, go to `Edit > Preferences > Add-ons`.
    
3. Click **Install…** and select the downloaded ZIP.
    
4. Enable **Little Bakery** in the add-ons list.

---

## About

The About section includes:
    
- v1.0.0 (Pale Buns)
    
- [Contacts](https://tomanov.art/category/portfolio/)
    
- [GitHub](https://github.com/tomankirilov/little_bakery/tree/1.0-pale-buns)