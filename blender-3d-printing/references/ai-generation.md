# AI Model Generation Guide

Complete guide for generating 3D models using AI tools integrated with Blender.

## Workflow: AI Generate Model for Printing

### Using Hyper3D Rodin

**1. Check status** (remember key type for later use)
```
mcp__blender__get_hyper3d_status
```

**2. Generate model**
- Text prompt: `mcp__blender__generate_hyper3d_model_via_text`
- Image input: `mcp__blender__generate_hyper3d_model_via_images`
- Optional: Use `bbox_condition` to control proportions [Length, Width, Height]

**3. Poll for completion**
```
mcp__blender__poll_rodin_job_status
```
Use `subscription_key` (MAIN_SITE) or `request_id` (FAL_AI) based on key type

**4. Import when complete**
```
mcp__blender__import_generated_asset
```
Provide object name and `task_uuid` (MAIN_SITE) or `request_id` (FAL_AI)

**5. Prepare for printing**
After import, follow the standard preparation workflow:
- Check scale and dimensions
- Check for non-manifold geometry
- Fix common issues
- Run 3D Print Toolbox check
- Export

### Using Hunyuan3D

**1. Check status** (remember key type)
```
mcp__blender__get_hunyuan3d_status
```

**2. Generate model**
```
mcp__blender__generate_hunyuan3d_model
```
Provide `text_prompt` and/or `input_image_url`

**3. Poll for completion**
```
mcp__blender__poll_hunyuan_job_status
```
Returns status "DONE" when complete, includes `ResultFile3Ds` with ZIP path

**4. Import when complete**
```
mcp__blender__import_generated_asset_hunyuan
```
Provide object name and `zip_file_url` from poll result

**5. Prepare for printing**
After import, follow the standard preparation workflow:
- Check scale and dimensions
- Check for non-manifold geometry
- Fix common issues
- Run 3D Print Toolbox check
- Export

## AI Generation Technical Details

### Hyper3D Rodin

**Modes**
- MAIN_SITE: Uses main Hyper3D API
- FAL_AI: Uses FAL.AI integration
- Mode is determined by the key type configured

**Text Prompts**
- Must be in English
- Short descriptions work best
- Be specific but concise

**Image Input**
- MAIN_SITE mode: Provide absolute file paths
- FAL_AI mode: Provide URLs to images
- Can use single or multiple images

**bbox_condition**
Controls model proportions using [Length, Width, Height] ratios:
- `[1.0, 1.0, 1.0]`: Cube proportions
- `[1.0, 1.0, 2.0]`: Tall model (2x height)
- `[2.0, 1.0, 1.0]`: Wide model (2x length)

**Generated Models**
- Always normalized size (need rescaling for printing)
- May have built-in materials
- Require manifoldness check before printing

### Hunyuan3D

**Input Options**
- Text only: Provide `text_prompt`
- Image only: Provide `input_image_url`
- Both: Use both parameters for better results

**Language Support**
- Accepts both Chinese and English prompts
- English recommended for consistency

**Image URLs**
- Supports both local file paths and remote URLs
- Ensure URLs are accessible from the system

**Output Format**
- Returns OBJ format in ZIP file
- Includes materials and textures when applicable

**Generated Models**
- Normalized size (need rescaling for printing)
- Generally high quality
- Require manifoldness check before printing

## Post-Generation Workflow

After generating any AI model, always follow these steps:

**1. Generate model**
- Returns job ID for tracking

**2. Poll status until complete**
- Hyper3D: Poll until "Done" or "COMPLETED"
- Hunyuan3D: Poll until "DONE"
- Don't poll too frequently (respect rate limits)

**3. Import the generated asset**
- Use appropriate import function for the service
- Provide a descriptive name for the object

**4. Always rescale**
- AI models are normalized (typically unit size)
- Scale to intended print dimensions
- Use scene's configured units (millimeters)

**5. Always check manifoldness**
- AI may generate non-printable geometry
- Run 3D Print Toolbox check
- Fix any detected issues

**6. Apply additional fixes as needed**
- Remove doubles
- Fill holes
- Fix normals
- Optimize polygon count if necessary

**7. Export for printing**
- Only after passing all checks
- Use appropriate file format (STL, OBJ, 3MF)

## Generation Best Practices

**Text Prompts**
- Be specific and descriptive
- Include key features and details
- Mention intended use if relevant
- Keep prompts concise

**Image Input**
- Use clear, well-lit images
- Multiple angles improve results
- Remove background clutter if possible
- Higher resolution generally better

**Polling**
- Don't poll more than once every 5-10 seconds
- Generation can take 1-5 minutes depending on complexity
- Be patient with the process

**Post-Processing**
- Always assume AI output needs work
- Budget time for fixes and optimization
- Test small prints first to validate model
- Keep original if you need to regenerate
