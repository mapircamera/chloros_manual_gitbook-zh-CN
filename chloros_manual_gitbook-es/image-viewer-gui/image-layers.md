# Image Layers

The Image Layers dropdown in the Chloros Image Viewer allows you to quickly switch between different versions of the same image - from the original captures to processed reflectance outputs and calculated index images.

## What are Image Layers?

In Chloros, **layers** refer to the different image outputs available for a single source image. When you process images, Chloros creates multiple versions:

* **Original images** (JPG and RAW files from your camera)
* **Reflectance calibrated** outputs (if reflectance calibration was enabled)
* **Target images** (if the image contains calibration targets)
* **Index images** (NDVI, NDRE, GNDVI, etc. if indices were configured)

The **Layer Selector dropdown** in the top-right of the Image Viewer lets you instantly switch between these versions without leaving the viewer.

***

## Available Layer Types

### JPG

* The original JPG preview image from your camera
* Always available for all images
* Unprocessed, as captured by the camera
* Fastest to load and display

**When to view:**

* Quick preview of original capture
* Checking image composition and framing
* Verifying capture quality before processing

### RAW (Original)

* The original RAW sensor data from your camera
* Debayered with no post processing applied
* Higher bit depth than JPG (typically 12-bit or 14-bit sensor data)

**When to view:**

* Inspecting original sensor data quality
* Checking for sensor issues or artifacts
* Comparing before/after processing results

### RAW (Target)

* Only appears for images identified as containing calibration targets
* Shows the original RAW image with target detected
* Used to verify target detection was successful

**When to view:**

* Confirming calibration targets were detected correctly
* Checking target image quality
* Troubleshooting calibration issues

{% hint style="info" %}
**Target Layer**: This layer only appears in the dropdown for images that contain calibration targets. Regular capture images will not have this option.
{% endhint %}

### RAW (Reflectance)

* The calibrated reflectance output image
* Vignette corrected (if enabled in processing)
* Reflectance calibrated using target data (if enabled)
* Multi-band TIFF with all camera channels
* Pixel values represent percent reflectance (when using percent mode)
* Ready to manipulate with the [Index/LUT Sandbox](index-lut-sandbox.md)

**When to view:**

* Inspecting calibrated results
* Verifying calibration quality
* Checking pixel values for scientific accuracy
* Comparing with original to see calibration effects

{% hint style="success" %}
**Recommended**: Use RAW (Reflectance) layer when checking pixel values for scientific measurements and analysis.
{% endhint %}

### RAW (NDVI Index)... and similar

* Calculated vegetation index image (NDVI in this example)
* The index name changes based on which index was configured during processing
* Examples: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index), etc.
* Single-band grayscale image showing index calculation results
* One layer appears for each index configured in Project Settings

**Possible index names:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* And many more... (see [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md))

**When to view:**

* Examining index calculation results
* Checking index value ranges
* Identifying areas of interest
* Verifying index images before using in GIS or analysis

***

## Using the Layer Selector

### Opening the Dropdown

1. Open an image in fullscreen mode (click any thumbnail in the Image Viewer)
2. Locate the **layer dropdown** in the top-right corner of the viewer
3. The dropdown shows the currently selected layer (e.g., "JPG")
4. Click the dropdown to see all available layers

### Switching Layers

1. Click the layer dropdown to open the list
2. All available layers for the current image are shown
3. Click any layer name to switch to that version
4. The image updates immediately to show the selected layer

**Quick switching:**

* The dropdown remembers your last selection
* When navigating to the next image, Chloros attempts to show the same layer type
* If that layer doesn't exist on the next image, it defaults to JPG

### Layer Availability

Not all layers are available for every image:

**Always available:**

* ✅ JPG (every image has a JPG preview)

**Conditionally available:**

* ⚠️ RAW (Original) - Only if image was captured in RAW or RAW+JPG mode
* ⚠️ RAW (Target) - Only if image contains detected calibration targets
* ⚠️ RAW (Reflectance) - Only after processing with reflectance calibration enabled
* ⚠️ RAW (\[Index] Index) - Only after processing with indices configured

***

## Layer Persistence

### Navigating Between Images

When you navigate to a different image (using arrow keys or clicking thumbnails):

**Layer preference is preserved:**

* If viewing "RAW (Reflectance)", next image shows "RAW (Reflectance)" (if available)
* If viewing "RAW (NDVI Index)", next image shows "RAW (NDVI Index)" (if available)
* If the same layer doesn't exist, defaults to JPG

**Example workflow:**

1. Open Image 1, switch to RAW (NDVI Index)
2. Press → to view Image 2
3. Image 2 automatically displays RAW (NDVI Index) layer
4. Continue navigating - all images show NDVI layer
5. Very efficient for reviewing index results across many images

***

## Common Workflows

### Workflow 1: Before/After Comparison

**Goal**: Compare original vs. calibrated image

1. Open processed image in Image Viewer
2. Select **RAW (Original)** from dropdown
3. Note the vignetting and uncalibrated values
4. Switch to **RAW (Reflectance)** from dropdown
5. Compare - vignetting removed, values calibrated

### Workflow 2: Index Review

**Goal**: Quickly review NDVI results across dataset

1. Open first processed image
2. Select **RAW (NDVI Index)** from dropdown
3. Use → arrow key to navigate to next image
4. NDVI layer persists automatically
5. Continue through all images, checking NDVI patterns
6. Switch to **RAW (NDRE Index)** to compare

### Workflow 3: Target Verification

**Goal**: Verify all target images were detected correctly

1. Navigate to a target image
2. Select **RAW (Target)** from dropdown
3. Verify calibration targets are clearly visible and detected
4. Navigate to next target image
5. Repeat verification for all targets

### Workflow 4: Pixel Value Inspection

**Goal**: Check reflectance values for scientific accuracy

1. Open processed image
2. Select **RAW (Reflectance)** layer
3. Enable **Pixel Percent** mode (button in top-right toolbar)
4. Move cursor over vegetation areas
5. Verify pixel values are in expected ranges (30-70% for NIR, 5-15% for Red)
6. Check soil and water areas for appropriate values

***

## Understanding Pixel Values by Layer

Different layers show different pixel value ranges:

### JPG Layer

* **Range**: 0-255 (8-bit)
* **Meaning**: Display values, gamma-corrected
* **Use**: Visual inspection only, not for scientific measurement

### RAW (Original)

* **Range**: 0-65535 (16-bit)
* **Meaning**: Raw sensor digital numbers
* **Use**: Checking sensor performance, not calibrated

### RAW (Reflectance)

* **Range**: 0-65,535 (16-bit TIFF) or 0.0-1.0 (32-bit Percent)
* **Meaning**: Calibrated percent reflectance
* **Use**: Scientific measurements and analysis

**For 16-bit TIFF:** Divide by 65,535 to get percent reflectance **For 32-bit Percent:** Values directly represent percent (0.5 = 50% reflectance)

### RAW (Index Images)

* **Range**: Varies by index (typically -1.0 to +1.0 for normalized indices)
* **Meaning**: Index calculation result
* **Examples**:
  * NDVI: -1 to +1 (vegetation typically 0.4 to 0.9)
  * NDRE: -1 to +1 (stress detection)
  * EVI: 0 to 1 (enhanced vegetation)

***

## Tips and Best Practices

### Efficient Layer Switching

* **Keyboard shortcut awareness**: While there's no keyboard shortcut for layers, navigation arrows (←/→) work across all layers
* **Consistent workflows**: Pick one layer (e.g., NDVI) and review entire dataset before switching to another
* **Quick comparisons**: Toggle between Original and Reflectance to verify processing quality

### Performance Considerations

* **JPG loads fastest**: Use for quick navigation through many images
* **RAW layers load slower**: Higher resolution and bit depth
* **Index layers**: Similar speed to Reflectance layers
* **First load is slowest**: Subsequent views of same layer are cached and faster

### Quality Verification

* **Always check RAW (Original)**: Verify source data quality before trusting processed outputs
* **Compare layers**: Use layer switching to validate processing worked correctly
* **Check index ranges**: Use Pixel Percent mode with index layers to verify values are reasonable

***

## Troubleshooting

### Layer Not Available

**Problem**: Expected layer doesn't appear in dropdown

**Possible causes:**

* Image wasn't processed (only JPG and RAW (Original) available)
* Reflectance calibration was disabled during processing
* Specific index wasn't configured in Project Settings
* Image is a target-only image (no indices generated for targets)

**Solutions:**

1. Verify image was processed (check output folder for processed files)
2. Check Project Settings to confirm indices were configured
3. Reprocess with desired indices enabled

### Wrong Layer Shown

**Problem**: Image opens in unexpected layer

**Cause**: Layer preference from previous image carried forward, but that layer doesn't exist on current image

**Solution**: Chloros automatically falls back to JPG when preferred layer unavailable - this is normal behavior

### Can't See Calibration Targets

**Problem**: RAW (Target) layer doesn't show target detection

**Possible causes:**

* Targets weren't detected during processing
* Image doesn't actually contain targets
* Target detection settings too strict

**Solutions:**

1. Check Debug Log for "Target found" messages
2. Verify image actually contains visible calibration targets
3. Adjust target detection settings in Project Settings
4. See [Choosing Target Images](../processing-images-gui/choosing-target-images.md)

***

## Related Features

### Image Viewer Tools

When viewing any layer, you can use:

* **Zoom controls**: Magnify to inspect details
* **Pan**: Click and drag to move around zoomed image
* **Pixel value inspection**: See values at cursor location
* **Navigation arrows**: Move between images while maintaining layer
* **Pixel Percent mode**: Toggle between DN and percent display

See [Opening an Image Full Screen](page-3.md) for complete Image Viewer documentation.

### Index/LUT Sandbox

For interactive index testing and visualization:

* **Real-time index calculation**: Test different index formulas
* **LUT color mapping**: Apply color gradients to grayscale indices
* **Export visualizations**: Save colored index images

See [Index/LUT Sandbox](index-lut-sandbox.md) for details.

***

## Next Steps

Now that you understand image layers:

* [**Opening an Image Full Screen**](page-3.md) - Complete Image Viewer guide
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Interactive index visualization
* [**Multispectral Index Formulas**](../project-settings/multispectral-index-formulas.md) - Available indices reference
* [**Finishing the Processing**](../processing-images-gui/finishing-the-processing.md) - Understanding processed outputs
