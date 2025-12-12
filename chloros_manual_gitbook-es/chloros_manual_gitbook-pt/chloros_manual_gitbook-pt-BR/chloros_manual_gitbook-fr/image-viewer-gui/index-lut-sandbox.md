# Index/LUT Sandbox

The Index/LUT Sandbox is an interactive workspace within the Chloros Image Viewer that allows you to experiment with multispectral index calculations and color visualizations in real-time. This powerful tool helps you test different indices, refine value ranges, and create publication-ready visualizations without reprocessing your entire dataset.

## What is the Index/LUT Sandbox?

### Purpose

The Sandbox provides:

* **Real-time index calculation** - Apply any vegetation index instantly
* **Interactive LUT adjustment** - Fine-tune color gradients and ranges
* **Workflow optimization** - Determine best settings before batch processing

### Sandbox vs. Project Processing

**Index/LUT Sandbox (Interactive):**

* Single image at a time
* Instant feedback
* Experimental and iterative
* No permanent changes to files
* Perfect for exploring and testing

**Project Processing (Batch):**

* Entire dataset at once
* Pre-configured settings
* Permanent output files
* Time-intensive
* Best when settings are finalized

{% hint style="success" %}
**Best Workflow**: Use the Sandbox to experiment and find optimal index and LUT settings, then apply those settings during Project Processing for your entire dataset.
{% endhint %}

***

## Working with the Index/LUT Sandbox

### Understanding Pre-Calculated Indices

In Chloros, indices can be applied during project processing. To determine which index and LUT settings you want to apply to exports it is easiest to use the image viewer sandbox.

The sandbox allows you to:

* **Apply new index and color gradients (LUTs)** to visualize the data
* **Adjust visualization settings** interactively
* **View** already-calculated index images
* **Inspect** pixel values at all zoom levels

### Opening the Sandbox

The Index/LUT Sandbox is accessed in the **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebar tab:

1. Click an image in the file browser image grid, it opens in the **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab
2. Click **the Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab to open the left pop-out sidebar if it's not already open

### Selecting an Image to Apply an Index/LUT to

To work with an index in the Image Viewer <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandbox:

1. **Open an image** from the main image grid by clicking on it
2. The **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab will then open
3. Click the **Layer dropdown** (top-right of viewer)
4. Select the layer from the dropdown:
   * RAW (Reflectance)

### Applying an Index to an Image

Once the image is fullscreen and the **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab sidebar is open:

1. Check the Index box at the top of the sidebar
2. Choose your camera's filter from the left dropdown
3. Choose the desired index formula from the right dropdown
4. Drag the filter channel color circles to the locations in the index formula below
5. Once the formula is valid the image will update and show the index values
6. Move your mouse cursor around to see the values at the cursor's location
7. Zoom in to see individual pixels and their associated values

Each index has a specific value range and meaning:

#### NDVI Example

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

For complete index formula documentation, see [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

***

## Working with LUTs (Look-Up Tables)

### What is a LUT?

A **Look-Up Table (LUT)** maps numerical index values to colors for visualization:

* **Input**: Index pixel value (e.g., NDVI 0.65)
* **Output**: RGB color (e.g., bright green)
* **Purpose**: Make patterns easier to see and interpret

**Grayscale vs. Color LUT:**

* Grayscale: Scientific and neutral, shows raw data
* Color LUT: Intuitive and impactful, highlights patterns and differences

{% hint style="success" %}
**Visualization Power**: Applying a color LUT to a grayscale index image makes it dramatically easier to identify patterns, anomalies, and areas of interest at a glance.
{% endhint %}

### Applying a LUT to an Index Image

Once you have an index image showing

1. Click the <img src="../.gitbook/assets/image.png" alt="" data-size="line"> "+Add LUT" button
2. Select the color gradient
3. Adjust the clipping min/max end points
4. Adjust the Clipping Mode
5. Check the Index box in the **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab sidebar to apply the LUT

### Choosing a Color Gradient

**Selecting a gradient:**

1. In the LUT panel, locate the **colored gradient bar**
2. Hover your mouse over it to view available gradient presets
3. Select desired gradient
4. The image **updates immediately** with new colors when the Index box is checked

{% hint style="success" %}
**Best Practice**: For vegetation indices like NDVI, the Red-Yellow-Green gradient is most intuitive because it aligns with natural color associations (green=healthy, yellow=moderate, red=stressed).
{% endhint %}

### Adjusting Color Classes

The **Classes control** determines how many discrete color steps appear in your gradient:

**Class count options:**

* **2-5 classes**: Very broad categories, distinct zones
* **6-10 classes**: Balanced, good for classification
* **11-20 classes**: Smooth gradients, continuous appearance
* **20+ classes**: Near-continuous, maximum smoothness

**How to adjust:**

1. In the LUT panel, locate the **color swatch squares below the gradient bar**
2. Adjust the number of classes by adding with the + button
3. Remove the number of classes by double clicking on a color swatch
4. The gradient updates **in real-time** on the image

**Effect on visualization:**

* **Fewer classes** (3-5): Creates distinct zones, simplified classification, easier to distinguish categories
* **Medium classes** (6-10): Balanced approach, good for most applications
* **More classes** (15-20): Smooth transitions, detailed variation, photographic appearance

**When to use:**

* **Few classes (3-5)**: Presentation slides, classification maps, simple reports
* **Medium classes (6-10)**: General analysis, balanced detail, standard reports
* **Many classes (15-20)**: Scientific analysis, detailed inspection, publication-quality outputs

### Fine-Tuning Value Ranges

The **value range controls** determine which index values map to which colors in your gradient:

**Range controls in LUT panel:**

* **Minimum value**: Lower bound of the color scale
* **Maximum value**: Upper bound of the color scale
* **Intermediate values**: Automatically distributed between min and max (based on class count)

#### Adjusting Min/Max Values

**To adjust value ranges:**

1. In the LUT panel, locate the **Min Value** and **Max Value** input fields
2. Click the **Min Value** field
3. Type the desired minimum value (e.g., `0.2`)
4. Press **Enter** or click outside the field
5. Repeat for **Max Value** field (e.g., `0.9`)
6. The visualization **updates immediately**

{% hint style="info" %}
**Auto-Scaling**: When you first apply a LUT, Chloros automatically sets the min/max to the actual data range in the image. You can then narrow this range to focus on specific value ranges of interest.
{% endhint %}

**Example NDVI range adjustments:**

* **Full range**: `-1.0` to `1.0` (show all possible values)
* **Vegetation-focused**: `0.2` to `0.9` (exclude bare soil and water)
* **Healthy vegetation only**: `0.5` to `0.9` (highlight only vigorous plants)
* **Stress detection**: `0.2` to `0.5` (emphasize problem areas)
* **Custom range**: Adjust based on your observed pixel values

**Why adjust ranges?**

* **Increase contrast** in your area of interest
* **Exclude irrelevant values** (e.g., water bodies, bare soil)
* **Standardize visualization** across multiple images or dates
* **Emphasize subtle differences** within a narrow value range

### Clipping Out-of-Range Values

When pixel values fall outside your defined min/max range, you can control how they're displayed using **clipping modes**.

#### **Available clipping mode options:**

#### 1. Minimum and Maximum

* Pixels **below minimum** → display using the **first color** in gradient (e.g., red)
* Pixels **above maximum** → display using the **last color** in gradient (e.g., green)
* **Use case**: Emphasize extremes, show full data range with saturated colors at limits
* **Example**: NDVI values below 0.2 all appear red, values above 0.9 all appear green

#### 2. Transparent Background

* Pixels **outside the range** become **fully transparent**
* Only pixels **within range** show color gradient
* **Use case**: GIS overlay, isolating specific value ranges, highlighting only areas of interest
* **Example**: Show only NDVI 0.4-0.7 in color, everything else transparent

{% hint style="warning" %}
**Transparency Limitation**: Transparent pixels will appear as the background color in the viewer. When exported during processing, transparency is preserved in PNG format but not in JPG.
{% endhint %}

#### 3. Index Background

* Pixels **outside range** display in **grayscale** (showing raw index values)
* Pixels **within range** show **color gradient**
* **Use case**: Subtle highlighting, maintain context while emphasizing areas of interest
* **Example**: Color-highlight stressed vegetation (NDVI 0.3-0.5) while showing healthy areas in gray

#### 4. Original Background

* Pixels **outside range** display the **original multispectral image**
* Pixels **within range** show **color gradient**
* **Use case**: Most intuitive - combines natural image context with analytical color overlay
* **Example**: See the actual field/crop appearance with color-coded stress areas overlaid

### Choosing the Right Clipping Mode

| Clipping Mode              | Best For                                   | Visualization Style          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum and Maximum**    | Full data display, scientific analysis     | All pixels colored           |
| **Transparent Background** | GIS overlays, isolating specific ranges    | Color on range, blank beyond |
| **Index Background**       | Subtle emphasis, maintaining data context  | Color on range, gray beyond  |
| **Original Background**    | Reports, presentations, intuitive analysis | Color on range, photo beyond |

### Creating Custom LUT Colors

For full control over your visualization, you can create **custom color gradients** by editing individual color stops.

**To create a custom gradient:**

1. In the LUT panel, locate the **gradient preview bar**
2. Look for **color swatch squares** below the gradient
3. **Click a color stop** to select it
4. A **color picker** opens
5. Choose a new color using:
   * **Color wheel**: Visual color selection
   * **RGB/HSV sliders**: Precise color control
   * **Hex code entry**: Exact color specification (e.g., `#FF0000` for red)
6. Click off the color picker **to apply the new color**
7. The gradient **updates immediately** on the image

**Adding or removing color stops:**

* **Add a stop**: Click the + icon to add a new swatch at the end
* **Remove a stop**: Double click the color square to remove the swatch

**Customization strategies:**

* **Invert gradient**: Flip color order to reverse the meaning (e.g., green=low, red=high)
* **Brand colors**: Match your organization's color palette for reports
* **Colorblind-friendly**: Use orange-blue or purple-yellow combinations
* **Print optimization**: Choose colors that work in both color and grayscale printing
* **Multi-threshold**: Use distinct colors at specific value thresholds for classification

{% hint style="info" %}
**Saving Custom Gradients**: Custom gradients can be saved and reused. Click the save icon in the LUT panel to preserve your custom color schemes for future use.
{% endhint %}

***

## Interactive Workflow

### Real-Time Updates

All LUT adjustments in the sandbox update the image **instantly and interactively**:

* **Switch layer** → Image changes immediately
* **Select gradient** → Colors update instantly
* **Adjust value range** → Contrast changes in real-time
* **Change classes** → Gradient smoothness updates immediately
* **Modify clipping** → Background display changes instantly
* **Edit colors** → Custom gradient applies immediately

**No "Apply" button needed** - all changes are live and interactive!

{% hint style="success" %}
**Live Feedback**: The instant visual feedback allows you to rapidly experiment with different settings until you find the optimal visualization for your analysis needs.
{% endhint %}

### Iterative Refinement Workflow

**Typical LUT optimization workflow:**

1. **Select index layer** (e.g., RAW (Reflectance))
2. **Apply index** - Choose camera filter and index formula, drag colored circles to appropriate location in the index formula
3. **Apply LUT gradient** - Start with Red-Yellow-Green preset
4. **Inspect pixel values** - Move cursor around, note value ranges
5. **Adjust min/max** - Narrow to focus on vegetation (e.g., 0.2 to 0.9)
6. **Choose clipping** - Try "Original Background" for context
7. **Refine colors** - Customize gradient if needed for specific emphasis
8. **Finalize settings** - Document settings and copy to Project Settings for export processing

### Pixel Value Inspection

Understanding actual pixel values is crucial for setting effective LUT ranges:

**How to inspect values:**

1. Pixel values show when the image has either the Index, or both the Index and LUT **boxes checked**.
2. **Move your cursor** over different areas of the image
3. **Observe pixel values** displayed in the legend as you hover
4. Zoom in to see individual pixels highlighted with a floating value
5. **Take notes** of value ranges for different features:
   * **Healthy vegetation**: e.g., NDVI 0.55-0.85
   * **Stressed vegetation**: e.g., NDVI 0.30-0.50
   * **Bare soil**: e.g., NDVI 0.05-0.25
   * **Water** (if present): e.g., NDVI -0.05 to 0.10

**Using pixel values to set LUT ranges:**

After inspecting pixel values, adjust your LUT min/max accordingly:

**Example scenario:**

* **Observation**: Soil values = 0.05-0.25, Stressed = 0.25-0.50, Healthy = 0.50-0.85
* **Goal**: Visualize only plant health (exclude soil)
* **LUT settings**: Min = `0.25`, Max = `0.85`
* **Clipping**: "Original Background" to see soil in natural color
* **Result**: Color gradient only applies to vegetation, soil shows as original image

{% hint style="info" %}
**Dynamic Range**: Different crops, seasons, and growth stages will have different value ranges. Always inspect pixel values in your specific dataset before setting LUT ranges.
{% endhint %}

***

## Custom Indices (Chloros+)

### Creating Custom Index Formulas

{% hint style="info" %}
**Where to Create**: Custom indices can be configured in **Project Settings** before processing, as well as in the Image Viewer sandbox sidebar.
{% endhint %}

**To create a custom index:**

1. **Open Project Settings** (before processing) or Image Viewer sandbox sidebar
2. Navigate to the **Index formula dropdown**
3. Look for **"Custom"** option (must be logged in with Chloros+ license)
4. **Define your formula** using band variables:
   * Band names: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operators: `+`, `-`, `*`, `/`, `^` (exponent)
   * Functions: `sqrt()`, `abs()`, etc. (if supported)
   * Parentheses: `()` for order of operations
5. **Name your index** (e.g., "MyIndex" or "CustomNDVI")
6. **Save the configuration**

**Example custom formulas:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Formula Validation**: Ensure your formula uses bands available in your camera. For example, RedEdge is only available on cameras with a RedEdge filter.
{% endhint %}

***

## Next Steps

Now that you understand the Index/LUT Sandbox:

* **Apply to processing**: Use discovered settings in [Project Settings](../project-settings/page-2.md)
* **Batch process**: Apply optimized indices to full datasets
* **Learn more**: Read [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md)

Related documentation:

* [**Image Layers**](image-layers.md) - Layer management and visualization
* [**Opening an Image Full Screen**](page-3.md) - Image Viewer basics
* [**Processing Images (GUI)**](../processing-images-gui/page-1.md) - Full processing workflow
