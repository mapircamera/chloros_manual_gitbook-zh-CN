---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# 校准靶板

MAPIR 提供多种校准靶板，以满足各类应用需求。下图所示的紧凑型 T4-R50 包含 4 个面板，其光反射率已在 250 至 2,500 nm 波长范围内经过测量。

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 漫反射参考靶具有以下反射率曲线，[点击此处下载数据](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157)：

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 反射率 :: 400-1000nm</p></figcaption></figure>T4P 漫反射参考靶的反射率曲线如下，[数据下载请点击此处](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157)：

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P 反射率 :: 400-1000nm</p></figcaption></figure>观察反射率图表可知，图中显示的是波长（x 轴）与反射率百分比（y 轴）的关系。当我们拍摄校准靶的图像时，便会在相机每个传感器波段所敏感的光谱范围内，建立像素值与反射率百分比之间的对应关系。

这意味着，使用我们的相机拍摄的每一张图像，您都可以利用我们的反射率标靶照片（例如 [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) 或 [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125)，对图像进行反射率校准。校准完成后，图像中的每个像素值即等同于反射率百分比。

对于**Survey3**的输出，如果您在Chloros 中将校准后的图像导出为常见的JPG或TIFF 格式，则反射率百分比是通过将像素值除以图像格式的位深度来计算的。因此，对于JPG格式，需除以255；对于TIFF 格式，则需除以65,535。 您也可以在Chloros 中选择“PERCENT”格式进行输出，此时每个像素的百分比值范围为0.0至1.0（即0%至100%反射率）。请注意，某些图像应用程序无法处理百分比（浮点）格式的图像，且此类图像在存储方面占用空间较大。

{% hint style="info" %}
**LATTICE 反射率采用不同的像素比例。** LATTICE 反射率的存储方式为：DN 32768 = 100% 反射率（而非 65535），且每个文件都包含一个 XMP `Chloros:PixelScale` 标签，用于标明其比例。 请读取该标签并按其数值进行除法运算，而非假设一个常数——详见 [输出图像格式](output-image-formats.md)。
{% endhint %}

## 使用 LATTICE 相机时的校准靶

对于 LATTICE 相机，反射率校准靶是**可选的**：Chloros 也可以参考由 DAQ 光传感器测得的下行辐照度来计算反射率（ρ = π·L/E）。 该参考值由项目的反射率源设置决定，该设置通过 `chloros-cli process --reflectance-source` 进行配置 （图形用户界面（GUI）中没有相关控件，而是使用项目中存储的值，默认值为 `auto` —— 参见 [项目设置](project-settings/project-settings.md)）：

| 值 | 行为 |
| --- | --- |
| `auto` *(默认)* | 通过质量保证（QA）的帧内目标即为**绝对参考**；当不存在目标或QA失败时，Chloros 将回退到数据采集（DAQ）下行分频。 通过 QA 但带有标记（面板掉落或波段故障）的目标不会被自动优先选用：系统将按其在保留面板上测得的得分顺序，依次尝试目标、目标+DAQ 和仅 DAQ 三种链。 |
| `target` | 严格仅限目标——不进行 DAQ 替换。 |
| `daq` | DAQ 权威——下行测量始终作为参考。 |

LATTICE 的其他目标行为：

* **目标几何形状** — 支持 ArUco 标记面板、固定 ROI 面板和条形目标；几何形状来自项目的目标配置。
* **按单元测量的目标数据** — `--target-reflectance-dir DIR` 指向包含按单元测量目标反射率扫描数据的目录（`<serial>.csv`，通过目标单元的序列号/QR进行查找）。 若未检测到目标，Chloros 将回退到标称的 T3/T4P 光谱。
* **时间锚定** — 检测到的目标会校准其周围的帧，并在两次目标观测之间保持该校准状态。

完整的标志语义和示例请参见 [CLI 参考](reference/cli-reference.md)（参见“按产品导出开关”）。

### F988

“F988反射率使用场景内反射率面板进行校准：该波段超出了DAQ光传感器的校准范围，因此Chloros 会应用您最近的面板捕获数据，并在两次面板观测之间保持该值。”

如果 F988 在仅使用 DAQ 校准的情况下运行，Chloros 将拒绝该波段基于 DAQ 的反射率并说明原因（跳过原因 `dls-uncalibrated-band-988`）；支持的处理方式是使用反射板工作流。

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
