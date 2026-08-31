---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# 输出图像格式

Chloros 以四种文件格式导出处理后的结果。 可在“项目设置”（GUI）中选择格式，也可通过 `--format`（CLI）或 `export_format` (SDK)。CLI 和 SDK 接受以下精确字符串。

| 格式字符串 | 扩展名 | 像素类型 | 像素范围 | 备注 |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(默认)* | `.tif` | uint16 数字 | 0 – 65535 | 推荐用于摄影测量/GIS。 |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0.0 – 1.0 | 1.0 表示 100% 反射率。部分应用程序无法读取浮点 TIFF 文件；文件体积较大。 |
| `PNG (8-bit)` | `.png` | uint8 数字 | 0 – 255 | 无损压缩，适用于网页浏览和可视化。 |
| `JPG (8-bit)` | `.jpg` | uint8 数字 | 0 – 255 | 有损压缩，文件体积最小。 |

## 输出文件的存储位置

生成文件保存在项目文件夹下，按相机分组，然后按文件格式排序：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE 的相机文件夹为 `LATT-<sensor>-<lens>-F<filter>`，而 Survey3 的相机文件夹为 `<model>_<filter>`（例如 `Survey3N_RGN`）。 **每个导出的产品都保留源文件的名称——文件夹用于标识产品，而非文件名后缀。** 有关完整规则，请参阅《CLI参考手册》中的[输出文件存储位置](reference/cli-reference.md)。

## LATTICE 产品（捕获和导出级别）

一个 LATTICE 原始帧会在单次处理中拆分生成所有请求的产品。每种产品类型都有其专属的开关（GUI 复选框，或 CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`，默认均开启）：

| 级别 | 内容 | 数据类型 |
| --- | --- | --- |
| `raw` | 直接来自传感器的拜耳数据（单色相机：单波段）。处理始终从原始数据开始。 | 原始捕获数据 |
| `debayered` | 线性去马赛克——M3C 为 3 通道，M3M 为 1 通道灰度。 | 线性 DN |
| `radiance` | 来自完整辐射测量链的绝对光谱辐射度，单位为 **W/m²/sr/nm**。 无论选择何种导出格式，均以 32 位 TIFF（`tiff32/Radiance_Images/`）形式存储。 | float32 |
| `reflectance` | 反射率 ρ，其中 **DN 32768 = ρ 1.0 (100%)**，且具有扩展至 ρ 2.0 的余量。支持 Pix4D 格式。 | uint16 |
| `preview` | 显示就绪渲染：RGB = 白平衡 + 伽马校正；多光谱 = 假色拉伸。 | 8 位显示 |

## 读取反射率像素值

反射率以整数数字值（DN）形式存储，而**表示 ρ = 1.0（100% 反射率）的 DN 值取决于源相机**：

| 源相机 | ρ = 1.0 对应的 DN | 如何判断 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768`（动态范围上限为 ρ 2.0） | 文件中带有 XMP 标签 `Chloros:PixelScale=32768`。 |
| Survey3 | `65535`（在 ρ 1.0 处被裁剪） | 没有 `Chloros:*` XMP 标签——这种缺失即为特征。 |

**读取 `Chloros:PixelScale` XMP 标签并以此除以**，而不是假设一个常数。 该标签定义在 uint16 域中，因此在进行比例调整的输出格式中其值保持不变——请先将存储的数据类型归一化回 uint16（8 位数据乘以 257，float32 数据乘以 65535）。

{% hint style="warning" %}
**根据设计，有一种情况不包含缩放因子。** 当将 8 位源捕获数据（BayerRG8）写入为 8 位 TIFF 时， 管道会将其裁剪为 0–255 范围，而非重新缩放，因此该文件不存在缩放信息——Chloros 在此处有意省略了 `Chloros:PixelScale`。 如果 LATTICE 反射率文件中缺少该标签，请勿假设存在比例；而是应以 16 位或 32 位格式重新导出。
{% endhint %}

有关完整规则（包括与 MicaSense 兼容的标签），请参阅 [CLI 参考指南](reference/cli-reference.md) 中的 **“读取反射率像素”**。
