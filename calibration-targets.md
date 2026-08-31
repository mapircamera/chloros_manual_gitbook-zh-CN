---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# 校准靶板

MAPIR提供多种校准靶板，以满足各类应用需求。下图所示的紧凑型T4-R50包含4块面板，其光反射率已在250至2,500 nm波长范围内经过测定。

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 漫反射参考靶片的反射率曲线如下，[点击此处下载数据](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157)：

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 400-1000nm</p></figcaption></figure>T4P 漫反射参考靶的反射率曲线如下，[数据下载请点击此处](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157)：

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 400-1000nm</p></figcaption></figure>观察反射率图表可知，图中数据以波长（x 轴）为横轴，反射率百分比（y 轴）为纵轴。当我们拍摄校准靶的图像时，便会在相机每个传感器波段所敏感的光谱范围内，建立像素值与反射率百分比之间的对应关系。

这意味着，使用我们的相机拍摄的每一张图像，您都可以利用我们的反射率标靶照片（例如 [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) 或 [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)，对图像进行反射率校准。校准完成后，图像中的每个像素值即等同于反射率百分比。

对于 **Survey3** 的输出结果，如果您将 Chloros 中的校准图像以常见的 JPG 或 TIFF 格式导出，则反射率百分比可通过将像素值除以图像格式的位深度来计算。 因此，对于 JPG 格式需除以 255，对于 TIFF 格式则需除以 65,535。 您也可以选择 Chloros 中的 PERCENT 格式输出，此时每个像素的百分比值范围为 0.0 到 1.0（反射率为 0% 到 100%）。 请注意，某些图像应用程序无法处理百分比（浮点）格式的图像，且此类文件在存储方面占用空间较大。

{% hint style="info" %}
**LATTICE 反射率采用不同的像素比例。** LATTICE 反射率的存储方式为：DN 32768 = 100% 反射率（而非 65535），且每个文件都包含一个 XMP `Chloros:PixelScale` 标签，其中注明了其比例。 请读取该标签并按其数值进行除法运算，而非假设一个常数——参见[输出图像格式](output-image-formats.md)。
{% endhint %}

## LATTICE 相机的校准靶

使用 LATTICE 相机时，反射率校准靶是**可选**的：Chloros 可以改用 DAQ 光传感器测得的下行辐照度作为反射率的参考（ρ = π·L/E）。该参考值通过“反射率源”设置进行选择 （GUI 中的“项目设置”；`--reflectance-source` 位于 CLI 中；`reflectance_source` 位于 SDK 中）：

| 值 | 行为 |
| --- | --- |
| `auto` *(默认)* | 通过质量控制（QA）的帧内目标为 **绝对参考**；当无目标存在或质量控制失败时，Chloros 将回退至数据采集（DAQ）下行分界值。 |
| `target` | 仅严格使用目标——不进行DAQ替换。 |
| `daq` | DAQ权威模式——下行测量始终作为参考。 |

LATTICE 的额外目标行为：

* **目标几何形状** — 支持带有 ArUco 标记的面板、固定 ROI 面板和条状目标；几何形状来自项目的目标配置。
* **按目标单元测量的目标数据** — `--target-reflectance-dir DIR` 指向包含按目标单元测量的反射率扫描数据的目录（`<serial>.csv`，通过目标单元的序列号/QR进行查找）。 若未检测到目标，Chloros 将回退至标称的 T3/T4P 光谱。
* **时间锚定** — 检测到的目标会校准其周围的帧，并在两次目标观测之间保持该校准状态。

完整的标志语义和示例详见 [CLI 参考文档](reference/cli-reference.md)（参见“按产品导出开关”）。

### F988

“F988 的反射率使用场景内的反射率面板进行校准：该波段超出了 DAQ 光传感器的校准范围，因此 Chloros 会应用您最近一次捕获的面板数据，并在两次面板观测之间保持该值。”

如果 F988 在仅使用 DAQ 校准的情况下运行，Chloros 将拒绝该波段基于 DAQ 的反射率并说明原因（跳过原因 `dls-uncalibrated-band-988`）；支持的处理方式是使用反射板工作流。

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
