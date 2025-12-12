# Adjusting Project Settings

Before processing your images, it's important to configure your project settings to match your workflow requirements. The Project Settings <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> panel provides comprehensive control over calibration, processing options, multispectral indices, and export formats.

## Accessing Project Settings

1. Open your project in Chloros
2. Click the **Project Settings** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> icon in the left sidebar
3. The Project Settings panel displays all configuration options

{% hint style="info" %}
**Settings are saved automatically** with your project. When you reopen a project, all settings are restored.
{% endhint %}

***

## Quick Setup for Common Workflows

### Default Settings (Recommended for Most Users)

For typical MAPIR Survey3 camera workflows, the default settings work well:

* ✅ **Vignette correction**: Enabled
* ✅ **Reflectance calibration**: Enabled (requires images of MAPIR targets)
* ✅ **Debayer method**: High Quality (Faster)
* ✅ **Export format**: TIFF (16-bit)

Simply import your images and start processing with these defaults.

***

## Project Settings Overview

The Project Settings panel is organized into several categories. Below is a summary of each section. For complete documentation, see [Project Settings](../project-settings/page-2.md).

### Target Detection

Controls how Chloros identifies calibration targets in your images.

**Key settings:**

* **Minimum calibration sample area**: Size threshold for target detection (default: 25 pixels)
* **Minimum target clustering**: Similarity threshold for grouping target regions (default: 60)

**When to adjust:**

* Increase sample area if getting false detections
* Decrease if targets aren't being detected
* Adjust clustering if targets are being split into multiple detections

### Processing

Main image processing and calibration options.

**Key settings:**

* **Vignette correction**: Compensates for lens darkening at edges ✅ Recommended
* **Reflectance calibration**: Normalizes values using calibration targets ✅ Recommended
* **Debayer method**: Algorithm for converting RAW to 3-channels multi-spectral
* **Minimum recalibration interval**: Time between using calibration targets (0 = use all)

**Advanced settings:**

* **Light sensor timezone offset**: For PPK time synchronization (default: 0)
* **Apply PPK corrections**: Uses GPS/exposure pin data from .daq files
* **Exposure Pin 1/2**: Assigns cameras to exposure pins for dual-camera setups

### Index (Multispectral Indices)

Configure which vegetation indices to calculate and export.

**How to add indices:**

1. Click **"Add index"** button
2. Select an index from the dropdown menu (NDVI, NDRE, GNDVI, etc.)
3. Configure visualization settings (LUT colors, value ranges)
4. Add multiple indices as needed

**Popular indices:**

* **NDVI**: General vegetation health (most common)
* **NDRE**: Early stress detection with RedEdge
* **GNDVI**: Chlorophyll concentration sensitive
* **OSAVI**: Works well with visible soil
* **EVI**: High leaf area index (LAI) regions

**Custom formulas (Chloros+ only):**

* Create custom multispectral index formulas
* Use band math with all image channels
* Save custom formulas for reuse

For all available indices and formulas, see [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

### Export

Controls output file format and quality.

**Available formats:**

* **TIFF (16-bit)**: Recommended for GIS and scientific analysis (0-65,535 range)
* **TIFF (32-bit, Percent)**: Floating-point reflectance values (0.0-1.0 range)
* **PNG (8-bit)**: Lossless compression for visualization (0-255 range)
* **JPG (8-bit)**: Smallest files, lossy compression (0-255 range)

***

## Saving and Loading Settings

### Save Project Template

Create reusable templates for consistent workflows:

1. Configure all desired settings in the Project Settings panel
2. Scroll to **"Save Project Template"** section at the bottom
3. Enter a descriptive template name (e.g., "Survey3N\_RGN\_Agriculture")
4. Click the save icon

**Benefits:**

* Apply identical settings across multiple projects
* Share configurations with team members
* Maintain consistency for repeated surveys

### Load Template on New Project

When creating a new project:

1. Select **"New Project"** from main menu
2. Choose **"Load from template"** option
3. Select your saved template
4. All settings are automatically applied

### Working Directory

The **"Save Project Folder"** setting specifies where new projects are created by default:

* **Default location**: `C:\Users\[Username]\Chloros Projects`
* **Change location**: Click edit icon and select new folder
* **When to change**:
  * Network drive for team collaboration
  * Different drive with more storage space
  * Organized folder structure by year/client

***

## PPK (Post-Processed Kinematic) Setup

If using MAPIR DAQ recorders with GPS for precise geolocation:

### Prerequisites

* MAPIR DAQ with GPS (GNSS) module
* .daq log file with exposure pin entries
* Camera connected to DAQ exposure pins during capture session

### Configuration Steps

1. Place the .daq log file in your project folder
2. In Project Settings, enable **"Apply PPK corrections"** checkbox
3. Set **"Light sensor timezone offset"** if needed (default: 0 for UTC)
4. Assign cameras to exposure pins:
   * **Single camera**: Automatically assigned to Pin 1
   * **Dual cameras**: Manually assign each camera to correct pin

**Exposure Pin Assignment:**

* **Exposure Pin 1**: Select camera model from dropdown
* **Exposure Pin 2**: Select second camera or "Do Not Use"
* Same camera cannot be assigned to both pins

{% hint style="warning" %}
**Important**: Exposure pins must be correctly assigned to their respective cameras. Incorrect assignment will result in wrong geolocation data.
{% endhint %}

***

## Advanced Scenarios

### Multi-Camera Projects

When processing images from multiple MAPIR cameras in one project:

1. Chloros automatically detects each camera model
2. Each camera gets appropriate processing profile
3. PPK: Manually assign each camera to correct exposure pin
4. All cameras use same export format and indices

**Example**: Survey3W RGN + Survey3N OCN dual-camera rig

### Time-Lapse or Multi-Date Surveys

For repeated surveys of the same area over time:

1. Create a template with your standard settings
2. Use consistent calibration target setup each session
3. Process each date as a separate project
4. Use identical settings for comparable results
5. Export in same format for temporal analysis

### Large Datasets

For projects with many images (500+):

* Consider breaking into smaller projects by date or area
* Use Chloros+ parallel processing for faster results
* Consider CLI or API for batch automation
* Adjust minimum recalibration interval to reduce target detection time

***

## Verifying Your Settings

Before starting to process, review these key settings:

* [ ] Camera model correctly detected in File Browser
* [ ] Vignette correction enabled
* [ ] Reflectance calibration enabled
* [ ] At least one calibration target image imported
* [ ] Desired multispectral indices added
* [ ] Export format appropriate for your workflow
* [ ] PPK settings configured (if using .daq with expposure events)

***

## Next Steps

Once your settings are configured:

1. **Mark calibration target images** - See [Choosing Target Images](choosing-target-images.md)
2. **Start processing** - See [Starting the Processing](starting-the-processing.md)
3. **Monitor progress** - See [Monitoring the Processing](monitoring-the-processing.md)

For complete details on all available settings, see the [Project Settings](../project-settings/page-2.md) reference documentation.
