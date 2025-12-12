# Choosing Target Images

Marking which images contain calibration targets is a crucial step that significantly speeds up the Chloros processing pipeline. By pre-selecting target images, you eliminate the need for Chloros to scan every image in your dataset for calibration targets.

## Why Mark Target Images?

### Processing Speed

Without marking target images, Chloros must:

* Scan every single image in your project
* Run target detection algorithms on each image
* Check hundreds or thousands of images unnecessarily

**Result**: Processing can take significantly longer, especially for large datasets.

### With Marked Target Images

When you check the Target column for specific images:

* Chloros only scans the checked images for targets
* Target detection completes much faster
* Overall processing time is greatly reduced

{% hint style="success" %}
**Speed Improvement**: Marking 2-3 target images in a 500-image dataset can reduce target detection time from 30+ minutes to under 1 minute.
{% endhint %}

***

## How to Mark Target Images

### Step 1: Identify Your Target Images

Look through your imported images in the File Browser and identify which images contain calibration targets.

**Common scenarios:**

* **Pre-capture target**: Captured before starting the session
* **Post-capture target**: Captured after completing the session
* **In-field targets**: Targets placed within the capture area
* **Multiple targets**: 2-3 target images per session (recommended)

### Step 2: Check the Target Column

For each image containing a calibration target:

1. Locate the image in the File Browser table
2. Find the **Target** column (rightmost column)
3. Click the checkbox in the Target column for that image
4. Repeat for all images containing targets

### Step 3: Verify Your Selection

Before processing, double-check:

* [ ] All images with calibration targets are checked
* [ ] No non-target images are accidentally checked
* [ ] Targets are clearly visible in checked images

***

## Best Practices for Target Images

### Target Capture Guidelines

**Timing:**

* Capture target images immediately before and throughout your capture session
* Within the same lighting conditions as your DAQ light sensor
* Ideally capture target images as often as possible for the best results. Otherwise, the light sensor data will be used to adjust the calibration over time.

**Camera Position:**

* Hold camera above target such that is is centered and fills around 40-60% of the image center.
* Keep camera parallel/nadir to target surface

**Lighting:**

* Same ambient lighting as your DAQ light sensor
* Avoid shadows on the target surfaces
* Don't block your light source with your body, vehicle or vegetation
* Overcast conditions provide most consistent results

**Target Condition:**

* Keep target panels clean and dry
* All 4 panels should be clearly visible and unobstructed
* Targets perpendicular/nadir to the light source if possible

### How Many Target Images?

**Minimum:** 1 target image per session. **Recommended:** 3-5 target images per session.

**Best practice schedule:**

* 3-5 images captured shortly after the light sensor is recording
* Rotate the camera between captures for the best results
* Optional: periodically mid-session if lighting conditions change constantly

***

## Working with Multiple Cameras

### Dual-Camera Setups

If using two MAPIR cameras simultaneously (e.g., Survey3W RGN + Survey3N OCN):

1. Capture target images with **both cameras** at the same time
2. Use the **same physical target** for both cameras
3. Mark target images for **both camera types** in the File Browser
4. Chloros will use appropriate targets for each camera's calibration

### Camera Model Column

The **Camera Model** column helps identify which images came from which camera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Use this column to verify you've marked targets for each camera type in your project.

***

## Target Detection Settings

### Adjusting Detection Sensitivity

If Chloros isn't detecting your targets correctly, adjust these settings in [Project Settings](adjusting-project-settings.md):

**Minimum calibration sample area:**

* **Default**: 25 pixels
* **Increase** if getting false detections on small artifacts
* **Decrease** if targets aren't being detected

**Minimum target clustering:**

* **Default**: 60
* **Increase** if targets are being split into multiple detections
* **Decrease** if targets with color variation aren't fully detected

***

## Common Target Image Issues

### Problem: No Targets Detected

**Possible causes:**

* Target images not marked in File Browser
* Target too small in frame (< 30% of image)
* Poor lighting (shadows, glare)
* Target detection settings too strict

**Solutions:**

1. Verify Target column is checked for correct images
2. Review target image quality in preview
3. Recapture targets if quality is poor
4. Adjust target detection settings if needed

### Problem: False Target Detections

**Possible causes:**

* White buildings, vehicles, or ground cover mistaken for targets
* Bright patches in vegetation
* Detection sensitivity too low

**Solutions:**

1. Mark only actual target images to limit detection scope
2. Increase minimum calibration sample area
3. Increase minimum target clustering value
4. Ensure target images show only the target (minimal background clutter)

***

## Verification Checklist

Before starting processing, verify your target image selection:

* [ ] At least 1 target image marked per session
* [ ] Target column checkboxes are checked for all target images
* [ ] Target images captured within same timeframe as survey
* [ ] Targets clearly visible in preview when clicked
* [ ] All 4 calibration panels visible in each target image
* [ ] No shadows or obstructions on targets
* [ ] For dual-camera: Targets marked for both camera types

***

## Target-Free Processing

### Processing Without Calibration Targets

While not recommended for scientific work, you can process without targets:

1. Leave all Target column checkboxes unchecked
2. **Disable** "Reflectance calibration" in Project Settings
3. Vignette correction will still be applied
4. Output will not be calibrated for absolute reflectance

{% hint style="warning" %}
**Not Recommended**: Without reflectance calibration, pixel values represent relative brightness only, not scientific reflectance measurements. Use calibration targets for accurate, repeatable results.
{% endhint %}

***

## Next Steps

Once you've marked your target images:

1. **Review your settings** - See [Adjusting Project Settings](adjusting-project-settings.md)
2. **Start processing** - See [Starting the Processing](starting-the-processing.md)
3. **Monitor progress** - See [Monitoring the Processing](monitoring-the-processing.md)

For more information about calibration targets themselves, see [Calibration Targets](../calibration-targets.md).
