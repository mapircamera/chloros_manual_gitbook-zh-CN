# LATTICE 相机

LATTICE 是 MAPIR 专为农业和科学成像设计的模块化多光谱相机系统。 每台 LATTICE 相机均基于索尼 IMX265 全局快门传感器（**

3.1 MP，3.45 µm 像素**）构建，并作为**GigE Vision** 设备通过以太网连接。

Chloros 1.2.0 可通过以下三种界面实时控制 LATTICE 相机——包括设备发现、实时预览、图像捕获以及多相机阵列同步功能：

| 界面    | 位置                                                          | 支持平台                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| 图形用户界面        | Chloros 侧边栏中的 **相机** 选项卡                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice` 命令集                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **正在寻找硬件吗？**摄像头模块、镜头、滤光片和波段、框架和安装件、电缆、PoE 及触发线接线均在 [**LATTICE 用户手册**](https://mapir.gitbook.io/lattice-camera) 中有详细说明。 本章介绍如何通过 Chloros 驱动摄像头。

LATTICE 捕获文件是标准的 `.tif`/`.tiff` 格式文件，而 Chloros 始终从原始捕获数据开始对其进行处理。 请参阅 [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md) 以了解完整的命令和 API 界面。

## 两种传感器配置

| 配置 | 传感器       | 滤波器                                | 单台相机提供的内容                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| 拜耳彩色 | 三波段带通滤光片                |**单次曝光即可获得三个校准波段**                 |
| **M3M**| 单色         | 单个窄带干涉滤光片 |**一个校准波段**；组合多台 M3M 相机以获得指数 |

由于 M3M 相机在单个滤光片后为单色，因此每个波段都有独立的曝光。M3C 相机则通过一次传感器曝光即可覆盖其全部三个波段。

## 型号字符串与命名

每台相机都会将其标识以型号字符串的形式存储在 GenICam `DeviceUserID` 中：

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros 会以 `LATT-` 为前缀显示该标识（例如 `LATT-M3M-L87-F450`）。 相同的 `LATT-…` 字符串会被写入每次导出的 EXIF `Model` 标签中，并在已处理的项目中用作相机的输出文件夹名称。

| 组件 | 值                                                   | 含义                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 传感器    | `M3C` / `M3M`                                            | 拜耳彩色 / 单色                                                                          |
| 镜头      | `L41` / `L87`                                            | 该数字表示**水平视场角（单位：度）**： L41 = 窄角（41°），L87 = 广角（87°）    |
| 滤光片    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) 或 `F<nm>` (M3M) | 参见 [滤光片与光谱波段](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

型号字符串决定了后续的所有设置：Chloros 会根据 `DeviceUserID` + `DeviceSerialNumber` 确定传感器配置文件、波段布局和出厂校准。 无需针对每台相机进行单独配置——请参阅 [连接相机](connecting.md)。

## 滤光片与光谱带

波段中心、FWHM波段边缘以及完整的 23 种 SKU 的 M3M 产品目录均属于产品规格，因此这些内容收录在硬件手册中：[**滤光片与光谱波段**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands)。

软件方面需要注意的是：模型字符串中的滤光片代码决定了Chloros能够生成哪些产品。 RGB 滤光片相机（`FRGB`）仅输出去拜耳化及预览产品——对于宽带传感器而言，各波段的辐射度和反射率并无实际意义， 因此 Chloros 会跳过这些参数并明确标注。其他所有滤光片均可生成完整的辐射度 → 反射率 → 指数转换链。

## 辐射校准概览

每台 LATTICE 相机均在工厂通过可追溯至 NIST 的校准链进行单独校准，并随附每台相机的校准证书。 校准范围、测量方法以及可引用的精度均详见硬件手册：[**工厂辐射校准**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)。

在软件方面，关键在于当相机连接时，Chloros 能确定正确的校准方案，并将应用的系数固定在每次导出中——详见 [连接相机](connecting.md)。

## 本章内容

* [连接相机](connecting.md) — 自动发现、GUI 连接对话框、 CLI/SDK 对应功能，以及相机连接时如何处理出厂校准（相机内置包与云端校准）。

本手册的其他 LATTICE 主题——相机设置与实时控制、捕获模式、多相机阵列，以及单色 (M3M) 处理和索引——将在本手册的相应章节中详细介绍，完整的命令列表请参见 [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md) 中。
