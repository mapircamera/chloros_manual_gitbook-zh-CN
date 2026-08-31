# 反射率工作流程

DAQ 光传感器将辐射度图像转换为反射率。主要有两种不同的工作流程：

1. **单传感器** —— 一个 DAQ 测量下行辐照度，同时相机进行拍摄，随后通过Chloros

将相机测得的辐射度除以该参考值。
2. **双传感器** —— 两个 DAQ 传感器，一个监测天空，另一个监测目标，无需相机即可生成实时光谱反射率曲线。

## 单传感器 + 相机（下行参考）

DAQ 充当下行光传感器 (DLS)： 相机测量上行辐射通量密度 **L**（W/m²/sr/nm），DAQ 测量下行辐照度**E**（W/m²/nm），Chloros

按以下公式计算各波段的反射率：

> ρ = π · L / E

DAQ的读数始终与**曝光时间戳匹配**——这就是为什么DAQ和相机共享一个受PTP协议约束的时钟（参见[DAQ-E 网络与时间同步](ethernet-ptp.md)）。 进行户外工作时，请佩戴 Sunshine 余弦遮光帽并正确声明；遮光帽的声明会直接影响 E 的数值（参见 [遮光帽轮廓与校准范围](caps-and-range.md)）。 进行定量工作时，请牢记仪器特性：定量辐照度需基于至少 15 秒的读数平均值。

### 实时采集

在“相机”选项卡中将数据采集器（DAQ）与相机绑定：每台相机的设置面板中都有一个**光传感器**下拉菜单，其中列出了“光传感器”选项卡中所有已连接的数据采集器（DAQ-U/M/E）；对于同步阵列，全阵列范围内的光传感器选择会传播到每个成员 （单个相机仍可覆盖该设置）。绑定后，传感器的光谱数据将输入相机的 DLS 插槽，而导出的反射率值将除以匹配的读数。



<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

有两点行为值得注意：

* **未绑定DAQ → 拒绝反射率数据，而非伪造数据。**Chloros

会拒绝反射率数据，并记录跳过原因，而非默默返回质量较低的数据。
* **实际使用的读数会被保留。** 对于每个反射率帧，实际应用的 DAQ 读数会被作为 `.daq` 旁路文件写入图像旁边，以便日后重新处理该捕获数据（[录制与 .daq 格式](recording.md)）。

### 处理记录的影像

对于飞行后处理，请在会话期间记录一个 `.daq` 文件并将其与影像一同保存——处理管道会自动解析时间戳匹配的下行波，并在首次使用时从MAPIR

的云端获取任何缺失的工厂校准数据。 GUI录制文件在停止后会自动添加到当前打开的项目中。

反射率参考可在处理时选择——`--reflectance-source` 位于 `chloros-cli process` 上，或通过 GUI 的“项目设置”中的“反射率源”设置选择：

| 值 | 行为 |
| --- | --- |
| `auto`（默认） | 通过质量保证（QA）检测的帧内校准目标作为绝对参考；DAQ下行辐射（ρ = π·L/E）作为备用 |
| `daq` | 以DAQ数据为准 |
| `target` | 严格要求帧内目标；不采用DAQ替代 |

有关目标工作流，请参阅 [校准目标](../calibration-targets.md)；有关完整处理流程，请参阅 [LATTICE 章节](../lattice/README.md) 以及 [CLI

参考文档](../reference/cli-reference.md) 了解完整的处理流程。读取导出的反射率像素时，请使用标注的标度（LATTICE：32768 = ρ 1.0，XMP `Chloros:PixelScale`；Survey3

： 65535）——详见 [输出图像格式](../output-image-formats.md)。

### 超出数据采集卡（DAQ）校准范围的波段

数据采集卡（DAQ）的辐射校准范围约为 374–974 nm。Chloros

会拒绝任何在该范围内光谱权重不足一半的相机波段基于 DAQ 的反射率数据，并报告跳过原因 `dls-uncalibrated-band-<nm>`。 在已上市的SKU中，这仅影响F988：F988的反射率是使用场景内反射率面板进行校准的；由于该波段超出了DAQ光传感器的校准范围，因此Chloros

会采用您最近一次的面板捕获数据，并在两次面板观测之间维持该值。 如果 F988 相机仅以 DAQ 模式运行，Chloros

将拒绝该波段基于 DAQ 的反射率数据，跳过原因代码为 `dls-uncalibrated-band-988`——此时应采用反射板工作流。

## 双传感器（环境光 + 物体）

任意两组数据采集传感器（无论通过何种传输方式）均可提供无需摄像头的实时反射率光谱：一个传感器朝向天空（**环境光源**），另一个朝向被测对象（**对象扫描仪**），Chloros

将按波长计算：

> R(λ) = 对象(λ) / 环境(λ)

（当环境光 ≤ 0 时，结果为零）。

### 在图形用户界面中

在“光传感器”选项卡中连接两个传感器后，打开添加传感器覆盖层（网格视图中图表磁贴上的“+”按钮），并选择 **组合环境光 + 物体**。 在“环境光源”和“物体扫描仪”下拉菜单中选择两个传感器，然后点击“创建”。该组将作为独立图表显示，并在侧边栏中以带有绿色**REF** 标记的行形式呈现。



<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

在反射率图表（列表视图）下方，一个实时**植被指数表**会利用蓝光 450 / 绿光 550 / 红光 670 /NIR

800 nm 的波段中心，根据曲线计算出各项指数。基于比值的指数（NDVI

、GNDVI

、ENDVI、WDRVI

、GRVI

、CVI

、GCI

、 MSR）始终显示；需要绝对反射率的指标（EVI

、SAVI

、OSAVI

、GSAVI、GOSAVI、MSAVI2、RDVI、TDVI、LAI

、NLI、MNLI、FCI、GEMI

）仅在两个传感器均为功率校准模型时才会显示。

### 表观反射率与相对反射率——标注规则

Chloros

根据传感器对实际可提供的数据对双传感器输出进行标注：

| 传感器对 | 标签 |
| --- | --- |
| 两个传感器均已校准 — 已加载出厂校准包 | **表观反射率** |
| 任一传感器未校准 | **相对反射率** |

这三种模型均为辐射计量型：一旦加载了传感器的出厂校准包，其光谱即为绝对值 W/m²/nm，因此一对已校准传感器的比值将对应一个绝对的表观反射率——该值不由传输模型决定。 若某传感器仍在传输原始计数数据（无法加载校准包），则结果将降级为相对曲线（光谱形状仍然有效）。两个传感器都应带有正确声明的上限值（[上限配置文件与校准范围](caps-and-range.md)）。

### 摘自Python



在汇总的SDK

界面中没有专用的双传感器调用：请使用 `chloros_sdk.connect_daq_sensor()` 打开两个会话，并自行将它们的 `latest()` 光谱进行比对，同时采用相同的标注规范。 （MAPIR

的内部直接硬件界面上也存在一个双传感器记录工具，为完整起见，该工具已列于[CLI

参考文档](../reference/cli-reference.md)中——但它不属于随CLI

发布的组件；上述GUI工作流才是受支持的实时操作路径。）
