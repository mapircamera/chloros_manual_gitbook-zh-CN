---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Can I process images from cameras that are not MAPIR brand with Chloros?</summary>

No, Chloros only supports processing MAPIR camera images. Please see the list of [supported camera models](supported-cameras.md) for more information. We do offer processing of other cameras on MAPIR Cloud, see full list [here](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Can I calibrate my images for reflectance without a calibration target?</summary>

No. Without an image of the calibration target captured around when the non target images are captured you will not be able to relate the image's pixel values to a known reflectance percent. If you also do not include the log from a MAPIR light sensor then the ambient light spectrum will not be measured, and the reflectance results will not be accurate.

</details>

<details>

<summary>Can I edit my images prior to processing in Chloros?</summary>

No. Chloros assumes the input data has not been modified. Do not change the file names.

</details>

<details>

<summary>Can I set my MAPIR Survey3 cameras to auto exposure and process the images in Chloros?</summary>

No. Survey3 image datasets must have a fixed/locked exposure, so no auto shutter speed or auto ISO. All images of the same camera model must have identical shutter speed and ISO (exposure).

</details>

<details>

<summary>Can Chloros process or analyze orthomosaic images?</summary>

No. Only individual MAPIR camera images are supported, not stitched images like an orthomosaic map.

</details>

<details>

<summary>How can I speed up the target detection step of Chloros?</summary>

In the file browser table pre-selecting the target images in the right column will tell Chloros to only look in those images for calibration targets, greatly speeding up the processing.

</details>

<details>

<summary>If I will upload my images to <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> should I process in Chloros prior to uploading?</summary>

If you plan to upload to our online processing platform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) do not edit the images prior to uploading. Cloud will perform all the same processing and more.

</details>

<details>

<summary>Will MAPIR ever support X feature? I really wish MAPIR offered X.</summary>

We are always interested in receiving feedback on our products. If you find an issue with our products, or have a suggestion on how we can improve our products please [CONTACT US](https://www.mapir.camera/community/contact) to share your thoughts. Most of our R\&D is guided by listening to our customer's biggest needs.

</details>
