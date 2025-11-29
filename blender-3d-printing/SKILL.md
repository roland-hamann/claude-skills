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

**If the connection fails**, see [Troubleshooting Guide](references/troubleshooting.md#mcp-connection-issues).

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

### FIRST: Search for Existing 3D Print Files

**ALWAYS check online repositories before creating or generating models.**

Most models already exist, are print-tested, and free to use. Search can save hours of work.

**See [Online Sources Guide](references/online-sources.md) for:**
- Complete list of free and paid repositories
- Search strategies and quality evaluation
- How to present findings categorized by free vs. paid
- When to use existing vs. create new

**Quick search process:**
1. Extract key search terms from user's description
2. Search top free repositories (MakerWorld, Printables, Thingiverse, MyMiniFactory)
3. Evaluate results for quality and print-readiness
4. Search paid repositories if free options insufficient
5. Present findings in two categories: **Free** and **Paid**
6. Let user decide: use existing or proceed with creation

### If No Suitable Existing Files Found

**AI Generation (text-to-3D or image-to-3D)**
- Use Hyper3D Rodin for fast generation
- Use Hunyuan3D for high-quality results
- **See [AI Generation Guide](references/ai-generation.md) for complete workflow**

**Downloading from Asset Libraries**
- Use Polyhaven for textures, HDRIs, and high-quality models
- Use Sketchfab for community models
- **See [MCP Tools Reference](references/mcp-tools.md) for tool details**

**Creating from Scratch**
- Use Python code execution for procedural modeling
- See "Procedural Modeling Workflow" below

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

**Need help?** See [Troubleshooting Guide](references/troubleshooting.md)

## Available MCP Tools

For complete tool reference including parameters and examples, see [MCP Tools Reference](references/mcp-tools.md).

**Quick overview:**
- Scene and object information tools
- Python code execution
- Polyhaven integration (textures, HDRIs, models)
- Sketchfab integration (community models)
- Hyper3D Rodin (AI generation)
- Hunyuan3D (AI generation)

## Import and Prepare Existing Model Workflow

This is the most common workflow for preparing downloaded or existing models for 3D printing.

### 1. Import the model

**From Sketchfab/Polyhaven:**
Use respective download tools (see [MCP Tools Reference](references/mcp-tools.md))

**From file:**
```python
import bpy

# Import STL
bpy.ops.import_mesh.stl(filepath="/path/to/model.stl")

# Import OBJ
bpy.ops.import_scene.obj(filepath="/path/to/model.obj")

# Import other formats as needed
```

### 2. Check the model

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

### 3. Scale to correct size

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

### 4. Fix common issues

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

### 5. Run 3D Print Toolbox check (MANDATORY)

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

### 6. Export for printing (only after passing 3D Print check)

```python
# Export as STL
bpy.ops.export_mesh.stl(
    filepath="/path/to/output.stl",
    use_selection=True,
    global_scale=1.0,
    use_mesh_modifiers=True
)
```

## Procedural Modeling Workflow

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

- **Search online first**: Always check existing 3D print files before creating - saves hours of work
- **Always check scene unit settings first**: Respect the configured default profile
- **ALWAYS run 3D Print Toolbox check**: Never export without running `bpy.ops.mesh.print3d_check_all()` first
- **Use screenshots**: Visual feedback prevents errors
- **Execute incrementally**: Small steps, validate each one
- **Fix all errors before export**: The 3D Print Toolbox will identify issues - fix them all
- **AI models need work**: Generated models always need preparation and scaling
- **Use global_scale=1.0**: Let Blender's scene units control export scaling
- **Test prints**: Start small to validate model integrity

## Additional Resources

- **[Online Sources Guide](references/online-sources.md)** - Search strategies for free and paid 3D print file repositories
- **[MCP Tools Reference](references/mcp-tools.md)** - Complete reference for all Blender MCP tools
- **[AI Generation Guide](references/ai-generation.md)** - Detailed workflows for Hyper3D Rodin and Hunyuan3D
- **[Troubleshooting Guide](references/troubleshooting.md)** - Solutions for common issues
