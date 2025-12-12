# Monitoring the Processing

Once processing has started, Chloros provides several ways to monitor progress, check for issues, and understand what's happening with your dataset. This page explains how to track your processing and interpret the information Chloros provides.

## Progress Bar Overview

The progress bar in the top header shows real-time processing status and completion percentage.

### Free Mode Progress Bar

For users without Chloros+ license:

**2-Stage Progress Display:**

1. **Target Detect** - Finding calibration targets in images
2. **Processing** - Applying corrections and exporting

**Progress bar shows:**

* Overall completion percentage (0-100%)
* Current stage name
* Simple horizontal bar visualization

### Chloros+ Progress Bar

For users with Chloros+ license:

**4-Stage Progress Display:**

1. **Detecting** - Finding calibration targets
2. **Analyzing** - Examining images and preparing pipeline
3. **Calibrating** - Applying vignette and reflectance corrections
4. **Exporting** - Saving processed files

**Interactive Features:**

* **Hover over** progress bar to see expanded 4-stage panel
* **Click** progress bar to freeze/pin the expanded panel
* **Click again** to unfreeze and auto-hide on mouse leave
* Each stage shows individual progress (0-100%)

***

## Understanding Each Processing Stage

### Stage 1: Detecting (Target Detection)

**What's happening:**

* Chloros scans images marked with Target checkbox
* Computer vision algorithms identify the 4 calibration panels
* Reflectance values extracted from each panel
* Target timestamps recorded for proper calibration scheduling

**Duration:**

* With marked targets: 10-60 seconds
* Without marked targets: 5-30+ minutes (scans all images)

**Progress indicator:**

* Detecting: 0% → 100%
* Number of images scanned
* Targets found count

**What to watch for:**

* Should complete quickly if targets properly marked
* If taking too long, targets may not be marked
* Check Debug Log for "Target found" messages

### Stage 2: Analyzing

**What's happening:**

* Reading image EXIF metadata (timestamps, exposure settings)
* Determining calibration strategy based on target timestamps
* Organizing image processing queue
* Preparing parallel processing workers (Chloros+ only)

**Duration:** 5-30 seconds

**Progress indicator:**

* Analyzing: 0% → 100%
* Fast stage, usually completes quickly

**What to watch for:**

* Should progress steadily without pauses
* Warnings about missing metadata will appear in Debug Log

### Stage 3: Calibrating

**What's happening:**

* **Debayering**: Converting RAW Bayer pattern to 3 channels
* **Vignette correction**: Removing lens edge darkening
* **Reflectance calibration**: Normalizing with target values
* **Index calculation**: Computing multispectral indices
* Processing each image through the full pipeline

**Duration:** Majority of total processing time (60-80%)

**Progress indicator:**

* Calibrating: 0% → 100%
* Current image being processed
* Images completed / Total images

**Processing behavior:**

* **Free mode**: Processes one image at a time sequentially
* **Chloros+ mode**: Processes up to 16 images simultaneously
* **GPU acceleration**: Significantly speeds up this stage

**What to watch for:**

* Steady progress through image count
* Check Debug Log for per-image completion messages
* Warnings about image quality or calibration issues

### Stage 4: Exporting

**What's happening:**

* Writing calibrated images to disk in selected format
* Exporting multispectral index images with LUT colors
* Creating camera model subfolders
* Preserving original filenames with appropriate suffixes

**Duration:** 10-20% of total processing time

**Progress indicator:**

* Exporting: 0% → 100%
* Files being written
* Export format and destination

**What to watch for:**

* Disk space warnings
* File write errors
* Completion of all configured outputs

***

## Debug Log Tab

The Debug Log provides detailed information about processing progress and any issues encountered.

### Accessing the Debug Log

1. Click the **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> icon in the left sidebar
2. Log panel opens showing real-time processing messages
3. Auto-scrolls to show latest messages

### Understanding Log Messages

#### Information Messages (White/Gray)

Normal processing updates:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Warning Messages (Yellow)

Non-critical issues that don't stop processing:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Action:** Review warnings after processing, but don't interrupt

#### Error Messages (Red)

Critical issues that may cause processing to fail:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Action:** Stop processing, resolve error, restart

### Common Log Messages

| Message                          | Meaning                                | Action Needed                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| "Target detected in \[filename]" | Calibration target found successfully  | None - normal                                         |
| "Processing image X of Y"        | Current progress update                | None - normal                                         |
| "No targets found"               | No calibration targets detected        | Mark target images or disable reflectance calibration |
| "Insufficient disk space"        | Not enough storage for output          | Free up disk space                                    |
| "Skipping corrupted file"        | Image file is damaged                  | Re-copy file from SD card                             |
| "PPK data applied"               | GPS corrections from .daq file applied | None - normal                                         |

### Copying Log Data

To copy log for troubleshooting or support:

1. Open Debug Log panel
2. Click **"Copy Log"** button (or right-click → Select All)
3. Paste into text file or email
4. Send to MAPIR support if needed

***

## System Resource Monitoring

### CPU Usage

**Free Mode:**

* 1 CPU core at \~100%
* Other cores idle or available
* System remains responsive

**Chloros+ Parallel Mode:**

* Multiple cores at 80-100% (up to 16 cores)
* High overall CPU utilization
* System may feel less responsive

**To monitor:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Performance tab → CPU section
* Look for "Chloros" or "chloros-backend" processes

### Memory (RAM) Usage

**Typical usage:**

* Small projects (< 100 images): 2-4 GB
* Medium projects (100-500 images): 4-8 GB
* Large projects (500+ images): 8-16 GB
* Chloros+ parallel mode uses more RAM

**If memory is low:**

* Process smaller batches
* Close other applications
* Upgrade RAM if regularly processing large datasets

### GPU Usage (Chloros+ with CUDA)

When GPU acceleration is enabled:

* NVIDIA GPU shows high utilization (60-90%)
* VRAM usage increases (requires 4GB+ VRAM)
* Calibrating stage is significantly faster

**To monitor:**

* NVIDIA System Tray icon
* Task Manager → Performance → GPU
* GPU-Z or similar monitoring tool

### Disk I/O

**What to expect:**

* High disk read during Analyzing stage
* High disk write during Exporting stage
* SSD significantly faster than HDD

**Performance tip:**

* Use SSD for project folder when possible
* Avoid network drives for large datasets
* Ensure disk isn't near capacity (affects write speed)

***

## Detecting Problems During Processing

### Warning Signs

**Progress stalls (no change for 5+ minutes):**

* Check Debug Log for errors
* Verify disk space available
* Check Task Manager to ensure Chloros is running

**Error messages appear frequently:**

* Stop processing and review errors
* Common causes: disk space, corrupted files, memory issues
* See Troubleshooting section below

**System becomes unresponsive:**

* Chloros+ parallel mode using too many resources
* Consider reducing concurrent tasks or upgrading hardware
* Free mode is less resource-intensive

### When to Stop Processing

Stop processing if you see:

* ❌ "Disk full" or "Cannot write file" errors
* ❌ Repeated image file corruption errors
* ❌ System completely frozen (not responding)
* ❌ Realized wrong settings were configured
* ❌ Wrong images imported

**How to stop:**

1. Click **Stop/Cancel button** (replaces Start button)
2. Processing halts, progress is lost
3. Fix issues and restart from beginning

***

## Troubleshooting During Processing

### Processing is Very Slow

**Possible causes:**

* Unmarked target images (scanning all images)
* HDD instead of SSD storage
* Insufficient system resources
* Many indices configured
* Network drive access

**Solutions:**

1. If just started and in Detecting stage: Cancel, mark targets, restart
2. For future: Use SSD, reduce indices, upgrade hardware
3. Consider CLI for batch processing large datasets

### "Disk Space" Warnings

**Solutions:**

1. Free up disk space immediately
2. Move project to drive with more space
3. Reduce number of indices to export
4. Use JPG format instead of TIFF (smaller files)

### Frequent "Corrupted File" Messages

**Solutions:**

1. Re-copy images from SD card to ensure integrity
2. Test SD card for errors
3. Remove corrupted files from project
4. Continue processing remaining images

### System Overheating / Throttling

**Solutions:**

1. Ensure adequate ventilation
2. Clean dust from computer vents
3. Reduce processing load (use Free mode instead of Chloros+)
4. Process during cooler times of day

***

## Processing Complete Notification

When processing finishes:

* Progress bar reaches 100%
* **"Processing Complete"** message appears in Debug Log
* Start button becomes enabled again
* All output files are in camera model subfolder

***

## Next Steps

Once processing completes:

1. **Review results** - See [Finishing the Processing](finishing-the-processing.md)
2. **Check output folder** - Verify all files exported correctly
3. **Review Debug Log** - Check for any warnings or errors
4. **Preview processed images** - Use Image Viewer or external software

For information about reviewing and using your processed results, see [Finishing the Processing](finishing-the-processing.md).
