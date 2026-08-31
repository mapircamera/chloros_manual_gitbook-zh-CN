# 完成处理

当 Chloros 完成处理后，您需要查看结果、验证输出质量，并准备好处理后的图像以供工作流程使用。本页面将引导您完成最后几步操作及后续步骤。

## 处理完成指示

当处理成功完成时，您将看到以下几个指示：

* ✅ **进度条**：达到 100% 完成
* ✅ **调试日志**：显示带有计数（图像、相机组、目标、校准图像、写入文件）的最后几行 `[RUN-SUMMARY]` 日志
* ✅ **开始按钮**：重新启用 （准备进行下一次处理）
* ✅ **输出文件**：所有处理后的图像已保存到项目的输出目录树中（如下所示）

{% hint style="warning" %}
**未写入任何图像的运行视为失败。** 如果您请求了图像产品，但运行未写入任何图像，Chloros 会将其报告为失败——日志名称中的 `[RUN-SUMMARY]` 提示了可能的原因（未导入任何内容、未检测到目标，或所有请求的产品均因不适用而被跳过）。 等效的 CLI 会以非零状态退出。 刻意仅处理元数据的运行（所有导出产品关闭，无索引）仍被视为成功。参见 [CLI 参考](../reference/cli-reference.md#a-run-that-writes-no-images-fails)。
{% endhint %}

***

## 查找已处理的图像

### 打开输出文件夹

1. 点击 **主菜单** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> 图标（左上角）
2. 选择 **“打开项目文件夹”**

3. 文件资源管理器将打开并跳转至项目目录
4. 根据项目名称查找相应项目

### 输出文件树

处理后的图像文件将保存在**项目文件夹下，按相机分类，再按文件格式分组**：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **相机文件夹**：LATTICE 相机对应 `LATT-<sensor>-<lens>-F<filter>`（与拍摄文件的 EXIF 信息 `Model` 匹配）， `<model>_<filter>` 对应 Survey3（例如 `Survey3N_RGN`）。 两台共享同一传感器和滤光片但镜头不同的相机将分别保存为独立的文件树——因其暗角、视场和畸变存在差异。
* **格式文件夹**：遵循您的导出格式设置——`tiff16`、`tiff8`、`png8`、`jpg8`， 或 `tiff32` 对应 TIFF（32 位，百分比）。辐射度数据始终为 float32 格式，并始终存储在 `tiff32` 文件中。
* **产品文件夹**：
  * `Reflectance_Calibrated_Images/` — 校准反射率
  * `Debayered_Images/` — 线性去拜耳化（LATTICE）
  * `Preview_Images/` — 显示预览（LATTICE）
  * `Radiance_Images/` — float32 谱辐射通量，W/m²/sr/nm（LATTICE 多光谱）
  * `Vignette_Corrected_Images/` **或** `Sensor_Response_Images/` — 针对无反射率参考帧的未校准备用数据；每次运行中仅存在其中一种，具体由“暗角校正”设置决定
  * `<INDEX>_Index_Images/` — 每个选定索引对应一个文件夹（例如 `NDVI_Index_Images`）

{% hint style="info" %}
**每个导出产品都保留源文件的名称。**`capture_..._raw.tif` 的辐射度导出文件仍名为 `capture_..._raw.tif` —— 它只是位于 `tiff32/Radiance_Images/` 文件夹中。**文件夹用于标识产品，而非文件名**，因此搜索 `*radiance*.tif` 不会找到任何结果；请改用目录进行匹配。
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### 应该有多少个文件？

不要按公式计算——输出文件的数量取决于启用了哪些产品以及哪些产品适用于每台相机（例如 RGB 相机不生成辐射度/反射率数据）。权威的文件数量记录在日志中：最后一行 `[RUN-SUMMARY]` 会精确报告写入的文件数量，提示行则解释了被跳过的任何内容。

***

## 查看处理后的图像

### 在文件资源管理器中快速预览

**Windows 内置预览：**

1. 导航至产品文件夹（例如 `tiff16/Reflectance_Calibrated_Images/`）
2. 选择一个图像文件
3. 预览将在 Windows 资源管理器的预览窗格中显示
4. 使用方向键浏览图像

### 在外部图像查看器中预览

**推荐的查看器：*** **QGIS** - 免费的 GIS 软件 （最适合地理参考多光谱分析）
* **IrfanView** - 快速、轻量级的图像查看器（支持 TIFF）
* **Adobe Photoshop** - 专业编辑（支持 TIFF）
* **GIMP** - Photoshop的免费替代品
* **Windows Photos** - 基本浏览（可能不支持16位TIFF）

### 在 Chloros 图像查看器中预览

使用 Chloros 内置的图像查看器进行高级可视化：

1. 在文件浏览器中点击图像缩略图
2. 图像将在主预览区域中打开
3. 在左侧边栏中点击 **图像查看器** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 选项卡
4. 使用 [索引/LUT 沙盒](../image-viewer-gui/index-lut-sandbox.md) 进行交互式分析

详细说明请参阅 [图像查看器](../image-viewer-gui/opening-an-image-full-screen.md)。

***

## 读取反射率像素值（GIS / Pix4D / 脚本）

反射率以整数 DN 形式存储，且 **表示 ρ = 1.0 的 DN 值取决于源相机**：

| 源相机          | ρ = 1.0 对应值 | 如何判断                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768**（容许范围最高至 ρ 2.0） | 文件中带有 XMP 标签 `Chloros:PixelScale=32768` |
| Survey3         | **65535**（在 ρ 1.0 处被截断）     | 无 `Chloros:*` XMP 标签——这种缺失即为信号 |

**读取 `Chloros:PixelScale` 标签并以此除以**，而不是假设统一的 65535 —— 将 LATTICE 反射率除以 65535 会悄无声息地将每个值减半。 有一种特殊情况按设计不包含缩放信息：以 8 位输出格式写入的 8 位源捕获数据会被裁剪而非重新缩放，且故意不包含缩放标签——请以 16 位或 32 位格式重新导出，而非进行除法运算。 详情请参阅 [输出图像格式](../output-image-formats.md)。***

## 导出时保留的元数据

每个产品都会保留源捕获的 **GPS 块**及其**EXIF 子 IFD**，因此
导出文件会包含 `FocalLength`、`FNumber`、 `ExposureTime`、`ISO`、`DateTimeOriginal` 和
`CameraSerialNumber`，以及地理参考信息。

{% hint style="warning" %}
**如果正射镶嵌图的比例尺出现异常，请首先检查 `FocalLength`。**
Pix4D 根据焦距和飞行高度计算地面采样距离。 如果没有该标签，
系统将回退到一个严重错误的比例尺——在一项包含49张照片的飞行测量中，一片411米×160米的
橘园被重建为47.8公里×13公里，生成了一张4.55亿像素的正射影像，其中大部分
是空地。 拼接速度缓慢和文件大小出乎意料地庞大正是这一问题的症状，而非独立的
问题。

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

并非*所有*标签都被复制。IFD0的结构标签是被有意保留的（复制
这些标签会导致LATTICE输出损坏），而`ExifImageWidth` / `ExifImageHeight` 被排除在外，
因为它们描述的是原始捕获内容——否则，经过尺寸调整的导出文件将
申报与其自身栅格相矛盾的尺寸。

***

## 查看调试日志

### 检查警告或错误

1. 打开 **调试日志** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 选项卡
2. 滚动浏览消息
3. 查找黄色警告或红色错误
4. 阅读 `[RUN-SUMMARY]` 相关行及任何提示
5. 联系 MAPIR 支持团队寻求帮助

### 保存日志

若要保留处理记录或发送给 MAPIR 技术支持：

1. 点击 **“复制”**或**“下载”** 按钮
2. 将其作为文本文件保存到项目文件夹中
3. 随项目文档一并保存
4. 若遇到问题，请发送给 MAPIR 支持团队

***

## 常见输出问题及解决方案

### 问题：缺少输出文件

**可能原因：**

* 该产品不适用于该相机（例如，RGB 相机不支持辐射度/反射率——日志中会有相关提示）
* 缺少必需的参考数据（例如：反射率数据缺少目标且无 `.daq` 下行辐射）
* 在“项目设置”中，该产品的导出复选框被禁用
* 导出过程中磁盘空间不足

**解决方案：**

1. 检查调试日志中的 `[RUN-SUMMARY]` 提示和 `[EXPORT-CHECK]` 行——它们解释了按相机跳过的具体原因
2. 验证 [项目设置](adjusting-project-settings.md) 中的产品导出复选框
3. 确认磁盘空间是否充足
4. 排除原因后重新处理

### 问题：边缘过暗或过亮（暗角仍可见）

**可能原因：**

* 未启用暗角校正
* 相机/镜头未收录在 Chloros 配置文件数据库中
* 暗角程度过重，超出校正能力范围

**解决方案：**

1. 确认已在“项目设置”中启用暗角校正
2. 检查是否正确检测到相机型号
3. 若暗角问题仍未解决，请联系 MAPIR 技术支持

### 问题：颜色或数值不正确

**可能原因：**

* 未检测到校准目标
* 选错了校准目标模型
* 反射率校准已禁用
* 目标图像质量较差

**解决方案：**

1. 确认已启用反射率校准
2. 检查调试日志中的“已找到校准目标”消息
3. 检查校准目标图像质量
4. 标记正确的校准目标后重新处理

### 问题：NDVI 数值似乎有误

**预期 NDVI 范围：*** **水体、岩石、土壤**：-0.1 至 0.2
* **稀疏/不健康的植被**：0.2 至 0.4
* **中等植被**：0.4 至 0.6
* **健康、茂密的植被**：0.6 至 0.9**如果数值超出这些范围：**

1. 确认已应用反射率校准
2. 确认已包含光传感器日志
3. 检查是否检测到校准目标
4. 确保检测到了正确的相机型号
5. 审查目标图像的拍摄时间和条件
6. 若您自行根据反射率文件计算指数，请确认已除以文件中的 `Chloros:PixelScale`（参见上文）

***

## 使用处理后的图像

### 用于摄影测量/正射镶嵌制作

**推荐工作流程：**

1.**将校准后的反射率图像**导入摄影测量软件：
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **保留 EXIF 元数据**：确保保留 GPS 数据以进行地理标记
3. **校准工作流程**：使用反射率图像以确保科学精度——LATTICE 反射率图像包含 Pix4D 可读取的 XMP 校准标签
4. **处理索引镶嵌图**：从单个索引图像生成 NDVI 正射镶嵌图
5. **导出地理参考的 GeoTIFF**：用于 GIS 应用

### 用于 GIS 分析

**推荐工作流程：**

1.**导入 QGIS、ArcGIS 或类似软件**

2.**使用 16 位 TIFF** 反射率图像进行多波段分析（除以文件的 `Chloros:PixelScale`）
3. **使用指数图像**（NDVI、NDRE）作为即用型植被图层
4. **栅格计算器**：合并波段以进行自定义分析
5. **导出**：生成分类图、变化检测图、植被健康图

### 用于直接分析/报告

**推荐工作流程：**

1.**使用带 LUT 配色的指数图像** 生成可视化报告
2. **提取统计数据**：按田块/样地计算 NDVI 的平均值
3. **时间序列**：比较多个观测时段的指数值
4. **生成报告**：包含地图、统计数据和可视化内容***

## 归档与备份

### 推荐的备份策略

**需保存的内容：*** ✅ **原始 RAW/JPG 图像或 LATTICE 原始捕获文件** - 存档于独立硬盘/云端；原始文件是处理流程的源数据，其余内容均可由此重新生成
* ✅ **`.daq` / `.csv` 光传感器文件** - 后续重新推导反射率时需要
* ✅ **处理后的输出结果** - 保留已校准的图像和指数
* ✅ **项目文件夹**（`project.json` 及相关文件）—— 包含所有设置，以便在需要时重新处理
* ✅ **调试日志**—— 记录处理细节
* ✅ **校准目标图像** - 用于验证和重新处理**存储建议：*** **即时备份**：外置硬盘
* **长期归档**：云存储（Google Drive、Dropbox 等）
* **关键数据**：在不同位置保留 2-3 份副本***

## 后续处理运行

### 复用项目设置

如果将来需要处理类似的数据集：

1. **保存项目模板**（如果尚未保存）
2. 使用已保存的模板**创建新项目**

3.**导入新图像**

4. 为保持一致性，使用相同的设置进行**处理**

### 批量处理多个会话

对于多个会话/数据集：

**方案 1：GUI - 多个项目**

* 为每个会话创建独立的项目
* 使用一致的模板设置
* 逐个进行处理

**方案 2：Chloros CLI（仅限 Chloros+ 及以上版本）**

* 自动化批量处理
* 使用脚本处理多个文件夹
* 请参阅 [CLI 文档](../CLI.md) 和 [CLI 参考指南](../reference/cli-reference.md)

**选项 3：Python SDK（仅限 Chloros 及以上版本）**

* 程序化控制
* 与分析管道集成
* 请参阅 [API 文档](../api-python-sdk.md) 和 [SDK 参考](../reference/sdk-reference.md)

***

## 后处理故障排除

### 使用不同设置重新处理

如果结果不理想：

1. 保留原始图像（切勿删除）
2. 在 Chloros 中打开同一项目
3. 在“项目设置”面板中调整设置
4. 重新处理 — 输出文件将保存到相同的产品文件夹中，因此会覆盖前一次运行中同名的文件

### 处理图像子集

若仅需重新处理特定图像：

1. 创建新项目
2. 仅导入需要重新处理的图像
3. 使用相同的设置模板
4. 处理较小的数据集

### 获取帮助

如果您遇到问题：

* 📧 **电子邮件**：info@mapir.camera （请附上调试日志）
* 🌐 **技术支持**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **常见问题解答**： [常见问题](../faq.md)
* 📖 **文档**：[Chloros 手册](../)***

## 总结：完整工作流

您现已完成完整的 Chloros 处理工作流：

1. ✅ **创建项目** - 参见 [项目](../projects.md)
2. ✅ **添加文件** - 参见 [添加文件](adding-files-to-a-project.md)
3. ✅ **调整设置** - 参见 [调整项目设置](adjusting-project-settings.md)
4. ✅ **标记目标** - 参见 [选择目标图像](choosing-target-images.md)
5. ✅ **开始处理** - 参见 [开始处理](starting-the-processing.md)
6. ✅ **监控进度** - 参见 [监控处理过程](monitoring-the-processing.md)
7. ✅ **查看结果** - 本页**您经过校准并经过反射率校正的多光谱图像现已准备就绪，可供分析！**

***

## 附加资源

### 高级功能

* [**图像查看器**](../image-viewer-gui/opening-an-image-full-screen.md) - 交互式可视化与分析
* [**指数/LUT 沙盒**](../image-viewer-gui/index-lut-sandbox.md) - 自定义指数测试
* [**多光谱指数公式**](../project-settings/multispectral-index-formulas.md) - 完整的指数参考

### 自动化与集成

* [**CLI 文档**](../CLI.md) - 命令行批处理
* [**Python SDK**](../api-python-sdk.md) - 编程自动化
* [**Chloros+ 功能**](../#chloros) - 高级处理功能

### 支持与学习

* [**常见问题解答**](../faq.md) - 常见问题解答
* [**校准目标**](../calibration-targets.md) - 了解反射率校准
* [**支持的相机**](../supported-cameras.md) - 兼容硬件
