# Finishing the Processing

Once Chloros completes processing, it's time to review your results, verify output quality, and prepare your processed images for use in your workflow. This page guides you through the final steps and next actions.

## Processing Complete Indication

When processing finishes successfully, you'll see several indicators:

* ✅ **Progress bar**: Reaches 100% completion
* ✅ **Debug Log**: Shows "Processing Complete" message
* ✅ **Start button**: Becomes enabled again (ready for next processing run)
* ✅ **Output files**: All processed images saved to camera model subfolder

***

## Locating Your Processed Images

### Opening the Output Folder

1. Click the **Main Menu** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> icon (top left)
2. Select **"Open Project Folder"**
3. Your file explorer opens to the project directory
4. Locate your project by name

***

## Reviewing Processed Images

### Quick Preview in File Explorer

**Windows built-in preview:**

1. Navigate to camera model subfolder
2. Select an image file
3. Preview appears in Windows Explorer preview pane
4. Use arrow keys to browse through images

### Preview in External Image Viewers

**Recommended viewers:**

* **QGIS** - Free GIS software (best for georeferenced multispectral analysis)
* **IrfanView** - Fast, lightweight image viewer (supports TIFF)
* **Adobe Photoshop** - Professional editing (TIFF support)
* **GIMP** - Free alternative to Photoshop
* **Windows Photos** - Basic viewing (may not support 16-bit TIFF)

### Preview in Chloros Image Viewer

Use Chloros's built-in Image Viewer for advanced visualization:

1. Click an image thumbnail in the File Browser
2. Image opens in the main preview area
3. Click **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab in left sidebar
4. Use [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) for interactive analysis

See [Image Viewer](../image-viewer-gui/page-3.md) for detailed instructions.

***

## Reviewing the Debug Log

### Check for Warnings or Errors

1. Open **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> tab
2. Scroll through messages
3. Look for yellow warnings or red errors
4. Review any issues noted
5. Contact MAPIR support for assistance

### Saving the Log

To keep a record of processing or to send to MAPIR Support:

1. Click **"Copy"** or **"Download"** button
2. Save as text file in project folder
3. Include with project documentation
4. Send to MAPIR support if issues encountered

***

## Common Output Issues and Solutions

### Issue: Missing Output Files

**Possible causes:**

* Files didn't meet processing criteria
* Target-only images (excluded from export)
* Disk space ran out during export
* File corruption during processing

**Solutions:**

1. Check Debug Log for skip/error messages
2. Verify disk space was sufficient
3. Count files: Should match (original count - target count) × (indices + 1)
4. Re-import and reprocess any missing files

### Issue: Dark or Bright Edges (Vignetting Still Visible)

**Possible causes:**

* Vignette correction disabled
* Camera/lens not in Chloros profile database
* Extreme vignetting beyond correction capability

**Solutions:**

1. Verify vignette correction was enabled in Project Settings
2. Check camera model correctly detected
3. Contact MAPIR support if vignetting persists

### Issue: Incorrect Colors or Values

**Possible causes:**

* No calibration targets detected
* Wrong calibration target model selected
* Reflectance calibration disabled
* Poor quality target images

**Solutions:**

1. Verify reflectance calibration was enabled
2. Check "Target found" messages in Debug Log
3. Review target image quality
4. Reprocess with proper targets marked

### Issue: NDVI Values Seem Wrong

**Expected NDVI ranges:**

* **Water, rocks, soil**: -0.1 to 0.2
* **Sparse/unhealthy vegetation**: 0.2 to 0.4
* **Moderate vegetation**: 0.4 to 0.6
* **Healthy, dense vegetation**: 0.6 to 0.9

**If values are outside these ranges:**

1. Verify reflectance calibration was applied
2. Verify light sensor log was included
3. Check calibration targets were detected
4. Ensure correct camera model was detected
5. Review target image capture timing and conditions

***

## Using Your Processed Images

### For Photogrammetry / Orthomosaic Creation

**Recommended workflow:**

1. **Import calibrated reflectance images** into photogrammetry software:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Keep EXIF metadata**: Ensure GPS data preserved for geotagging
3. **Calibrated workflows**: Use reflectance images for scientific accuracy
4. **Process index mosaics**: Create NDVI orthomosaics from individual index images
5. **Export georeferenced GeoTIFF**: For use in GIS applications

### For GIS Analysis

**Recommended workflow:**

1. **Load into QGIS, ArcGIS, or similar**
2. **Use 16-bit TIFF** reflectance images for multi-band analysis
3. **Use index images** (NDVI, NDRE) as ready-to-use vegetation layers
4. **Raster calculator**: Combine bands for custom analysis
5. **Export**: Create classification maps, change detection, vegetation health maps

### For Direct Analysis / Reporting

**Recommended workflow:**

1. **Use index images with LUT colors** for visual reports
2. **Extract statistics**: Mean NDVI per field/plot
3. **Time series**: Compare indices across multiple sessions
4. **Generate reports**: Include maps, statistics, and visualizations

***

## Archiving and Backup

### Recommended Backup Strategy

**What to save:**

* ✅ **Original RAW/JPG images** - Archive on separate drive/cloud
* ✅ **Processed outputs** - Keep calibrated images and indices
* ✅ **Project file** - Contains all settings for reprocessing if needed
* ✅ **Debug Log** - Documents processing details
* ✅ **Calibration target images** - For verification and reprocessing

**Storage recommendations:**

* **Immediate backup**: External hard drive
* **Long-term archive**: Cloud storage (Google Drive, Dropbox, etc.)
* **Critical data**: Keep 2-3 copies in different locations

***

## Next Processing Runs

### Reusing Project Settings

If processing similar datasets in the future:

1. **Save Project Template** (if not already done)
2. **Create new project** using saved template
3. **Import new images**
4. **Process** with identical settings for consistency

### Batch Processing Multiple Sessions

For multiple sessions/datasets:

**Option 1: GUI - Multiple Projects**

* Create separate project for each session
* Use consistent template settings
* Process one at a time

**Option 2: Chloros CLI (Chloros+ only)**

* Automate batch processing
* Process multiple folders with scripts
* See [CLI Documentation](../CLI.md)

**Option 3: Python SDK (Chloros+ only)**

* Programmatic control
* Integration with analysis pipelines
* See [API Documentation](../api-python-sdk.md)

***

## Troubleshooting Post-Processing

### Re-Processing with Different Settings

If results aren't satisfactory:

1. Keep original images (never delete)
2. Open same project in Chloros
3. Adjust settings in Project Settings panel
4. Process again - outputs will overwrite previous results

### Processing Subset of Images

To reprocess only specific images:

1. Create new project
2. Import only the images needing reprocessing
3. Use same settings template
4. Process smaller dataset

### Getting Help

If you encounter issues:

* 📧 **Email**: info@mapir.camera (include Debug Log)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Frequently Asked Questions](../faq.md)
* 📖 **Documentation**: [Chloros Manual](../)

***

## Summary: Complete Workflow

You've now completed the full Chloros processing workflow:

1. ✅ **Created project** - See [Projects](../projects.md)
2. ✅ **Added files** - See [Adding Files](page-1.md)
3. ✅ **Adjusted settings** - See [Adjusting Project Settings](adjusting-project-settings.md)
4. ✅ **Marked targets** - See [Choosing Target Images](choosing-target-images.md)
5. ✅ **Started processing** - See [Starting the Processing](starting-the-processing.md)
6. ✅ **Monitored progress** - See [Monitoring the Processing](monitoring-the-processing.md)
7. ✅ **Reviewed results** - This page

**Your calibrated, reflectance-corrected multispectral images are ready for analysis!**

***

## Additional Resources

### Advanced Features

* [**Image Viewer**](../image-viewer-gui/page-3.md) - Interactive visualization and analysis
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Custom index testing
* [**Multispectral Index Formulas**](../project-settings/multispectral-index-formulas.md) - Complete index reference

### Automation & Integration

* [**CLI Documentation**](../CLI.md) - Command-line batch processing
* [**Python SDK**](../api-python-sdk.md) - Programmatic automation
* [**Chloros+ Features**](../#chloros) - Advanced processing capabilities

### Support & Learning

* [**FAQ**](../faq.md) - Common questions answered
* [**Calibration Targets**](../calibration-targets.md) - Understanding reflectance calibration
* [**Supported Cameras**](../supported-cameras.md) - Compatible hardware
