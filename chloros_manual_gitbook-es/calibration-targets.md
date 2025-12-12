---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Calibration Targets

MAPIR offers various calibration targets to cover a range of applications. The compact T4-R50 seen below contains 4 panels that have been measured for light reflectance from 250 - 2,500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

The T4 diffuse reference targets have the following reflectance curves, [data download here](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 400-1000nm</p></figcaption></figure>

Looking at the reflectance graph you can see that the values are wavelength (x-axis) versus reflectance percent (y-axis). When we capture an image of the calibration target we then create a relationship between pixel value and reflectance percent, within the spectrum that each of the camera's sensor bands are sensitive to.

This means that with every image you capture with our cameras, you can use a photo of our reflectance targets, such as the [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) or [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) to calibrate the images for reflectance. Once calibrated each pixel in the image is equal to percent reflectance.

If you output the calibrated images in Chloros as the typical JPG or TIFF then the reflectance percent is calculated by dividing the pixel value by the bit depth of the image format. So for JPG divide by 255, and for TIFF divide by 65,535. You can also choose the PERCENT format output in Chloros, and then each pixel will range from a percent value of 0.0 to 1.0 (0% to 100% reflectance). Just keep in mind that some image applications cannot accept the percent (floating point) images, and they are large in size storage wise.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
