# Adding Files to a Project

Once you've created or opened a project in Chloros, the next step is to add your multispectral images to begin processing. The File Browser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> tab makes it easy to import images and manage your dataset.

## Accessing the File Browser

1. Open or create a project in Chloros
2. Click the **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> icon in the left sidebar
3. The File Browser panel will display your project's file list

{% hint style="info" %}
**Supported File Types**: Chloros supports RAW+JPG and JPG image files from MAPIR Survey3W and Survey3N cameras. Only RAW+JPG are recommended.
{% endhint %}

***

## Adding Images to Your Project

There are two primary ways to add images to your project:

### Method 1: Add Files

Use this option to import individual image files or a small selection of files.

1. Click the **"Add Files"** button at the top of the File Browser panel
2. Navigate to the folder containing your images
3. Select one or more image files (hold **Ctrl** to select multiple files)
4. Click **"Open"** to import the selected files

### Method 2: Add Folder

Use this option to import all images from a folder at once.

1. Click the **"Add Folder"** button at the top of the File Browser panel
2. Navigate to and select the folder containing your capture session images
3. Click **"Select Folder"** to import all supported images from that folder

***

## Understanding the File Browser Table

Once images are imported, they appear in a table with the following columns:

### Thumbnail

* Small preview of each image
* Click thumbnail to view full image in the main preview area

### File Name

* Original filename from the camera
* Maintains camera naming convention (e.g., IMG\_0001.RAW)

### Timestamp

* Date and time the image was captured
* Extracted from image EXIF metadata
* Used for PPK synchronization and calibration target detection

### Camera Model

* Automatically detected camera and filter configuration
* Examples: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Used to apply correct processing profiles

### Target Column (Checkbox)

* Check this box for images that contain calibration targets
* Greatly speeds up target detection during processing
* See [Choosing Target Images](choosing-target-images.md) for details

***

## Managing Files in Your Project

### Removing Files

To remove unwanted images from your project:

1. Select one or more images in the File Browser table
2. Click the **"Remove Selected"** button
3. Confirm removal (files are not deleted from disk, only removed from the project)

### Sorting and Filtering

* **Sort by column**: Click any column header to sort images
* **Timestamp sort**: Useful for organizing chronological capture sequences
* **Camera model filter**: Group images by camera type if using multiple cameras

***

## Image Preview

### Viewing Full Image

Click any image thumbnail in the File Browser to display it in the main preview area:

1. Image appears in the center preview panel
2. Use zoom controls to inspect image details
3. Navigate between images using arrow keys

### Quick Navigation

* **Previous Image**: Click left arrow or press ← key
* **Next Image**: Click right arrow or press → key
* **Zoom In/Out**: Use mouse wheel or zoom buttons
* **Pan**: Click and drag on image when zoomed in

***

## Duplicate File Handling

Chloros automatically detects and ignores duplicate files:

* Files with identical filenames are skipped
* Prevents accidental double-processing
* Warning message displayed when duplicates are detected

{% hint style="warning" %}
**Important**: Do not rename or modify your original image files before importing. Chloros relies on original filenames and metadata for proper processing.
{% endhint %}

***

## Mixed Camera Datasets

If your project contains images from multiple MAPIR cameras:

1. Chloros automatically detects each camera model
2. Each camera type is processed with its appropriate calibration profile
3. File Browser displays camera model in the Camera Model column
4. Processing applies correct settings for each camera type

**Example scenario**: Survey3W RGN + Survey3N OCN dual-camera setup

***

## Best Practices

### Organize Before Import

* Keep calibration target images in the same folder as survey images
* Maintain original folder structure from your camera/SD card
* Don't mix datasets from different sessions in one project

### File Naming

* Preserve original camera filenames (IMG\_0001.RAW, etc.)
* Don't rename files before import
* Original names contain important metadata

### Calibration Target Images

* Always include 1-2 calibration target images per session
* Capture targets before and after the capture session
* Place targets in the same lighting conditions as capture area
* Mark target images using the Target checkbox to speed up processing

***

## Common Issues and Solutions

### Images Not Appearing After Import

**Possible causes:**

* File format not supported (only RAW+JPG and JPG from MAPIR cameras)
* Images are from non-MAPIR cameras (see [Supported Cameras](../supported-cameras.md))
* File corruption or incomplete transfer from SD card

**Solution**: Verify file format and camera model compatibility

### Camera Model Not Detected

**Possible causes:**

* Modified EXIF metadata
* Images edited in external software
* Incomplete file transfer

**Solution**: Re-import original, unmodified files from camera/SD card

### Missing Timestamps

**Possible causes:**

* Camera clock not set correctly
* EXIF data stripped by external software

**Solution**: Verify camera time settings were correct during capture

***

## Next Steps

Once your files are imported:

1. **Review the file list** - Ensure all images loaded correctly
2. **Check camera models** - Verify correct camera detection
3. **Mark target images** - See [Choosing Target Images](choosing-target-images.md)
4. **Adjust settings** - Configure processing options in [Project Settings](adjusting-project-settings.md)
5. **Start processing** - See [Starting the Processing](starting-the-processing.md)

For detailed information about project configuration, see [Adjusting Project Settings](adjusting-project-settings.md).
