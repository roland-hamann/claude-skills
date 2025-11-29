# Troubleshooting Guide

Common issues and solutions when preparing 3D models for printing.

## Non-Manifold Geometry Detected

**Problem**: The model has edges that don't connect exactly 2 faces, making it unprintable.

**Solutions**:

1. **Select non-manifold geometry**:
```python
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='DESELECT')
bpy.ops.mesh.select_non_manifold()
```

2. **Remove duplicate vertices**:
```python
bpy.ops.mesh.remove_doubles(threshold=0.0001)
```

3. **Fill holes manually**:
```python
bpy.ops.mesh.edge_face_add()
```

4. **Check for intersecting faces**:
- Use 3D Print Toolbox to identify intersections
- Separate or merge intersecting geometry
- Use Boolean operations if needed

## Model is Too Small/Large

**Problem**: Model dimensions don't match intended print size.

**Solutions**:

1. **Check current dimensions** (in scene units):
```python
obj = bpy.context.active_object
print(f"Dimensions: {obj.dimensions}")
```

2. **Set specific dimensions**:
```python
# Method 1: Direct dimension setting
obj.dimensions = (50, 50, 100)  # x, y, z in scene units
bpy.ops.object.transform_apply(scale=True)

# Method 2: Scale factor
target_height = 100.0  # in scene units
current_height = obj.dimensions.z
scale_factor = target_height / current_height
obj.scale = (scale_factor, scale_factor, scale_factor)
bpy.ops.object.transform_apply(scale=True)
```

3. **Verify scene units**:
```python
print(f"Unit system: {bpy.context.scene.unit_settings.system}")
print(f"Length unit: {bpy.context.scene.unit_settings.length_unit}")
```

## AI Generation Failed

**Problem**: AI model generation returns an error or fails to complete.

**Troubleshooting steps**:

1. **Check API status**:
```
# For Hyper3D
mcp__blender__get_hyper3d_status

# For Hunyuan3D
mcp__blender__get_hunyuan3d_status
```

2. **Verify prompt language**:
- Hyper3D Rodin: Use English
- Hunyuan3D: English or Chinese accepted

3. **Check image input**:
- MAIN_SITE mode: Ensure file paths are absolute and accessible
- FAL_AI mode: Ensure URLs are valid and accessible
- Verify image format is supported (JPG, PNG)

4. **Adjust polling interval**:
- Don't poll too frequently (wait 5-10 seconds between polls)
- Generation can take 1-5 minutes
- Check for rate limiting

5. **Simplify prompt**:
- Make description shorter and clearer
- Remove ambiguous or complex requests
- Try without special characters

## Export Failed

**Problem**: Export operation fails or produces corrupted file.

**Solutions**:

1. **Ensure object is selected**:
```python
obj = bpy.context.active_object
obj.select_set(True)
bpy.context.view_layer.objects.active = obj
```

2. **Apply all modifiers before export**:
```python
for mod in obj.modifiers:
    bpy.ops.object.modifier_apply(modifier=mod.name)
```

3. **Check file path**:
- Ensure path is valid and writable
- Use absolute paths
- Check directory exists
- Verify permissions

4. **Verify export format supports mesh**:
- STL: Mesh only, no materials
- OBJ: Mesh with materials
- 3MF: Advanced features, not all software supports

5. **Check for errors in 3D Print Toolbox**:
- Must pass all checks before export
- Fix all reported errors
- Re-run check after fixes

## Slicer Shows Errors or Weird Geometry

**Problem**: Slicer software shows errors, missing layers, or unexpected geometry even though the model looks fine in Blender.

**Common cause**: Internal faces confuse the slicer about what's inside vs. outside the model.

**Solutions**:

1. **Remove internal faces**:
```python
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='DESELECT')
bpy.ops.mesh.select_interior_faces()
bpy.ops.mesh.delete(type='FACE')
bpy.ops.object.mode_set(mode='OBJECT')
```

2. **Check for intersecting geometry**:
- Run 3D Print Toolbox check
- Look for "Intersections" warnings
- Separate or merge intersecting parts

3. **Verify the model after removing internal faces**:
```python
# Re-check with 3D Print Toolbox
bpy.ops.mesh.print3d_check_all()
```

4. **Re-export and test in slicer**:
- Export fresh STL after cleanup
- Verify in slicer preview
- Check layer view for anomalies

**Why this matters**: Internal faces don't affect how the model looks in Blender, but slicers need to understand the interior vs. exterior boundary. Internal faces break this understanding, causing print failures.

## Flipped or Incorrect Normals

**Problem**: Model appears inside-out or has rendering issues.

**Solutions**:

1. **Recalculate normals**:
```python
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.mesh.normals_make_consistent(inside=False)
bpy.ops.object.mode_set(mode='OBJECT')
```

2. **Flip normals if needed**:
```python
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.mesh.flip_normals()
bpy.ops.object.mode_set(mode='OBJECT')
```

3. **Check in viewport**:
- Enable face orientation overlay
- Blue = correct (outward)
- Red = incorrect (inward)

## Too Many Polygons / File Too Large

**Problem**: Model has excessive polygon count, making it slow or too large.

**Solutions**:

1. **Use Decimate modifier**:
```python
obj = bpy.context.active_object
mod = obj.modifiers.new(name="Decimate", type='DECIMATE')
mod.ratio = 0.5  # Reduce to 50% of original poly count
bpy.ops.object.modifier_apply(modifier="Decimate")
```

2. **Adjust ratio based on needs**:
- 0.5 = 50% reduction (moderate)
- 0.25 = 75% reduction (aggressive)
- 0.75 = 25% reduction (conservative)

3. **Check polygon count**:
```python
obj = bpy.context.active_object
mesh = obj.data
print(f"Polygons: {len(mesh.polygons)}")
print(f"Vertices: {len(mesh.vertices)}")
```

4. **Alternative: Remesh modifier**:
```python
mod = obj.modifiers.new(name="Remesh", type='REMESH')
mod.mode = 'VOXEL'
mod.voxel_size = 0.5  # Adjust for desired detail level
bpy.ops.object.modifier_apply(modifier="Remesh")
```

## Thin Walls / Zero Thickness

**Problem**: 3D Print Toolbox reports walls are too thin or have zero thickness.

**Solutions**:

1. **Use Solidify modifier**:
```python
obj = bpy.context.active_object
mod = obj.modifiers.new(name="Solidify", type='SOLIDIFY')
mod.thickness = 2.0  # Thickness in scene units (e.g., 2mm)
bpy.ops.object.modifier_apply(modifier="Solidify")
```

2. **Adjust thickness based on printer**:
- FDM printers: Minimum 1-2mm
- Resin printers: Minimum 0.8mm
- Consider printer capabilities

3. **Check after applying**:
```python
bpy.ops.mesh.print3d_check_all()
```

## MCP Connection Issues

**Problem**: Cannot connect to Blender MCP server.

**Solutions**:

1. **Verify Blender is running**:
- Check Blender application is open
- Verify no crash or freeze

2. **Check MCP server configuration**:
- Ensure MCP server is enabled in Blender
- Verify port settings are correct
- Check firewall isn't blocking connection

3. **Restart Blender**:
- Close Blender completely
- Reopen Blender
- Retry connection check

4. **Check MCP server logs**:
- Look for error messages
- Verify server started successfully
- Check for port conflicts

5. **Test connection**:
```
mcp__blender__get_scene_info
```
