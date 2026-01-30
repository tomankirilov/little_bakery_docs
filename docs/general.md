# Little Bakery

Little Bakery is a Blender add-on focused on baking textures for **real-time rendering** workflows. It bakes to the **active UV map** and does **not** support UDIMs.

The core workflow is:

1. Configure **Global Settings** (rendering, bake targets, projection, output)
    
2. Create one or more **Texture Sets** (similar to a “material set” in Substance Painter)
    
3. Add **Targets** (meshes that receive the bake)
    
4. Optionally add **Sources** per Target (meshes that are projected onto the target)
    
5. Press **BAKE**
    

---

## Key Concepts

### Bake Targets

Bake Targets define **which texture types** are produced (Normal, AO, Curvature, etc.).  
The bake target name is used as the **export suffix**.

### Texture Sets

A Texture Set represents a group of baked outputs (think “material set”).  
Each Texture Set produces **one output image per Bake Target**.

Texture Sets can override:

- selected render settings (resolution, AA, padding)
    
- bake targets (add or replace vs global targets)
    

### Targets

Targets are the meshes that are baked **onto** (the receiving/low-poly side).  
All Targets within the same Texture Set are baked into the same output image for each Bake Target.

### Sources

Each Target can have:

- **no Sources** → the Target is treated as both **source and target**
    
- **one or more Sources** → Sources are projected onto the Target
    

---

## Global Settings

### Rendering

#### Rendering Device

Textures can be baked on the **CPU** or **GPU**. To use the GPU, Cycles must be configured with a compatible device:

`Edit > Preferences > System > Cycles Render Devices`

#### Resolution

Final output texture resolution used for baking.

#### Anti-Aliasing

Improves edge quality (mainly along geometry and UV island borders) by baking at a higher resolution and scaling down.

- **MSAA x2**
    
- **MSAA x4**
    
- **MSAA x8**
    

Note: Higher MSAA increases bake time and memory usage. Padding is also computed at the higher resolution, which can slow padding down.

#### Padding

Padding (edge padding / bleed) extends pixels beyond UV island borders to prevent seams when mipmaps or texture filtering sample outside UV shells.

Padding is applied as a post-process and supports three algorithms:

- **Fast:** Expands pixels in 4 directions. Very fast, especially for small padding values.
    
- **Chamfer:** Expands pixels in 8 directions. Slightly slower, generally more accurate than Fast.
    
- **EDT:** Euclidean Distance Transform. Produces the most consistent padding, typically at higher cost.
    

Padding Distance controls how far pixels are extended. The algorithms are designed to be fast, so large padding values are generally practical.

---

### Bake Targets

Bake Targets are managed as a list:

- add / remove items
    
- reorder items (up/down)
    
- rename targets (double-click)
    

The bake target **name** determines the exported texture suffix.  
A separator can be configured in add-on preferences (default: `_`).

#### Normal

Bakes a normal map.

- **Space-Tangent:** Tangent-space normals (commonly used for real-time/game assets).
    
- **Space-Object:** Object-space normals (commonly used for static/offline workflows).
    
- **Swizzle:** Remaps normal directions **X, Y, Z** to output channels **R, G, B**.
    

Note: Normal baking can be influenced by source shading if the source material uses **bump/normal** inputs.

#### Ambient Occlusion

Bakes an **Ambient Occlusion (AO)** map: how much ambient light is blocked by nearby geometry. Creases and tight gaps bake darker; open areas bake lighter.

- **Ray Count:** Number of AO rays cast per texel. Higher reduces noise but increases bake time.
    
- **Render Samples:** Cycles sample count used for the bake. Higher reduces noise but increases bake time.
    
- **Distance:** Maximum ray length used to find occluders. Lower emphasizes small creases; higher captures broader occlusion.
    
- **Contrast:** Adjusts the AO intensity distribution.
    

**AO Mode** controls which objects are considered for occlusion:

- **Global:** AO considers **all objects in the scene**.
    
- **Set:** AO considers **only objects that belong to the Texture Set**:
    
    - all Sources
        
    - Targets that have no Sources (since they act as their own Source)
        
- **Local:** AO considers **only the Sources attached to the current Target**.  
    Sources from other Targets are excluded.
    
- **Isolated:** AO is computed **per Source**, where each Source is **self-occluding only**.  
    (No other objects contribute to the occlusion for that Source.)
    

AO is geometry-based (materials do not affect it).

#### Curvature

Bakes a **Curvature** map: where the surface is **convex** (edges/ridges) or **concave** (valleys). Useful for edge wear, dirt buildup, stylized shading, and masking.

- **Exponent (Default 2.2):** Applies a gamma-style correction so the map is easier to blend in typical color workflows.  
    With **2.2**, _neutral curvature_ (flat / no curvature) maps to **50% gray**.  
    With **1.0**, the neutral point is not at 50% gray.
    
- **Contrast:** Applied to the **raw curvature values before baking** (pre-encoding).  
    This increases or decreases separation between edges and flats **without compressing** the raw data,  
    while preserving a stable neutral midpoint (50% gray after the exponent step).
    

Curvature is geometry-based (materials do not affect it).

#### Thickness

Bakes a **Thickness** map: approximate distance from the surface to the opposite side of the mesh (how “thick” the object is). Commonly used for translucency or subsurface-scattering masks.

- **Ray Count:** Number of rays cast per texel. Higher reduces noise but increases bake time.
    
- **Render Samples:** Cycles sample count used for the bake. Higher reduces noise but increases bake time.
    
- **Distance:** Maximum thickness to search for. Lower emphasizes thin features; higher captures thicker volumes.
    

#### Position

Generates a texture storing the **X, Y, Z position** of each object.

#### Bakery Position

Similar to **Position**, but normalized to the **0–1** range.

#### Random Island

Bakes a random value per **Source Mesh**. Useful for masking and variation.

#### Color Attribute

Bakes a color attribute per **Source Mesh**. Default attribute name is `"Color"`.

- **Color Attribute:** Default attribute name to bake. If the attribute does not exist on the Source Mesh, the result will be pure black.  
    This can be overridden per Source (see Sources).
    

#### Custom

Custom bake target using a user-specified material on the Source Mesh.

- **Material:** Material to be used for the bake.
    
- **Bake Type:** Cycles bake type.
    

---

### Projection

Projection controls how Sources project onto Targets.

- **Cage Extrusion:** Inflates the Target (or Cage) to help capture all details during projection.
    
- **Max Ray Distance:** Limits how far projection rays can travel.
    

Important: **Apply object scale** in Blender for consistent projection behavior.

---

### Output

Defines where and how baked images are saved.

- **Directory:** Output directory (relative path). Default: `/bakes`.
    
- **Format:** Output image format.
    
- **PNG**
    
    -- **Color:** RGB / RGBA
    
    - **Color Depth:** 8 / 16
        
    - **Compression:** slider (default: 15%)
        
- **TGA**
    
    - **Color:** RGB / RGBA
        

**File naming:**

`<texture_set_name><separator><bake_target_name>`

Example (separator `_`): `weapon_normal`, `weapon_ao`, `weapon_curvature`

---

## Texture Sets

Texture Sets are managed as a list:

- add / remove items
    
- reorder items (up/down)
    
- enable/disable baking per set (checkbox)
    
- rename sets (name controls export filename prefix)
    

Each Texture Set produces one output image per bake target, containing all Targets in that set.

### Override Render Settings

Per-set overrides use checkboxes. Only enabled overrides apply.

- **Resolution** (per set)
    
- **Padding** (per set)
    
- **Anti-Aliasing** (per set)
    

### Override Bake Targets

Texture Sets can override which bake targets are used.

- **Mode:**
    
    - **Add:** Adds additional bake targets for this set on top of the global bake targets.
        
    - **Replace:** Uses only the bake targets listed here (replacing the global bake targets for this set).
        
- Includes add/remove/reorder controls like the global Bake Targets list.
    

---

## Targets

Targets are meshes that receive the bake (projection destination). Targets are managed as a list:

- add / remove items
    
- reorder items (up/down)
    
- object picker (mesh objects only)
    

Convenience: If mesh objects are selected when pressing `+`, an entry is created for each selected mesh.

All Targets within the same Texture Set are baked into the same output image for each bake target.

### Override Projection Settings (per Target)

Per-target overrides use checkboxes. Only enabled overrides apply.

- **Cage:** Optional custom cage object.
    
    - If an object is selected when setting the override, it can be auto-filled into this slot.
        
- **Cage Extrusion:** Overrides the extrusion value for this Target.
    
    - Note: Even if a Cage object is provided, extrusion still applies to the cage.
        
    - To use the cage exactly as-is, override Cage Extrusion to **0.0**.
        
- **Max Ray Distance:** Overrides the maximum ray distance for this Target.
    

If no Cage object is provided, the Target itself is used and extruded.

---

## Sources

Sources are only shown when a Target is selected. Each Target can have:

- **no Sources:** the Target is treated as both Source and Target
    
- **one or more Sources:** Sources are projected onto the Target
    

Sources are managed as a list with add/remove/reorder controls.

Convenience: selecting multiple objects and pressing `+` adds them all.

### Per-Source Settings

- **Color Attribute:** Overrides the Color Attribute name for this Source (used by the Color Attribute bake target).
    

---

## Baking

Press **BAKE** to start baking.

During the bake, Little Bakery displays:

- which texture is currently baking
    
- progress bar
    
- a summary at the end (basic bake information)
    

---

## About

The About section includes:

- documentation link
    
- version information
    
- contact info
    
- GitHub link