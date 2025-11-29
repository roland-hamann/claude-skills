---
name: blender-3d-printing
description: "Comprehensive 3D printing workflow using Blender via MCP. Use when working with 3D models for printing: (1) Importing/generating models from various sources, (2) Preparing models for 3D printing (manifold checks, scaling, orientation), (3) Modifying and repairing geometry, (4) Exporting print-ready files, or (5) Any 3D printing preparation tasks"
---

# Blender 3D Printing Workflow via MCP

## Overview

This skill guides you through using Blender via MCP (Model Context Protocol) for 3D printing workflows. Blender is a powerful 3D modeling tool, and through MCP integration, you can programmatically control it to prepare models for 3D printing.

**Key capabilities:**
- Import models from multiple sources (Polyhaven, Sketchfab, AI generation)
- Generate 3D models using AI (Hyper3D Rodin, Hunyuan3D)
- Modify and repair geometry using Python scripts
- Check and fix common 3D printing issues (non-manifold geometry, scale, normals)
- Export print-ready files (STL, OBJ, etc.)

## Initial Connection Check

**CRITICAL - First Step**: Before performing ANY Blender operations, verify the MCP server connection:

```
mcp__blender__get_scene_info
```

This command confirms that:
- Blender MCP server is running and accessible
- Connection is properly established
- You can proceed with Blender operations

**If the connection fails:**
- Verify Blender is running
- Check that the MCP server is properly configured
- Restart Blender if necessary
- Check MCP server logs for connection errors

**Only proceed with other operations after successful connection verification.**

## User's Blender Configuration

**IMPORTANT**: The user has configured Blender with a default profile that includes:
- **Units**: Metric (millimeters) - already set as default
- **3D Print Toolbox addon**: Already enabled and ready to use

**When to use the default profile:**
- Starting new projects from scratch
- Opening files that don't have proper 3D printing setup
- After generating AI models (they need proper unit configuration)

## Required Blender Settings for 3D Printing

**At the start of every project, check and configure these settings if not already set:**

```python
import bpy

# Check and set scene units if needed
if bpy.context.scene.unit_settings.system != 'METRIC':
    bpy.context.scene.unit_settings.system = 'METRIC'
    print("Set unit system to METRIC")

if bpy.context.scene.unit_settings.length_unit != 'MILLIMETERS':
    bpy.context.scene.unit_settings.length_unit = 'MILLIMETERS'
    print("Set length unit to MILLIMETERS")

if bpy.context.scene.unit_settings.scale_length != 0.001:
    bpy.context.scene.unit_settings.scale_length = 0.001
    print("Set unit scale to 0.001")

# Check and set viewport overlay scale if needed
for area in bpy.context.screen.areas:
    if area.type == 'VIEW_3D':
        for space in area.spaces:
            if space.type == 'VIEW_3D':
                if space.overlay.grid_scale != 0.1:
                    space.overlay.grid_scale = 0.1
                    print("Set viewport overlay scale to 0.1")

# Check and set spacebar to search if needed
if not bpy.context.preferences.inputs.use_spacebar_search:
    bpy.context.preferences.inputs.use_spacebar_search = True
    print("Set spacebar action to Search")

print("Scene configuration verified")
```

**Required settings:**
- **Unit system**: Metric
- **Length unit**: Millimeters
- **Unit scale**: 0.001000
- **Viewport overlay scale**: 0.1
- **Spacebar action**: Search

## Workflow Decision Tree

### Getting Models into Blender

**AI Generation (text-to-3D or image-to-3D)**
- Use Hyper3D Rodin for fast generation
- Use Hunyuan3D for high-quality results
- See "AI Model Generation" section below

**Downloading Existing Models**
- Use Polyhaven for textures, HDRIs, and high-quality models
- Use Sketchfab for community models
- See "Importing Models" section below

**Creating from Scratch**
- Use Python code execution for procedural modeling
- See "Procedural Modeling" section below

### Preparing Models for 3D Printing

**Basic Preparation**
1. Check scale and dimensions
2. Check for non-manifold geometry
3. Verify normals orientation
4. Optimize polygon count if needed

**Common Issues to Fix**
- Non-manifold edges (use 3D Print Toolbox addon)
- Holes in mesh
- Intersecting geometry
- Improper scale

## Available MCP Tools

### Scene and Object Information
```
mcp__blender__get_scene_info - Get overview of current scene
mcp__blender__get_object_info - Get details about specific object
mcp__blender__get_viewport_screenshot - Capture current 3D view
```

### Code Execution
```
mcp__blender__execute_blender_code - Execute Python code in Blender
```
**IMPORTANT**: Break complex operations into smaller steps. Execute code incrementally to avoid errors.

### Polyhaven Integration
```
mcp__blender__get_polyhaven_status - Check if Polyhaven is enabled
mcp__blender__get_polyhaven_categories - Get asset categories
mcp__blender__search_polyhaven_assets - Search for assets
mcp__blender__download_polyhaven_asset - Download and import asset
mcp__blender__set_texture - Apply texture to object
```

### Sketchfab Integration
```
mcp__blender__get_sketchfab_status - Check if Sketchfab is enabled
mcp__blender__search_sketchfab_models - Search for models
mcp__blender__download_sketchfab_model - Download model by UID
```

### Hyper3D Rodin (AI Generation)
```
mcp__blender__get_hyper3d_status - Check status and key type
mcp__blender__generate_hyper3d_model_via_text - Generate from text prompt
mcp__blender__generate_hyper3d_model_via_images - Generate from images
mcp__blender__poll_rodin_job_status - Check generation progress
mcp__blender__import_generated_asset - Import completed model
```

### Hunyuan3D (AI Generation)
```
mcp__blender__get_hunyuan3d_status - Check status and key type
mcp__blender__generate_hunyuan3d_model - Generate from text/image
mcp__blender__poll_hunyuan_job_status - Check generation progress
mcp__blender__import_generated_asset_hunyuan - Import completed model
```

## Common 3D Printing Workflows

### Workflow 1: Import and Prepare Existing Model

1. **Import the model**
   - From Sketchfab/Polyhaven: Use respective download tools
   - From file: Use `execute_blender_code` with `bpy.ops.import_mesh.stl()` or similar

2. **Check the model**
   ```python
   import bpy

   # Select object
   obj = bpy.context.active_object

   # Check dimensions
   print(f"Dimensions: {obj.dimensions}")

   # Check for non-manifold
   bpy.ops.object.mode_set(mode='EDIT')
   bpy.ops.mesh.select_all(action='DESELECT')
   bpy.ops.mesh.select_non_manifold()
   # If vertices selected, there are non-manifold elements
   ```

3. **Scale to correct size**
   ```python
   # Scale to desired size (use scene's configured units)
   # Check current size first
   print(f"Current dimensions: {obj.dimensions}")

   # Set target size in scene units
   target_height = 100.0  # in scene units (check unit settings)
   current_height = obj.dimensions.z
   scale_factor = target_height / current_height
   obj.scale = (scale_factor, scale_factor, scale_factor)
   bpy.ops.object.transform_apply(scale=True)
   ```

4. **Fix common issues**
   ```python
   # Enter edit mode
   bpy.ops.object.mode_set(mode='EDIT')
   bpy.ops.mesh.select_all(action='SELECT')

   # Recalculate normals
   bpy.ops.mesh.normals_make_consistent(inside=False)

   # Remove doubles
   bpy.ops.mesh.remove_doubles(threshold=0.0001)

   # Fill holes
   bpy.ops.mesh.select_all(action='DESELECT')
   bpy.ops.mesh.select_non_manifold()
   bpy.ops.mesh.edge_face_add()

   bpy.ops.object.mode_set(mode='OBJECT')
   ```

5. **Run 3D Print Toolbox check (MANDATORY)**
   ```python
   # CRITICAL: Always run 3D Print Toolbox before export
   import bpy

   # Ensure object is selected
   obj = bpy.context.active_object
   obj.select_set(True)
   bpy.context.view_layer.objects.active = obj

   # Run 3D Print check
   bpy.ops.mesh.print3d_check_all()

   # Check the results in the 3D Print Toolbox panel
   # Fix any errors before proceeding to export
   ```

6. **Export for printing** (only after passing 3D Print check)
   ```python
   # Export as STL
   bpy.ops.export_mesh.stl(
       filepath="/path/to/output.stl",
       use_selection=True,
       global_scale=1.0,
       use_mesh_modifiers=True
   )
   ```

### Workflow 2: AI Generate Model for Printing

**Using Hyper3D Rodin:**

1. **Check status** (remember key type for later use)
   ```
   mcp__blender__get_hyper3d_status
   ```

2. **Generate model**
   - Text prompt: `mcp__blender__generate_hyper3d_model_via_text`
   - Image input: `mcp__blender__generate_hyper3d_model_via_images`
   - Optional: Use `bbox_condition` to control proportions [Length, Width, Height]

3. **Poll for completion**
   ```
   mcp__blender__poll_rodin_job_status
   ```
   Use `subscription_key` (MAIN_SITE) or `request_id` (FAL_AI) based on key type

4. **Import when complete**
   ```
   mcp__blender__import_generated_asset
   ```
   Provide object name and `task_uuid` (MAIN_SITE) or `request_id` (FAL_AI)

5. **Prepare for printing** (see Workflow 1, steps 2-6)

**Using Hunyuan3D:**

1. **Check status** (remember key type)
   ```
   mcp__blender__get_hunyuan3d_status
   ```

2. **Generate model**
   ```
   mcp__blender__generate_hunyuan3d_model
   ```
   Provide `text_prompt` and/or `input_image_url`

3. **Poll for completion**
   ```
   mcp__blender__poll_hunyuan_job_status
   ```
   Returns status "DONE" when complete, includes `ResultFile3Ds` with ZIP path

4. **Import when complete**
   ```
   mcp__blender__import_generated_asset_hunyuan
   ```
   Provide object name and `zip_file_url` from poll result

5. **Prepare for printing** (see Workflow 1, steps 2-6)

### Workflow 3: Procedural Modeling for Printing

Create custom parametric models using Python:

```python
import bpy
import bmesh

# Clear existing mesh
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

# Create base mesh
mesh = bpy.data.meshes.new("CustomModel")
obj = bpy.data.objects.new("CustomModel", mesh)
bpy.context.collection.objects.link(obj)

# Use BMesh for procedural generation
bm = bmesh.new()

# Example: Create custom geometry
# ... your procedural code here ...

# Finalize
bm.to_mesh(mesh)
bm.free()

# Make manifold
bpy.context.view_layer.objects.active = obj
obj.select_set(True)
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.mesh.normals_make_consistent(inside=False)
bpy.ops.object.mode_set(mode='OBJECT')
```

## 3D Printing Best Practices

### Critical Checks Before Exporting

**MOST IMPORTANT: Run 3D Print Toolbox**
```python
bpy.ops.mesh.print3d_check_all()
```
This will automatically check for most issues below. **Fix ALL errors** before proceeding.

1. **Manifold Geometry** (checked by 3D Print Toolbox)
   - Every edge must connect exactly 2 faces
   - No holes, no intersecting faces
   - Manual check: `bpy.ops.mesh.select_non_manifold()`

2. **Correct Scale** (verify manually)
   - Verify dimensions match intended print size
   - Always check scene unit settings first
   - Work in the scene's configured units

3. **Wall Thickness** (checked by 3D Print Toolbox)
   - Minimum 1-2mm for most FDM printers
   - Minimum 0.8mm for resin printers
   - 3D Print Toolbox will flag thin walls

4. **Normals Orientation** (checked by 3D Print Toolbox)
   - All normals should point outward
   - Manual fix: `bpy.ops.mesh.normals_make_consistent(inside=False)`

5. **No Loose Geometry** (checked by 3D Print Toolbox)
   - Remove disconnected vertices/edges
   - Manual fix: `bpy.ops.mesh.delete_loose()`

### Common Issues and Solutions

**Non-Manifold Edges:**
```python
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='DESELECT')
bpy.ops.mesh.select_non_manifold()
# If count > 0: fix by filling holes or removing doubles
bpy.ops.mesh.remove_doubles(threshold=0.0001)
```

**Incorrect Scale:**
```python
# Check current dimensions (in scene units)
print(f"Size: {obj.dimensions.x:.1f} x {obj.dimensions.y:.1f} x {obj.dimensions.z:.1f}")

# Scale to specific size (in scene units)
obj.dimensions = (50, 50, 100)  # Example: 50 x 50 x 100 in scene units
bpy.ops.object.transform_apply(scale=True)
```

**Flipped Normals:**
```python
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.mesh.normals_make_consistent(inside=False)
bpy.ops.object.mode_set(mode='OBJECT')
```

**Too Many Polygons:**
```python
# Add Decimate modifier
mod = obj.modifiers.new(name="Decimate", type='DECIMATE')
mod.ratio = 0.5  # Reduce to 50% of original poly count
bpy.ops.object.modifier_apply(modifier="Decimate")
```

### Using the 3D Print Toolbox

**MANDATORY before every export**: Run the 3D Print Toolbox to check for errors.

```python
import bpy

# Select the object to check
obj = bpy.context.active_object
obj.select_set(True)
bpy.context.view_layer.objects.active = obj

# Run all checks
bpy.ops.mesh.print3d_check_all()

# The results will appear in the 3D Print Toolbox panel
# Common checks include:
# - Solid: Non-manifold edges, bad contiguous edges
# - Intersections: Self-intersecting faces
# - Degenerate: Zero-length edges, zero-area faces
# - Distorted: Heavily distorted faces
# - Thickness: Thin walls that may not print
# - Edge Sharp: Sharp edges that may cause issues
# - Overhang: Overhanging faces that need support
```

**Common 3D Print Toolbox errors and fixes:**

1. **Non-Manifold Edges**
   - Fix: Remove doubles, fill holes, ensure each edge connects exactly 2 faces

2. **Intersecting Geometry**
   - Fix: Use Boolean operations to merge, or manually separate intersecting parts

3. **Zero-Thickness Walls**
   - Fix: Apply Solidify modifier or thicken geometry manually

4. **Degenerate Geometry**
   - Fix: Delete zero-area faces, merge close vertices

**Always fix ALL errors before exporting** - your slicer may not catch these issues.

### Export Settings for Different Printers

**STL Export (most common):**
```python
bpy.ops.export_mesh.stl(
    filepath="model.stl",
    use_selection=True,
    global_scale=1.0,  # Use 1.0 to respect scene units
    use_mesh_modifiers=True,
    ascii=False  # Binary is smaller
)
```

**OBJ Export (with colors/textures):**
```python
bpy.ops.export_scene.obj(
    filepath="model.obj",
    use_selection=True,
    global_scale=1.0,  # Use 1.0 to respect scene units
    use_materials=True,
    use_triangles=True  # Some slicers prefer triangulated
)
```

**3MF Export (advanced features):**
```python
bpy.ops.export_mesh.threemf(
    filepath="model.3mf",
    use_selection=True,
    global_scale=1.0  # Use 1.0 to respect scene units
)
```

## Step-by-Step Execution Strategy

**CRITICAL**: Always execute Blender code in small, manageable steps. Break complex operations into:

1. **Setup** (select objects, enter mode)
2. **Single operation** (one modification at a time)
3. **Validation** (check result with screenshot or object info)
4. **Next operation**

**Example of proper step-by-step approach:**
```
Step 1: Select and enter edit mode
Step 2: Select non-manifold geometry only
Step 3: Fix one issue (e.g., remove doubles)
Step 4: Validate with screenshot
Step 5: Fix next issue (e.g., fill holes)
```

**DON'T** try to do everything in one large script. **DO** break it into digestible chunks.

## AI Model Generation Notes

### Hyper3D Rodin
- **Modes**: MAIN_SITE or FAL_AI (determined by key type)
- **Text prompts**: Must be in English, short descriptions work best
- **Image input**:
  - MAIN_SITE: Provide absolute file paths
  - FAL_AI: Provide URLs to images
- **bbox_condition**: Controls model proportions [L, W, H], e.g., [1.0, 1.0, 2.0] for tall model
- **Generated models**: Normalized size, requires rescaling for printing

### Hunyuan3D
- Supports both text and image input
- Can use both simultaneously for better results
- Accepts Chinese and English prompts
- Local or remote image URLs supported
- Returns OBJ format in ZIP file

### Post-Generation Workflow
1. Generate model (returns job ID)
2. Poll status until "DONE" or "COMPLETED"
3. Import the generated asset
4. **Always rescale** - AI models are normalized
5. **Always check manifoldness** - AI may generate non-printable geometry
6. Apply fixes as needed
7. Export for printing

## Troubleshooting

### "Non-manifold geometry detected"
- Select non-manifold: `bpy.ops.mesh.select_non_manifold()`
- Remove doubles with tight threshold
- Fill holes manually or with `edge_face_add()`
- Check for intersecting faces

### "Model is too small/large"
- Check dimensions: `obj.dimensions`
- Apply scale: Set `obj.dimensions` or use `obj.scale`
- Apply transformation: `bpy.ops.object.transform_apply(scale=True)`

### "Generation failed"
- Check API status first
- Verify prompt is in correct language (English for most)
- For image input, ensure paths/URLs are accessible
- Check polling interval (don't poll too frequently)

### "Export failed"
- Ensure object is selected
- Apply all modifiers before export
- Check file path is valid and writable
- Verify export format supports mesh type

## Quick Reference: Complete Print-Ready Pipeline

```python
# 1. Select object
obj = bpy.context.active_object

# 2. Apply all modifiers
for mod in obj.modifiers:
    bpy.ops.object.modifier_apply(modifier=mod.name)

# 3. Enter edit mode
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')

# 4. Clean geometry
bpy.ops.mesh.remove_doubles(threshold=0.0001)
bpy.ops.mesh.delete_loose()
bpy.ops.mesh.fill_holes()

# 5. Fix normals
bpy.ops.mesh.normals_make_consistent(inside=False)

# 6. Return to object mode
bpy.ops.object.mode_set(mode='OBJECT')

# 7. Scale to printing size (use scene units)
print(f"Current size: {obj.dimensions.z}")
obj.dimensions.z = 100  # Example: 100 in scene units
bpy.ops.object.transform_apply(scale=True)

# 8. Run 3D Print Toolbox check (MANDATORY)
bpy.ops.mesh.print3d_check_all()
# Review results and fix any errors before proceeding

# 9. Export STL (only after passing all checks)
bpy.ops.export_mesh.stl(
    filepath="/path/to/print.stl",
    use_selection=True,
    global_scale=1.0,  # Respect scene units
    use_mesh_modifiers=True
)
```

## Remember

- **Always check scene unit settings first**: Respect the configured default profile
- **ALWAYS run 3D Print Toolbox check**: Never export without running `bpy.ops.mesh.print3d_check_all()` first
- **Use screenshots**: Visual feedback prevents errors
- **Execute incrementally**: Small steps, validate each one
- **Fix all errors before export**: The 3D Print Toolbox will identify issues - fix them all
- **AI models need work**: Generated models always need preparation and scaling
- **Use global_scale=1.0**: Let Blender's scene units control export scaling
- **Test prints**: Start small to validate model integrity
