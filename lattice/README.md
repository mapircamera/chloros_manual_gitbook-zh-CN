# LATTICE 相机

LATTICE 是 MAPIR 专为农业和科学成像设计的模块化多光谱相机系统。每台 LATTICE 相机均基于索尼 IMX265 全局快门传感器（**

3.1 MP，3.45 µm 像素**）构建，并作为**GigE Vision** 设备通过以太网连接。

Chloros 1.3.0 可通过以下三种界面实时控制 LATTICE 相机——包括设备发现、实时预览、图像捕获以及多相机阵列同步：

| 界面    | 位置                                                          | 支持平台                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| 图形界面        | Chloros侧边栏中的**“相机”**选项卡                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice` 命令集                           | Windows 10/11 x64、Linux x86\_64、Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64、Linux x86_64、Linux aarch64 (Jetson) |

> **正在寻找硬件吗？**摄像头模块、镜头、滤光片和波段、机架和安装支架、线缆、PoE 及触发线接线等内容均在 [**LATTICE 用户手册**](https://mapir.gitbook.io/lattice-camera) 中有详细说明。本章介绍如何通过 Chloros 驱动摄像头。

LATTICE 捕获的文件为标准 `.tif`/`.tiff` 格式，Chloros 始终从原始捕获数据开始对其进行处理。 有关完整的命令和 API 接口，请参阅 [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md)。

## 两种传感器配置

| 配置 | 传感器       | 过滤器                                | 单台摄像头提供的内容                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| 拜耳彩色 | 三波段带通滤光片                |**单次曝光即可获得三个校准波段**                 |
| **M3M**| 单色            | 单窄带干涉滤光片                |**一个校准波段**；组合多台 M3M 相机以获取指数 |

由于 M3M 相机在单个滤光片后为单色，因此每个波段都有独立的曝光。 而 M3C 相机仅需一次传感器曝光即可覆盖其全部三个波段。

## 型号字符串与命名

每台相机都会将其标识以 GenICam `DeviceUserID` 的形式存储为型号字符串：

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros会以`LATT-`为前缀显示该字符串（例如`LATT-M3M-L87-F450`）。 相同的 `LATT-…` 字符串会被写入每次导出的 EXIF `Model` 标签中，并作为已处理项目中相机的输出文件夹名称使用。

| 组件 | 值                                                   | 含义                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 传感器    | `M3C` / `M3M`                                            | 拜耳彩色 / 单色                                                                          |
| 镜头      | `L41` / `L87`                                            | 该数字表示**水平视场角（单位：度）**： L41 = 窄视场 (41°)，L87 = 广视场 (87°)    |
| 滤光片    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) 或 `F<nm>` (M3M) | 参见 [滤光片与光谱波段](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

型号字符串决定了后续的所有操作：Chloros会根据 `DeviceUserID` + `DeviceSerialNumber` 解析传感器配置文件、波段布局和出厂校准。 无需针对每台相机进行单独配置——请参阅 [连接相机](connecting.md)。

## 滤光片与光谱带

光谱带中心、FWHM边界以及完整的 23 种 SKU 的 M3M 产品目录均属于产品规格，因此这些信息收录在硬件手册中：[**滤光片与光谱带**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands)。

软件方面需要注意的是：型号字符串中的滤光片代码决定了Chloros可以生成哪些产品。 RGB滤光片相机（`FRGB`）仅输出去拜耳化及预览产品——对于宽带传感器而言，各波段的辐射度和反射率数据并无实际意义，因此Chloros会跳过这些数据并予以说明。其他所有滤光片均可生成完整的辐射度 → 反射率 → 指数转换链。

## 辐射校准概览

每台 LATTICE 相机均在工厂针对可追溯至 NIST 的校准链进行单独校准，并随附每台相机的校准证书。 校准范围、测量方法以及可引用的精度均详见硬件手册：[**出厂辐射校准**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)。

在软件方面，关键在于当相机连接时，Chloros能解析正确的校准参数，并将应用的系数固定到每次导出中——详见[连接相机](connecting.md)。

## 本章内容

* [连接相机](connecting.md) —— 自动发现、GUI 连接对话框、CLI/SDK 对应操作，以及相机连接时如何确定出厂校准（相机内置包与云端校准）。
* [相机设置](camera-settings.md) — “相机”选项卡中的所有控件：相机列表、单个相机设置面板以及阵列设置面板（包括ROI编辑器和对齐功能）。
* [采集设置与模式](capture.md) — 全部采集、单帧/连续/间隔采集、导出类型以及采集文件夹结构。
* [多摄像头阵列](arrays.md) — 阵列连接向导、同步电缆探针、带宽、PTP 以及实时阵列视图。
* [单色相机与植被指数](mono-indices.md) — M3M 相机以及对齐 → 堆叠 → 指数的工作流程。

完整的命令集详见 [CLI 参考](../reference/cli-reference.md) 和 [SDK 参考](../reference/sdk-reference.md)。
