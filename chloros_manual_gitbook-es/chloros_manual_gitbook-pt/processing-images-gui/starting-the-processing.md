# Starting the Processing

Once you've imported your images, marked your calibration targets, and configured your project settings, you're ready to begin processing. This page guides you through initiating the Chloros processing pipeline.

## Pre-Processing Checklist

Before clicking the Start button, verify that everything is ready:

* [ ] **Files imported** - All images appear in File Browser
* [ ] **Target images marked** - Target column checked for calibration images
* [ ] **Camera models detected** - Camera Model column shows correct cameras
* [ ] **Settings configured** - Project Settings reviewed and adjusted
* [ ] **Indices selected** - Desired multispectral indices added (if needed)
* [ ] **Export format chosen** - Output format appropriate for your workflow

{% hint style="info" %}
**Tip**: Click through a few images in the File Browser to verify they loaded correctly before processing.
{% endhint %}

***

## Starting the Processing

### Locate the Start Button

The Start/Play button is located in the top header bar of Chloros:

* Position: Top center of the window
* Icon: **Play/Start button** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: Button is enabled (bright) when ready to process

### Click to Start

1. Click the **Play/Start button** in the top header
2. Processing begins immediately
3. The button becomes disabled (grayed out) during processing
4. Progress bar updates, showing processing status

{% hint style="success" %}
**Processing Started**: Once clicked, Chloros automatically handles all processing steps - target detection, debayering, calibration, index calculation, and export.
{% endhint %}

***

## Understanding Processing Modes

Chloros operates in two different processing modes depending on your license:

### Free Mode (Sequential Processing)

**Available to all users**

**How it works:**

* Processes images one at a time, sequentially
* Single-threaded operation
* Lower memory usage

**Progress bar shows 2 stages:**

1. **Target Detect** - Scanning for calibration targets
2. **Processing** - Applying calibration and exporting images

**Processing time:**

* Much slower than Chloros+ parallel mode
* Suitable for small to medium datasets (< 200 images)

### Chloros+ Mode (Parallel Processing)

**Requires Chloros+ license**

**How it works:**

* Processes multiple images simultaneously
* Multi-threaded operation (up to 16 parallel workers)
* Utilizes multiple CPU cores
* Optional GPU (CUDA) acceleration with NVIDIA graphics cards

**Progress bar shows 4 stages:**

1. **Detecting** - Finding calibration targets
2. **Analyzing** - Examining image metadata and preparing pipeline
3. **Calibrating** - Applying corrections and calibrations
4. **Exporting** - Saving processed images and indices

**Progress bar interaction:**

* **Hover mouse** over bar to see detailed 4-stage dropdown panel
* **Click** progress bar to freeze the dropdown panel in place
* **Click again** to unfreeze and hide panel

**Processing time:**

* Significantly faster than free mode
* Scales with CPU core count
* GPU acceleration further improves speed

{% hint style="info" %}
**Chloros+ Speed**: Parallel processing can be 5-10x faster than sequential mode for large datasets. A 500-image project that takes 2 hours in free mode may complete in 15-20 minutes with Chloros+.
{% endhint %}

***

## What Happens During Processing

### Stage 1: Target Detection

**What Chloros does:**

* Scans marked target images (or all images if none marked)
* Identifies the 4 calibration panels in each target
* Extracts reflectance values from target panels
* Records target timestamps for calibration scheduling

**Duration:** 1-30 seconds (with marked targets), 5-30+ minutes (unmarked)

### Stage 2: Debayering (RAW Conversion)

**What Chloros does:**

* Converts RAW Bayer pattern data to full RGB images
* Applies high-quality demosaicing algorithm
* Preserves maximum image quality and detail

**Duration:** Varies by image count and CPU speed

### Stage 3: Calibration

**What Chloros does:**

* **Vignette correction**: Removes lens darkening at edges
* **Reflectance calibration**: Normalizes using target reflectance values
* Applies corrections across all bands/channels
* Uses appropriate calibration target for each image based on timestamp

**Duration:** Majority of processing time

### Stage 4: Index Calculation

**What Chloros does:**

* Calculates configured multispectral indices (NDVI, NDRE, etc.)
* Applies band math to calibrated images
* Generates index images for each selected index

**Duration:** A few seconds per image

### Stage 5: Export

**What Chloros does:**

* Saves calibrated images in selected format
* Exports index images with configured LUT colors
* Writes files to camera model subfolders
* Preserves original filenames with suffixes

**Duration:** Varies by export format and file size

***

## Processing Behavior

### Automatic Processing Pipeline

Once started, the entire pipeline runs automatically:

* No user interaction needed
* All configured steps execute in sequence
* Progress updates shown in real-time

### Computer Usage During Processing

**Free Mode:**

* Relatively low CPU usage (single-threaded)
* Computer remains responsive for other tasks
* Safe to minimize Chloros and work in other applications

**Chloros+ Parallel Mode:**

* High CPU usage (multi-threaded, up to 16 cores)
* With GPU acceleration: High GPU usage
* Computer may be less responsive during processing
* Avoid starting other CPU-intensive tasks

{% hint style="warning" %}
**Performance Tip**: For best Chloros+ performance, close other applications and let Chloros use full system resources.
{% endhint %}

### Processing Cannot Be Paused

**Important limitations:**

* Once started, processing cannot be paused
* You can cancel processing, but progress is lost
* Partial results are not saved
* Must restart from beginning if canceled

**Planning tip:** For very large projects, consider processing in batches or using CLI for better control.

***

## Monitoring Your Processing

While processing runs, you can:

* **Watch progress bar** - See overall completion percentage
* **View current stage** - Detect, Analyze, Calibrate, or Export
* **Check log tab** - See detailed processing messages and warnings
* **Preview completed images** - Some export files may appear during processing

For detailed information on monitoring, see [Monitoring the Processing](monitoring-the-processing.md).

***

## Canceling Processing

If you need to stop processing:

### How to Cancel

1. Locate the **Stop/Cancel button** (replaces Start button during processing)
2. Click the Stop button
3. Processing halts immediately
4. Partial results are discarded

### When to Cancel

**Valid reasons to cancel:**

* Realized incorrect settings were used
* Forgot to mark target images
* Wrong images imported
* System running too slow or unresponsive

**After canceling:**

* Review and fix any issues
* Adjust settings as needed
* Restart processing from the beginning
* For the cleanest experience, completely close Chloros and restart

{% hint style="warning" %}
**No Partial Results**: Canceling discards all progress. Chloros does not save partially processed images.
{% endhint %}

***

## Processing Time Estimates

Actual processing time varies greatly based on:

* Number of images
* Image resolution
* RAW vs JPG input format
* Processing mode (Free vs Chloros+)
* CPU speed and core count
* GPU availability (Chloros+ only)
* Number of indices to calculate
* Export format complexity

### Rough Estimates (Chloros+, 12MP images, modern CPU)

| Image Count | Free Mode | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 images   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 images  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 images  | 1-1.5 hrs | 20-30 min      | 10-15 min      |
| 500 images  | 2-3 hrs   | 45-60 min      | 20-30 min      |
| 1000 images | 4-6 hrs   | 1.5-2 hrs      | 40-60 min      |

{% hint style="info" %}
**First Run**: Initial processing may take longer as Chloros builds caches and profiles. Subsequent processing of similar datasets will be faster.
{% endhint %}

***

## Common Issues at Start

### Start Button Disabled (Grayed Out)

**Possible causes:**

* No images imported
* Backend not fully started
* Previous processing still running
* Project not fully loaded

**Solutions:**

1. Wait for backend to fully initialize (check main menu icon)
2. Verify images are imported in File Browser
3. Restart Chloros if button remains disabled
4. Check Debug Log for error messages

### Processing Starts Then Immediately Fails

**Possible causes:**

* No valid images in project
* Corrupted image files
* Insufficient disk space
* Insufficient memory (RAM)

**Solutions:**

1. Check Debug Log <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> for error messages
2. Verify disk space available
3. Try processing a smaller subset of images
4. Verify images are not corrupted

### "No Targets Detected" Warning

**Possible causes:**

* Forgot to mark target images
* Target images don't contain visible targets
* Target detection settings too strict

**Solutions:**

1. Review [Choosing Target Images](choosing-target-images.md)
2. Mark appropriate images in Target column
3. Verify targets are visible in marked images
4. Adjust target detection settings if needed

***

## Tips for Successful Processing

### Before Starting

1. **Test with small subset first** - Process 10-20 images to verify settings
2. **Check available disk space** - Ensure 2-3x dataset size free
3. **Close unnecessary applications** - Free up system resources
4. **Verify target images** - Preview marked targets to ensure quality
5. **Save project** - Project auto-saves, but good practice to save manually

### During Processing

1. **Avoid system sleep** - Disable power saving modes
2. **Keep Chloros in foreground** - Or at least visible in taskbar
3. **Monitor progress occasionally** - Check for warnings or errors
4. **Don't load other heavy applications** - Especially with Chloros+ parallel mode

### Chloros+ GPU Acceleration

If using NVIDIA GPU acceleration:

1. Update NVIDIA drivers to latest version
2. Ensure GPU has 4GB+ VRAM
3. Close GPU-intensive applications (games, video editing)
4. Monitor GPU temperature (ensure adequate cooling)

***

## Next Steps

Once processing has started:

1. **Monitor the progress** - See [Monitoring the Processing](monitoring-the-processing.md)
2. **Wait for completion** - Processing runs automatically
3. **Review results** - See [Finishing the Processing](finishing-the-processing.md)

For information about what to do during processing, see [Monitoring the Processing](monitoring-the-processing.md).
