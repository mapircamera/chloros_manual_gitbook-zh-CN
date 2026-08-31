# Chloros CLI 参考文档

**版本：**

1.2.0**生成时间：**2026-07-29 19:19 ·**修订日期：** 2026-08-30**目标受众：** 针对大型语言模型（LLM）使用进行优化；同时兼顾人类可读性。**范围：** `chloros-cli` 工具中所有面向用户的子命令，包含选项及可直接复制粘贴的示例。

本文档是随MAPIR Chloros 发布的 `chloros-cli` 命令行工具的完整参考指南。本文档刻意做到详尽无遗，以便 LLM（或人类）无需查看源代码，即可根据下文列表构建任何受支持的工作流。

若您只需了解重点内容，请跳转至：
- [五分钟快速入门](#five-minute-quickstart)
- [LATTICE 相机首次连接工作流](#lattice-camera-first-connect-workflow)
- [DAQ 传感器首次连接工作流](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [采集模式、记录器与离线重处理](#捕获模式-记录器--离线重新处理)

---

## 命名规范

- 所有命令均以 `chloros-cli` 为前缀。在 Windows 上，二进制文件为 `chloros-cli.exe`；在 Linux /Jetson 上，则为 `chloros-cli`。
- 可选参数表示为 `--flag`。 必需的位置参数不带方括号。
- 若提供了默认值，省略该标志时将使用该值。
- CLI 是一个基于 Chloros 后端（运行在 `127.0.0.1:5000` 上的 Flask 服务器）的轻量级 HTTP 客户端。该后端由大多数命令自动启动。`CHLOROS_BACKEND_URL=<url>`将**`lattice`**、**`project`**和**`daq pool-*`**命令族指向远程后端——核心命令 (`process`、`login`、 `logout`、`status`、`export-status`、`time-sync`、`selftest`) 故意固定 `http://127.0.0.1:<port>` 并忽略它（IPv4 字面量可避免 Windows&#x27; `localhost`→`::1` 每请求约 2 秒的性能开销）。参见 [环境变量](#environment-variables)。
- 所有 SDK / CLI 调用均需使用 Chloros+ 账户登录（每台机器运行一次 `chloros-cli login`；结果缓存于 `~/.chloros/`）。
- 示例中使用 Linux 路径；在 Windows 上，请将 `/home/user/...` 替换为 `C:/Users/.../...`。

---

## 顶级概述

```
chloros-cli [global options] COMMAND [command options]
```

### 全局选项

| 标志 | 描述 |
| --- | --- |
| `--backend-exe PATH` | 覆盖自动检测到的后端可执行文件。 |
| `--port N` | 后端HTTP端口（默认：`5000`）。 |
| `-v, --verbose` | 启用详细输出。 |
| `--restart` | 强制重启后端（终止所有正在运行的 `backend_server.py`）。 |
| `--version` | 显示版本信息（`Chloros CLI 1.2.0`）。 |
| `--help` | 显示顶级帮助。 |

### 命令索引

| 命令 | 用途 |
| --- | --- |
| [`process`](#chloros-cli-process) | 对Survey3或LATTICE捕获文件夹进行端到端处理。 |
| [`login`](#chloros-cli-login) | 使用Chloros+ 账户对本机进行身份验证。 |
| [`logout`](#chloros-cli-logout) | 清除缓存的凭据。 |
| [`status`](#chloros-cli-status) | 显示当前许可证/身份验证状态。 |
| [`export-status`](#chloros-cli-export-status) | 在 `process` 运行期间显示 Thread-4 的实时导出进度。 |
| [`language`](#chloros-cli-language) | 设置或列出CLI的显示语言 （支持 38 种语言）。 |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | 默认项目文件夹（与图形界面共享）。 |
| [`update`](#chloros-cli-update) | 检查并安装CLI更新（Linux /Jetson）。 |
| [`selftest`](#chloros-cli-selftest) | 系统诊断 + 基本功能测试。 |
| [`time-sync`](#chloros-cli-time-sync) | PTP 主时钟状态/控制。 |
| [`lattice`](#chloros-cli-lattice) | LATTICE 相机控制与采集（45+ 个子命令）。 |
| [`daq`](#chloros-cli-daq) | DAQ 光谱传感器控制（DAQ-U / DAQ-M / DAQ-E）。 |
| [`project`](#chloros-cli-project) | 打开并运行已保存的Chloros项目（相机 + DAQ）。 |

---

## 安装

`chloros-cli` 包含在所有受支持平台上的 Chloros 桌面安装程序中——无需单独下载 CLI。安装平台软件包后，`chloros-cli` 将与 桌面应用程序及其驱动的后端二进制文件一同安装到您的 `PATH` 中。

最新下载：[`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> 安装程序还附带了便捷的启动脚本（`Chloros_CLI.bat` / `Chloros_CLI.ps1`、`Launch_CLI.*`、`chloros-cli.sh`），这些脚本可直接打开的CLI命令行界面；相关内容已在[CLI用户指南](../CLI.md)中详细说明，此处不再赘述。

### Windows (.exe)

1. 从下载页面下载 Windows 安装程序。
2. 运行 `Chloros-Setup-x.y.z.exe` 并按照向导操作。默认安装路径为 `C:\Program Files\Chloros\`（CLI 文件位于 `C:\Program Files\Chloros\cli\`， 安装程序会将其添加到 PATH 环境变量中）。
3. 打开一个新的终端（`cmd.exe`、PowerShell 或 Windows 终端），以便识别更新后的 `PATH`。

```powershell
chloros-cli --version
```

安装程序会自动将 `chloros-cli.exe` 添加到您的系统 `PATH` 中，并捆绑了 LATTICE 相机所需的 Arena SDK 运行时。

### Linux amd64 (.deb)

适用于 Ubuntu 22.04 LTS 或更新版本 / 基于 Debian 的 x86_64 工作站。

> **不支持 Ubuntu 20.04。** 该软件包的依赖项列表源自
> 后端实际链接的对象，其中包括 `libc6 (>= 2.34)`；
> Focal 系统自带 glibc 2.31。`apt` 会直接拒绝安装，而非在
> 运行时导致安装失败。

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

该 .deb 包将安装：
- `chloros-cli` 升级至 `/usr/bin/chloros-cli`
- 编译后的后端升级至 `/usr/lib/chloros/chloros-backend`
- Arena SDK 运行时（适用于 LATTICE 相机）
- 去噪模型、校准包和更新通道配置

### Linux arm64 — Jetson（JetPack 6）

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

布局与 amd64 .deb 包相同，包含针对 Jetson Orin / Orin NX / Orin Nano 优化的 CUDA 构建版本。

### 每台机器仅需认证一次

在调用 SDK / CLI 之前，每个平台都需要进行一次 Chloros+ 登录：

```bash
chloros-cli login user@example.com 'YourPassword'
```

凭据缓存于 `~/.chloros/user_session.json` 中。

### 验证安装

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+ 订阅必不可少。**CLI 需要有效的 Chloros+ 套餐。**Copper**是入门级Chloros+套餐——所有付费Chloros+套餐均包含CLI / SDK访问权限；仅免费的**Iron**套餐不包含。（套餐ID映射：`0`=Iron/免费，`1`=Copper，`2`=Bronze, `3`=Silver, `4`=Gold。）请通过 [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing)处进行升级。
>
> 该限制由后端强制执行，而不仅仅是CLI：带有SDK / CLI标记但未订阅付费套餐的请求将被拒绝并返回状态码`403 PLAN_UPGRADE_REQUIRED`，无论该请求来自`chloros-cli`、Python SDK，还是手动编写的HTTP客户端。已登录的调用者则会收到错误代码 `401 AUTH_REQUIRED`。在套餐的宽限期内（按年计费的套餐为每月 30 天， 年费计划则至到期日）仍可离线访问，该期限届满后即停止；`chloros-cli status` 仍可正常工作，因此原因一目了然（这是 SDK / CLI 路径中唯一不受分级限制（`GET /api/license-status`）的例外）。

---

## 五分钟快速入门

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

将一个图像文件夹通过完整的Chloros处理流程进行处理（目标检测 → 校准 → 暗角校正 → 反射率 → 参数导出）。

### 概述

```
chloros-cli process INPUT [OPTIONS]
```

### 位置参数

| 参数 | 描述 |
| --- | --- |
| `INPUT` | 包含 `.raw + .jpg` （Survey3）、`.tif/.tiff`（LATTICE）或 `.dng` 文件的输入文件夹路径。 |

### 常用选项

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `-o, --output PATH` | 在默认项目路径下创建一个带有时间戳的新文件夹（除非另有配置，否则为 `~/Chloros Projects`） | 要创建或重用的项目文件夹。如果该文件夹中已存在 `project.json`文件，则会创建名为 `_1`/`_2` 的同级文件夹，而非覆盖原有文件夹。 |
| `-n, --project-name NAME` | 自动（时间戳） | 项目名称。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` 使用 Chloros+ 神经网络去拜耳算法；速度较慢但质量更高。 |
| `--vignette / --no-vignette` | `--vignette` | 暗角校正。 |
| `--reflectance / --no-reflectance` | `--reflectance` | 反射率校准（若检测到面板标靶则使用该标靶，LATTICE 设备则采用 NIST 序列校准）。对于 LATTICE 多光谱数据，此操作同时兼作反射率**乘积** 开关 — 参见 [按产品导出开关](#per-product-export-toggles-lattice-multispectral)。 |
| `--ppk` | 关闭 | 应用来自 sidecar 文件的 PPK GNSS 校正。 |
| `--exposure-pin-1 MODEL` | 关闭 | 固定Survey3双摄像头支架的“pin-1”模型。 |
| `--exposure-pin-2 MODEL` | 关闭 | 固定“pin-2”模型。 |
| `--recal-interval SECONDS` | 0 | 强制每 N 秒（以采集时间计）重新运行校准计算。 |
| `--timezone-offset HOURS` | 本地 | 覆盖输出元数据中内置的时区偏移量。 |
| `--format FORMAT` | `TIFF (16-bit)` | 以下其中之一：`TIFF (16-bit)`、`TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)`中的任意一项。 |
| `--indices NAME [NAME ...]` | 无 | 植被指数（`NDVI`、`NDRE`、 `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …）。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | 强制 LATTICE TIFF 的管道入口点（Survey3 .raw 文件不受影响）。此外，还提供了一个“逃生通道”，允许对包含 **无原始数据**的捕获完全不进行处理的“逃生通道”——参见 [捕获文件夹的结构](#what-a-captures-folder-looks-like)。 |
| `--debayered / --no-debayered` | 启用 | 输出线性去拜耳化结果（`Debayered_Images`)。参见 [按产品导出开关](#per-product-export-toggles-lattice-multispectral)。 |
| `--preview / --no-preview` | 开启 | 输出显示预览（`Preview_Images`）：RGB = 白 平衡（如有可用则采用DAQ光源，否则采用灰度世界）+ 伽马值；多光谱 = false-色彩拉伸。 |
| `--radiance / --no-radiance` | 启用 | 输出 float32 辐射度 (`Radiance_Images`, W/m²/sr/nm)。 |
| `--reflectance-source {daq,target,auto}` | `auto` | LATTICE反射率产品的参考标准：`auto` = 通过质量保证（QA）的帧内目标作为绝对参考，DAQ下行辐射（ρ = π·L/E）作为备用方案；`target` = 严格（不进行DAQ替换）；`daq` = 以DAQ数据为准。参见 [按产品导出开关](#per-product-export-toggles-lattice-multispectral)。 |
| `--target-reflectance-dir DIR` | 无 | 按单位**测得**的目标反射率扫描目录（`<serial>.csv`）；若未找到，则回退到标称的 T3/T4P 光谱。 |
| `--array-alignment / --no-array-alignment` | 启用 | LATTICE 数组：将 每个捕获任务的 `Chloros:Alignment*` XMP 文件中标记的模块间对齐信息，应用于所有处理后的产品（去拜耳化/预览/辐射度/反射率/索引）。对于没有这些标签的图像，此操作无效。 |
| `--array-alignment-crop / --no-array-alignment-crop` | 裁剪 | 将对齐后的导出结果裁剪至阵列的公共重叠区域，使所有模块共享同一图像区域；`--no-…` 则保留完整的传感器画布（源图像外部填充黑色）。 |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | 针对对齐变形进行重采样。`nearest` 保留精确的源数据动态范围 （不进行辐射值像素间混合）。 |

### 目标检测选项

| 标志 | 描述 |
| --- | --- |
| `--min-target-size PIXELS` | 检测器的最小面板目标尺寸（像素）。 |
| `--target-clustering 0-100` | 聚类灵敏度。 |
| `--target / --targets` | 将输入文件夹视为仅含目标面板（跳过调查检测）。 |

### 示例

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### 按产品导出开关（LATTICE 多光谱）

LATTICE 处理会在 **单次处理中生成所有适用的产品**。四种按类型设置的开关 ——`--debayered`、`--preview`、`--radiance`、`--reflectance`——均**默认开启**；使用 `--no-<type>` 表单禁用其中一个。RGB 主摄像头仅输出去拜耳化后的预览图像（不包含各波段的辐射度/反射率），因此对于它们而言，`--radiance`/`--reflectance` 属于无操作。对于 Survey3 `.raw`（其遵循标准的反射率/目标路径）时，这些开关会被忽略。*（旧的 `--radiometric-output {reflectance,radiance,sensor-response}` 标志已被**移除**并由这些开关取代；现已不再存在 `sensor-response` 级别。）*

| 产品 | 输出 | 是否需要 DAQ 下行数据？ |
| --- | --- | --- |
| `--debayered` | 线性去马赛克 (`Debayered_Images`). | 否 |
| `--preview` | 显示预览（`Preview_Images`）：RGB=白平衡 + 伽马校正；多光谱 = 假色拉伸。 | 否 |
| `--radiance` | float32 W/m²/sr/nm，来自完整的辐射测量链（`Radiance_Images`）。 | 编号 |
| `--reflectance` | uint16 反射率 ρ（`32768` = 1.0），Pix4D 兼容。 | **是**，除非有通过质量保证（QA）的帧内目标将其锚定 （见下文）。 |

`--reflectance-source` 选择反射率参考：**`auto`**（默认）将通过质量保证（QA）的帧内目标设为**绝对参考**——基于目标锚定的经验线链会在保留的面板上进行交叉评分，并应用测得的优胜结果——当无目标存在或质量保证失败时，则回退至数据采集（DAQ）向下分界线（ρ = π·L/E）；**`target`**采用严格模式（不进行数据采集替换）；**`daq`**选择采用 DAQ 权威行为。目标几何参数（ArUco / 固定 ROI / 条带）来自项目目标配置；`--target-reflectance-dir DIR` 通过目标单元的序列号/QR 查询每个单元的**测量**扫描数据（`<serial>.csv`），这些数据通过目标单元的序列号/QR进行查找，并以标称的 T3/T4P 光谱作为备用方案。

DAQ反射率路径会从记录的**`.daq`**（DAQ-U/M/E）**或与图像一同发现的DAQ-M原生`.csv`**中自动解析出**时间戳匹配的下行光谱**。如果本地未缓存相应的相机或DAQ校准包，处理管道将在首次使用时**从AWS自动获取**（需连接互联网一次；缓存于`~/.chloros/`中）。

#### 读取反射率像素（Pix4D / Metashape / 您自己的脚本）

反射率以整数 DN 形式存储，**表示 ρ = 1.0 的具体数值取决于源相机**：

| 源 | ρ = 1.0 对应 | 如何识别 |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768`（ρ 范围上限为 2.0） | 文件上标有 XMP `Chloros:PixelScale=32768` 标记。 |
| Survey3 | `65535` （在 ρ 1.0 处截断） | 没有 `Chloros:*` XMP 标签——这种缺失*本身*就是信号。 |

**读取 `Chloros:PixelScale` 并以此除**，而不是假设一个常数。该标签在 uint16 域中定义，因此即使在进行重新缩放的输出格式中，它仍保持不变——`32768` `TIFF (16-bit)`、`PNG (8-bit)`、`JPG (8-bit)` 和 `TIFF (32-bit, Percent)` 均具有自描述性（需先将存储的数据类型归一化回 uint16： 将 8 位数据归一化为 uint16 的 257 倍，将浮点数归一化为 65535 倍）。

> **其中一种情况按设计不包含缩放因子。** 当 8 位源捕获数据（BayerRG8）被写入为 8 位TIFF 格式时，处理管道会将数据 *裁剪* 范围裁剪至 0..255 而非重新缩放，因此所有大于 ρ≈0.008 的值都会被压平为 255，且该文件不包含任何缩放信息。Chloros 会刻意省略其中的 `Chloros:PixelScale` 和 `MicaSense:RadiometricCalibration` 元组，并记录了原因。**如果 LATTICE 反射率文件中缺少该标签，请勿假设存在比例——应以 16 位或 32 位重新导出**，而不是对原本不可除的像素进行除法运算。

#### 导出时保留的 EXIF 信息

`process` 会将源捕获文件的 **GPS 块及其 ExifIFD** 复制到每个生成文件中，因此
导出文件中会包含 `FocalLength`、`FNumber`、`ExposureTime`、`ISO`、`DateTimeOriginal` 以及
`CameraSerialNumber`等标签，并附带地理参考信息。

**`FocalLength`对摄影测量而言是必不可少的。** Pix4D会根据
焦距和飞行高度计算地面采样距离；若缺少该标签，系统将退而采用一个严重错误的比例尺。在一次
包含49张航拍照片的橘园飞行任务中，由于该标签缺失，原本411米×160米的场地被重建为
47.8公里×13公里——这张4.55亿像素的正射影像中大部分区域显示为“无数据”，随后被误判为平铺问题，
BigTIFF问题，直到有人检查GSD之前都未能发现。如果您的正射影像生成时比例尺不合常理，
请先对导出的产品运行`exiftool -FocalLength`。

此副本特意**不是**`-all:all`：IFD0的结构标签在
复制时会导致LATTICE输出异常，而`ExifImageWidth` / `ExifImageHeight`被排除在外，因为它们描述的是
*源*采集数据——否则，任何曾被调整过尺寸的导出文件都会携带与自身 栅格。XMP文件是直接写入而非复制，因为当XMP块被复制时，ExifTool
会丢弃同一调用中的XMP标签（否则会导致MAPIR
校准标签丢失）。

### 输出文件的位置

生成文件将写入**项目文件夹下，按相机分组，然后按文件格式分组**：

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

相机文件夹为 `LATT-<sensor>-<lens>-F<filter>`（对应 LATTICE 的相机文件夹为 `LATT-<sensor>-<lens>-F<filter>`（与拍摄文件的 EXIF
`Model` 信息一致），而 Survey3 的相机文件夹为 `<model>_<filter>`——这两款相机虽共用同一传感器和滤光片，但因
镜头不同（导致暗角、视场和畸变存在差异），因此各自拥有独立的文件夹结构。格式
文件夹 遵循 `--format`：`tiff16`、`tiff8`、`png8`、`jpg8`，或者 `tiff32` 对应
`TIFF (32-bit, Percent)`。

> **每个导出文件都保留源文件的名称。** 例如，将
> `capture_…_raw.tif` 的 Radiance 导出文件仍名为 `capture_…_raw.tif` —— 它只是位于
> `tiff32/Radiance_Images/` 目录下。**文件夹标识了该产品，而非文件名**，因此使用通配符
> `*radiance*.tif` 不会找到任何内容；请改用目录进行匹配。

### 光传感器记录 — 已校准的 `.daq` + `.csv`

`process` 还能处理输入文件夹中的 `.daq` 记录，且**无需**
任何图像数据即可完成：单独运行的 DAQ-U / DAQ-M / DAQ-E 单独飞行即可完成
完整采集，而仅包含 `.daq` 文件的文件夹也是有效的输入。

DAQ 可以在**未**进行校准的情况下进行记录——这正是公开的
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) 记录仪
(`record_daq.py`) 的默认行为：它们写入原始传感器计数并为文件添加时间戳，以便
Chloros **通过串行接口**（优先使用本地缓存，
然后MAPIR云端）获取该传感器的出厂校准值并应用。`process`将结果写回文件：

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` 文件中每行对应一次读数：UTC 时间戳、积分时间、总功率、
明视/暗视照度（lux）、PPFD（及其蓝/绿/红分量）、峰值波长，随后是
基于传感器自身波长网格上的完整光谱。`.daq` 会直接重新导入数据，无需
再次校准。

成功时，运行报告显示为 `Light-sensor products written: N (calibrated .daq + .csv)`。
括号中的内容描述了实际写入的内容，因此对于无文件包的传感器，显示为
`(RAW COUNTS — this sensor has no calibration bundle)`；对于同时包含两者的文件夹，则显示为
`(N calibrated, M raw counts)` 则显示为`(N calibrated, M raw counts)`。后端自身的
`[DAQ-EXPORT]`和`[RUN-SUMMARY]`标题的表述方式与此相同——这
三者均不会将未经校准的原始导出数据称为“已校准”。

如果 DAQ-U / DAQ-M / DAQ-E 的记录无法获取校准包——可能是
您处于离线状态，或者该传感器没有存档的校准数据——则该记录将在
`[DAQ-EXPORT]` 行上被 **将**跳过并附带原因说明**，显示在
`[DAQ-EXPORT]` 行中，绝不会作为包含原始计数值的“已校准”文件写出。
请连接互联网并重新运行。该原因即为读取器实际
针对该文件确定的原因 （模式无法读取、无校准包、写入错误），而运行
摘要中列出的原因均为**独立**项——因同一原因被跳过的二十个文件会被视为一个
原因，而非该原因的二十次重复。

#### DAQ-A 记录以原始计数值导出

**DAQ-A**系列早于按序列号分捆的系统，因此没有可获取的校准捆包
——它是在现场通过反射率标靶进行校准的，这
正是它从未需要校准捆包的原因。拒绝这些记录导致用户完全无法
导出数据，因此它们会以**不同的名称**导出：

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

采用不同的文件名而非文件内部的标记，是因为该声明必须在
作为纯文件名通过电子邮件发送时仍能保持有效。`.csv` 文件头声明
`raw spectral sensor counts (NOT irradiance)`，并警告说这些数值仅
**在**文件内部可比——这正是基于目标的校准使用它们的目的——
而非跨传感器比较。与功率相关的光度学列（总功率、明视和
暗视勒克斯、PPFD）被写入为 **NULL** ，而非通过计数积分得出，且运行
摘要显示为 `RAW COUNTS`，因此日志中“导出”的这些数据不能被解读为辐照度。

旧版 **v1.01 / v1.02** 记录（由 DAQ-A-SD 写入）不包含每读数的时戳，
仅包含文件的写入时间。图像↔下行光匹配器仍然拒绝接受这些数据——将
帧与写入时间进行匹配会导致隐性错误 ——但导出工具能读取这些数据，且
CSV会输出`clock=daq_created_on`，因此该产品会明确标注其采用的时钟类型。

### 注意事项

- `process`会自动检测您的文件夹属于Survey3、LATTICE还是混合类型。
- 进度通过服务器发送事件（Server-Sent Events）流式传输；CLI 显示每个线程的实时进度（检测中、分析中、处理中、导出中）。
- 对于 Linux /Jetson，CLI 会检查交换空间，并在处理大型文件夹前可能发出警告。支持纹理的去拜耳算法还会在低功耗 Jetson 设备（Nano、 Orin Nano）上自动应用 GPU 频率限制。
- 运行成功时，报告会显示写入的图像产品数量（`Image products written: N`）。

#### 未写入任何图像的运行将失败

如果您请求了图像产品，但运行结果为**零**——仅输出 `project.json` 和
`calibration_data.json`——`process` 会将此视为失败：它会输出
`Processing finished but wrote no image products.` 并 **以非零状态退出**， 因此脚本可以
检测到此情况。该消息会列出项目文件夹及常见原因：

- 输入文件夹未被识别为捕获文件夹（请检查布局和 `--input-level`），或者
- 所有请求的产品均因不适用于这些相机而被跳过（例如，请求
  RGB 专用相机请求辐射度/反射率）。

请使用 `--verbose` 重新运行，并在后端日志中检查 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 相关行，
这些记录解释了各相机被跳过的具体原因，而这些信息通常不会出现在CLI的输出中。

一次刻意仅运行元数据的测试——所有产品选项均关闭且不包含`--indices`——仍然是
**成功的**，因为在此情况下输出空图像才是正确的结果。

同样，**仅光传感器运行**也是如此：一个包含 `.daq` 记录文件的文件夹
，该运行将根据其生成的已校准文件 `.daq` / `.csv` 进行评估。

---

## `chloros-cli login`

使用 Chloros+ 云账户对这台机器进行身份验证。凭据将安全地缓存于 `~/.chloros/user_session.json` 中。

```
chloros-cli login EMAIL PASSWORD
```

### 示例

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$`（从密码中去除或复制部分内容）。遇到 401 错误时，CLI 会自动重试：先在 `$$` 后追加内容，然后使用去重后的密码一半进行尝试； 如果重试成功，系统将登录并显示下次应使用的正确单引号语法。

> **无界面/脚本使用： 无缓存会话意味着会出现交互式提示符，而非快速失败。** 任何启动后端的命令（`process`、`status`、`export-status`、`time-sync`、 …）若在未缓存许可证/会话的情况下运行，会在继续执行前通过标准输入（stdin）进入交互式 `Email:` / `Password:` 提示符。因此，没有缓存会话的无人值守作业会因等待输入而挂起 ——在调度无界面作业之前，请先在每台机器上运行一次 `chloros-cli login EMAIL PASSWORD`。

---

## `chloros-cli logout`

清除缓存的会话，并在下次调用时强制进行新登录。

```bash
chloros-cli logout
```

---

## `chloros-cli status`

显示当前许可证等级 （铁/铜/青铜/银/金）、已认证用户以及设备绑定数量。

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

轮询 Thread-4 实时导出进度。在另一个 shell 中运行 `process` 期间，调用此命令是安全的。

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

设置CLI的显示语言（支持38种语言，包括中日韩、从右到左（RTL） 及印度文字）。在无法渲染脚本的旧版控制台上，会优雅地回退到英语。

```
chloros-cli language [LANG_CODE] [--list]
```

### 示例

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## 项目文件夹命令

这些命令用于管理默认项目文件夹的位置（与 GUI 共享）。

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ 仅限 Jetson。会检查 `version_url` 与 `/etc/chloros/update.conf` 是否匹配，并提示下载和安装相应的 `.deb`。

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

在 Linux/Jetson 上，CLI 还会在**每次启动时自动检查更新**（非阻塞，绝不会延迟命令执行）：它读取 `/etc/chloros/update.conf`，将结果缓存 1 小时至 `~/.chloros/update_cache.json`， 并在存在更新版本时输出 `Update available: vX.Y.Z / Run: chloros-cli update`。若出现任何错误或遇到 Windows，则静默跳过。

---

## `chloros-cli selftest`

运行一个包含7个步骤的初步测试：版本检查、端口可用性、后端启动、`/api/test`、`/api/system-info` （GPU/CUDA/PyTorch）、去噪模型是否存在、CUDA+去噪器就绪状态。

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP主时钟状态与控制。Chloros主机运行PTP主时钟；LATTICE摄像头和DAQ-E单元作为从设备与其联动，以实现跨设备时间戳同步。

| 子命令 | 描述 |
| --- | --- |
| `status` | 显示主时钟状态、BMCA优先级及时钟标识。 |
| `peers` | 列出通过 Delay_Req 检测到的从设备（摄像头 + DAQ-E传感器）。 |
| `cameras` | 每台摄像机的PTP健康状态（`PtpStatus`、`PtpOffsetFromMaster`、`PtpMeanPathDelay`）。 |
| `restart` | 重启主控进程。 |
| `set-priority --priority1 N --priority2 N` | 覆盖 BMCA 优先级。 |

### 示例

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE 摄像头控制。每个子命令均通过 Chloros 后端进行路由；该后端拥有摄像头池，因此后续的 CLI 调用将复用同一个已打开的句柄。

### 常用选项（大多数子命令共用）

| 标志 | 说明 |
| --- | --- |
| `-d, --device N` | 摄像头索引（默认：0）。 |
| `-s, --serial SN` | 特定序列号；覆盖 `--device`。 |
| `--serials SN1,SN2,…` | 逗号分隔的序列号，用于多摄像头操作。 |
| `--all` | 对每个已发现的摄像头执行操作。 |
| `--exposure US` | 曝光时间（单位：微秒）。 |
| `--gain DB` | 以分贝（dB）为单位的增益。 |
| `--pixel-format FMT` | 例如：`BayerRG8`、`BayerRG12`。 |
| `--width N` / `--height N` | 图像尺寸。 |
| `--preset {default,high_quality,high_speed,triggered}` | 应用预设。所有运行，但 `triggered` 除外——该预设会使摄像头等待第 2 条线路出现硬件高电平信号；若该线路无信号驱动，系统将无限期等待而非进行捕获。 |
| `-o, --output DIR` | 输出目录（默认：`output`）。 |
| `--packet-size {auto,jumbo,standard,N}` | GVSP 数据包大小。`auto` 运行 ICMP+GVSP 探测；`jumbo` = 9000；`standard` = 1500。 |

### LATTICE 摄像头首次连接工作流

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### 子命令参考

#### 发现与信息

| 子命令 | 用途 |
| --- | --- |
| `lattice info` | 列出已连接的摄像头（厂商、型号、序列号、IP、MAC）。 |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | 分析主机系统以获取最佳摄像头配置。 `--no-discover` 跳过摄像头发现（速度更快，仅分析网卡）。 |
| `lattice network [--fix] [--estimate] [--cameras N]` | 检查/修复网卡设置；估算带宽/帧率。 |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | 基于稳定架构的后端网络能力评估及数组推荐（返回 `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`）。`auto_capped_fps` 保留请求的分辨率，但对目标 帧率（fps）——读取 `recommended.recommended_target_fps` 并将其作为连接目标传递；将其视为成功，而非错误。 |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | 无需打开摄像头即可进行假设分析。**`--n-active` 表示网络上的摄像头总数，而不仅仅是此数组中的**——当独立摄像头并发流媒体传输时，或网络带宽预算基于 （默认：`len(--models)`）。始终打印聚合值 `Wire budget:`（MB/s 需求值与防冲突上限的对比）以及 `Max cameras:` 行， 并在数组对总线超额订阅时标记 `**OVER-SUBSCRIBED**` —— 参见 [数组帧率与突发模型](#array-fps--burst-model)。 |
| `lattice gpu` | 显示 GPU 状态。 |
| `lattice firmware [--update] [--force] [-y\|--yes]` | 检查或更新相机固件。本地 `.fwa` 选项被锁定：若存在与该构建版本的 `firmware/<MODEL_PREFIX>/`中与该构建版本的`MIN_FIRMWARE_VERSION`匹配的文件，若存在则会被刷入 （仅将最高版本作为备用方案），因此存储在磁盘上的更新供应商镜像在该引脚被触发前处于非活动状态——较新的版本会通过签名的 AWS 清单有意地推送至设备，当有更新版本时，建议优先采用此方式。 |
| `lattice presets [--apply NAME]` | 列出或应用摄像头预设。 |
| `lattice status` | 摄像头实时状态。 |

#### 捕获

| 子命令 | 用途 |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | 单帧。**默认保存所有导出类型**（`--processing all`）；参见 [捕获导出级别](#capture-export-levels-the-all-default)。`--levels` 保存明确指定的子集（覆盖 `--processing`）；`--force-daq` 会将分配的 DAQ 读数写入为 `.daq` 旁文件，即使在仅原始数据的抓取中也是如此。 `--jpeg-quality` = JPEG 质量 1–100（默认 95）。 |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | 持续将流写入磁盘，直至按下 Ctrl+C。 |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | 基于浏览器的实时 MJPEG 预览。`--ae-damping` 设置自动曝光阻尼（0.4–100）。 |

#### 传感器调校

| 子命令 | 用途 |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | 读取/写入任意 GenICam 节点。 |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | 曝光与自动曝光。 |
| `lattice gain [--auto] [--off] [--set DB]` | 增益与自动增益。 |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | 传感器 ROI 与像素合并。 |
| `lattice format [--set FMT] [--list]` | 像素格式。 |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | 硬件/软件触发。 |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]`（无标志位 = 单次白平衡）| 白平衡操作。RGB仅限Bayer相机；在单色M3M上为无操作（跳过） 在单色M3M上。|
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB 显示色彩处理管道。`natural`（默认）是低成本的实时处理方案；`enhanced` 添加了去色差、鲜活度及CLAHE局部 对比度，以实现完整的“hub-parity”效果，每帧处理成本约为前者的2倍，因此会降低**实时** 帧率——无论哪种方式，保存的截图始终会获得完整的后期处理效果。RGB仅限/Bayer相机；在单色M3M上跳过。 |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | 显示饱和度/对比度（RGB滤镜相机）。在单色 M3M 上跳过。 |
| `lattice filter [--set NAME] [--list]` | 设置摄像头的滤镜型号 （`RGN-IMX265`、`OCN`、`NGB`、…）。 |
| `lattice power [--sleep]` | 探测器电源/热节点；用于实现低功耗待机。 |

#### 校准与传感器

| 子命令 | 用途 |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | 通过反射率标靶进行校准。 |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | 内置下射光传感器命令。 |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | 对现有图像应用暗角校正。 |

#### 多摄像头（瞬态会话）

| 子命令 | 用途 |
| --- | --- |
| `lattice multi-info` | 列出所有具有同步角色的摄像头。 |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | 从每台摄像机获取一帧同步帧。**默认保存所有导出类型**；当连接持久化数组时，临时无数组的备用方案仅进行**去拜耳处理**（如需处理其余帧，请先运行 `array-connect`）。 |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | 流式传输同步帧（临时）。 |
| `lattice multi-test [--count N]` | GPIO 同步时序测试。 |
| `lattice multi-detect [--line LINE] [--json]` | 自动检测 GPIO 主从接线。 |

#### 对齐

| 子命令 | 用途 |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — 加上检测器/匹配器旋钮 `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`、 RANSAC 参数 `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`、多帧组合 `[--averaging mean\|median\|inlier_weighted]`、几何约束 `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`、 空间限制 `[--roi X0,Y0,X1,Y1] [--mask PATH]`，以及每个从属节点的覆盖设置 `[--per-cam-override SN:KEY=VALUE]`（可重复） | 从实时摄像头计算对齐轮廓。`--prefilter` 默认值为 `gradient`（边缘图；与GUI/阵列对齐器一致——边缘在各光谱波段间保持一致）。`--matcher flann` 在特征数超过 ~5000 时效果显著；`--averaging median` 对单次错误捕获具有鲁棒性，`inlier_weighted` 按匹配次数加权；`--lock-scale` 投影至最近的旋转（无缩放），`--lock-axis` 将一个平移分量设为零；`--mask` 适用于所有相机（如需按相机设置，请使用 `--per-cam-override`，例如`--per-cam-override 214701292:method=phase`）。`--rms-threshold-px` 拒绝保存重投影均方根误差（RMS）超出阈值的校准。|
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | 捕获一帧对齐的多波段图像。 `--bit-depth` 默认匹配相机；`--no-crop` 保留全帧（用黑色填充）；`--interpolation`（默认 `linear`） 以及 `--border-mode`/`--border-value`（默认 `constant`/0）控制 CPU 畸变校正——无论如何，GPU 路径均为双线性插值。 |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | 流对齐多波段帧 （与 `align-apply` 具有相同的 Warp 控制参数）。 |
| `lattice align-info --profile PATH [--json]` | 显示配置文件详细信息。 |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | 更改图层顺序。 |

#### 索引 / 植被数学

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

完整标志集：`--input PATH | --live --profile PATH`、 `--preset NAME`（NDVI / NDRE / EVI / SAVI / GNDVI /…），`--formula EXPR`，`--channel SYM=BAND` （可重复），`--capture-level raw|debayered|radiance|reflectance|unknown`（覆盖源文件 TIFF 中记录的捕获级别；默认：从 TIFF 元数据中读取），`--output PATH`，`--output-format all|raw|tif|colorized|lut|png`，`--gradient NAME|JSON`、`--vmin/--vmax/--percentile LO,HI`、`--bg-mode clip|transparent|indexColor|backgroundColor`、`--colorize`、`--list-presets`、`--list-gradients`。对于 `--live`，对齐扭曲旋钮同样适用：`--save-multiband`、`--gpu/--no-gpu`、`--no-crop`、`--bit-depth 8|12|16`、`--vignette`、`--interpolation nearest|linear|cubic|lanczos`、`--border-mode constant|replicate|reflect|wrap`、`--border-value N`。

> **`--channel` 符号区分大小写。** 符号部分必须与预设的通道名称完全一致（预设使用小写，例如：NDVI= `red`,`nir` ——请检查 `--list-presets`），且频段部分必须与对齐堆栈中的频段名称匹配（或在离线模式下为以 0 为起点的频段索引）。`--channel red=Red_660 --channel nir=NIR_850` 有效；`--channel RED=660` 会因 `channel_map missing entries` 错误而失败。

#### 持久连接（Smart-Prep、 等效于GUI的流程）

这些命令可在多次调用CLI时，将摄像头保持在后端池中。

| 子命令 | 用途 |
| --- | --- |
| `lattice cam-connect [--serial SN]` | 将一台摄像头添加到池中 （单摄像头，非数组）。 |
| `lattice cam-disconnect [--serial SN] [--all]` | 释放。 |
| `lattice cam-list` | 列出池中的摄像头。 |
| **`lattice array-connect`**|**连接一个持久化同步阵列（推荐的入口点）。** 运行完整的 GUI 智能预处理流程。 |
| `lattice array-disconnect [--array-id ID] [--all]` | 释放阵列。 |
| `lattice array-list` | 列出已连接的阵列。 |
| `lattice array-status [--array-id ID]` | 实时帧率（fps）、PTP、上次错误。 |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | 从实时数组中获取一次同步捕获数据——单次/连续/间隔/最快。**默认值为 `all`**（每台摄像头按适用导出类型生成一个文件）。被跳过的摄像头（例如RGB等）将通过 `Skipped: SN:<serial> (<reason>)` 报告；用于反射率的 DAQ 读数将与之一同保存，并通过 `DAQ: <path>` 报告。 参见 [采集模式、记录器与离线重处理](#capture-modes-recorders--offline-reprocess)。 |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | 将实时组合索引视图记录为视频/GIF （监控级；需打开组合流）。 |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | 高帧率原始拜耳连拍（分析级；需离线重新处理）。 |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | 将已保存的原始连拍数据重新处理为校准后的视频。 |

##### `array-connect` 选项

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--serials SN1,SN2,…` | 自动发现所有 LATTICE 摄像头（需 ≥2 台） | 序列号最前的一台为 MASTER。若省略该参数，则仅发现 LATTICE（`TRI032*`）型号并连接所有设备。 |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO 同步线。 |
| `--target-fps F` | 自动 | 主控触发器发射速率。 |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | 自动 | 覆盖层级选择器。 |
| `--wire-ceiling-mbps MB_PER_S` | 自动检测 | **主机的持续带宽预算，单位为 MB/s ——整个数组分配所依赖的数值。**当数组报告GVSP损坏帧时，请降低该值：自动值源自网卡声明的链路速率， 这会高估 USB 适配器、带宽受限的 PCIe 通道以及繁忙的共享互连结构的实际性能。该值保存在项目阵列捕获块中，因此重新打开 / CLI / SDK 重新连接后将恢复该值。参见 [阵列健康状况](#array-health--which-subsystem-is-losing-frames)。 |
| `--binning {1,2,4}` | 自动 | 硬件分桶。 |
| `--no-recommend` | 关闭 | 跳过网络分析步骤。 |
| `--no-ptp` | 关闭 | 禁用 PTP（此时跨摄像头时间戳将**无法**进行比较）。 |

### 智能自动曝光（Smart-AE）/ 智能捕获（Smart-Capture）

LATTICE 阵列一经连接便会在后台持续运行自动曝光（AE），但新对焦的场景需要片刻时间才能收敛。`array-capture --smart` 是一个**预设的便捷功能**：它会等待阵列中所有摄像头的自动曝光稳定后，再触发拍摄。当您在-会话中切换场景时请使用此功能。

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

默认的稳定策略较为保守：5 秒超时，1.5 秒稳定性窗口，±5% 的曝光散布容差。若需通过自动化实现不同的行为，可通过SDK（`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`）进行调整 。

### 捕获导出级别（`all` 默认设置）

自本版本起，`lattice capture`、`lattice multi-capture` 和 `lattice array-capture` **默认设置为 `--processing all`** —— 每种导出类型对应一个保存文件，适用于每台相机，与 GUI 的“捕获全部”行为一致。级别如下：

| 级别 | 输出 | 适用对象 |
| --- | --- | --- |
| `raw` | 单通道拜耳 （单色摄像头：单波段）数据，直接来自传感器。 | 所有摄像头。 |
| `debayered` | 3 通道 BGR 去马赛克（单色摄像头：1 通道灰度）。 | 所有摄像头。 |
| `radiance` | 通过完整的辐射测量链获得的 float32 W/m²/sr/nm 值。 | 仅限多光谱（M3C/M3M）——**RGB滤光片相机将跳过此项**。 |
| `reflectance` | uint16 ρ（`32768` = 1.0)，支持Pix4D。 | 仅限多光谱，且**仅当已绑定DAQ且相机已校准时**有效；否则跳过。 |
| `preview` / `display` | 完整的 GUI 预览处理链（根据相机配置文件进行色彩校正（CCM）、白平衡（WB）和伽马校正）。`lattice capture` 将此  命名为00502；`array-capture`/`multi-capture` 使用 `display`。 | 所有相机。 |

传递单个级别参数以仅保存该级别（`--processing debayered`）。当您查询 `all` 时，不适用于特定凸轮的级别会被跳过（并予以报告），不会报错 ——未连接或未校准的摄像头仍会获得 `raw` / `debayered` / `preview`。

对于任何反射率帧，DAQ 实际使用的下行读数会被 写入图像旁边的 **`.daq`** 辅助文件中（以便日后可重新处理该捕获数据），并在 `DAQ:` 行中报告。

### 捕获文件夹的结构

每种导出类型都会存放在 `-o` 下的 **独立子文件夹** 中， 因此多级捕获中绝不会混淆不同类型：

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` 表示捕获时间戳，`<serial>` 表示相机序列号， 因此，一个同步组内的
所有相机都共享同一时间戳。**请注意以下一个不对称之处：** `display` 层级的文件存储在名为
`preview/` 的文件夹中，而文件本身则保留 `_display` 作为后缀——仅该层级中
文件夹名称与后缀不一致。未知层级 将回退到以自身名称命名的文件夹中；若子文件夹
无法创建，文件将写入输出根目录而非丢失。

**重新处理“captures”文件夹：**将 `chloros-cli process` 指向**“captures”根目录**
（`output/`）。`process`通常仅导入您指定的文件夹，但当该文件夹中没有
图像且包含子文件夹时，它会自动向下遍历——因此根目录的层级子文件夹以及
根目录下的`.daq` 都会一次性被全部导入。捕获的每一层都会作为单张图像导入，
其他层级则作为模式提供，而非每层生成一张图像。

直接为**层级子文件夹**命名 （例如 `output/raw/`）同样有效。这样做会保留根目录
`.daq`，因此当您从 `raw/` 重新导出辐射测量
产品时，请将DAQ读取数据复制或指向至该位置——否则时间戳匹配将无法找到对应的文件。

**处理始终从`raw`开始。**在每次采集中，原始帧是管道的源文件；
`debayered`、`radiance`、`reflectance` 和 `preview` 作为可视化模式存在，但绝不会
被回传至处理管道。 重新处理衍生产品将重新应用已经烘焙到其像素中的暗角、CCM 和
辐射度计算，因此 Chloros 会拒绝处理，而非
进行重复处理。 有两点后果值得注意：

- `index/` 和 `composite/` 的渲染结果**绝不会**被处理。它们是输出文件，而非捕获文件——
  NDVI的LUT渲染结果在辐射度方面没有实质性的解释。
- **未**包含`raw`（例如`array-capture --processing reflectance`）而导出的“captures”文件夹
  没有有效的管道源。这些捕获文件可以正常导入和显示，但 `process` 会跳过
  它们并提示如下：

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  如果您确实需要将衍生产品推入 ——一个启用了
  `demosaic`的集线器会话，或一个旧版文件夹——`--input-level {raw,debayered,processed}`将强制指定
  入口点并覆盖跳过设置。该标志是刻意设计的应急通道； `auto`（默认值）
  绝不会处理没有原始数据的捕获数据。

### 混合滤波器阵列中的跳过捕获

当您在一个阵列中混合使用RGB和多光谱相机时，`array-capture --processing radiance`（或`reflectance`）会保存多光谱帧，并**跳过**RGB相机——对于宽带传感器而言，基于拜耳阵列的辐射度数据没有实际意义。 CLI会打印每个已保存的文件（及其导出级别）， 每次写入的`.daq`文件，以及每次跳过的操作都会明确记录，因此文件数量并不令人意外：

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

跳过原因标记遵循`<level>-not-applicable-to-rgb-cam`的模式。反射反射数据也可通过 `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` 进行跳过，当波段主要位于 DAQ 光传感器辐射校准范围之外时，则通过 `dls-uncalibrated-band-<nm>` 进行跳过 （~374–974 nm）时，可使用 `dls-uncalibrated-band-<nm>` 进行跳过——在发货 SKU 中，仅 F988 支持此路径，其支持的工作流为“反射率面板”工作流。

使用 `--processing debayered`（或 `display`）可包含所有摄像头（无论滤光片类型），或使用默认的 `all`，可一次性获取每台相机所有适用的电平。

---

## 捕获模式、记录器与离线重处理

这些功能均基于**持久数组**运行 上运行（请先运行 `array-connect`）。它们与 GUI 捕获面板功能一致。

### `array-capture` 模式

`array-capture` 是一条包含四种快门模式及一组导出开关的单命令：

| 模式 | 标志 | 行为 |
| --- | --- | --- |
| **单次** *(默认)* | (无) | 捕获一个同步捕获组，然后退出。 |
| **连续** | `--continuous` | 连续执行，直至遇到 `Ctrl+C`、`--count N` 或 `--duration S`。 |
| **间隔** | `--interval S` | 每 `S` 秒执行一次扫描 （从每次扫描开始计时），边界范围相同。 |
| **最快** | `--fastest` | 仅原始数据 + 分配的DAQ读数 + 组合指数合成数据；跳过辐射度/反射率/显示运算，从而加快帧渲染速度。隐含 `--processing raw --force-daq`。稍后将保存的 `.daq` 重新处理为校准产品。 |

导出选项 （可与任何模式结合使用；均共享同一 GUI/SDK 端点）：

| 标志 | 效果 |
| --- | --- |
| `--processing LEVEL` | 单级导出，或 `all`（默认）。 |
| `--levels L1,L2,…` | 导出类型的显式子集（例如 `raw,radiance,reflectance`）；**覆盖 `--processing`**。 |
| `--aligned` / `--no-aligned` | 将每个成员的非原始导出映射到数组的 [对齐配置文件](#对齐)（协同对齐）。原始数据保持不变，但会在元数据中携带变换信息。如果数组没有对齐配置文件，则回退为未对齐（并发出警告）。 |
| `--index` / `--no-index` | 保存/跳过已配置的每台相机植被指数叠加层。默认：渲染该叠加层。 |
| `--force-daq` | 即使所选层级不需要（例如仅抓取原始数据），也应将分配的 DAQ/DLS 读数保存为 `.daq` 侧载文件，以便日后可将帧重新处理为反射率/指数。 |
| `--smart` | 在触发前等待所有摄像头的自动曝光（AE）稳定（参见 [智能自动曝光 / 智能捕获](#smart-ae--smart-capture))。 |
| `--compression {deflate,none}` | TIFF 像素压缩。`deflate` （默认）= 无损 zlib L1 + 水平预测器，每帧全分辨率数据约 4.1 MB；`none` = 未压缩，写入速度约快 5 倍，每帧约 6.3 MB — 当磁盘空间允许时，用于实现最大持续写入速率。两种模式均为无损压缩，导入时显示效果完全一致。 |

> **单次写入 TIFF + 持续速率模式。**捕获数据通过**单次**TIFF文件写入过程完成，包含像素数据 + XMP + IFD0 制造商/型号（基于全分辨率 Mono12 测试：压缩模式 36 毫秒 / 无压缩模式 6.5 毫秒，而旧版“先写入再用 ExifTool 重写”方式需约 148 毫秒）； 仅剩的 ExifTool 处理（EXIF 子 IFD 优化）在异步后台工作线程中运行，即使该线程从未执行，帧也已完成并可直接导入。请注意，DEFLATE 压缩会占用 Python GIL，因此压缩写入操作**不会**在各相机写入线程间并行化——以 8 台相机全全分辨率拍摄（以传感器速率进行，约 10.4 fps）需要同时具备 `--compression none`**和** NVMe 级硬盘（持续写入速率约 500 MB/s）。 该参数在 `POST /api/camera/array/capture` 中以 `compression` 的形式提供。

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — 视频/GIF 组合索引（监控级）

将**实时组合索引视图**中显示的内容录制到 `.avi`（以及可选的 `.gif`）。由于该功能会从实时复合流中提取数据，因此组合流必须处于打开状态 （例如，该数组正在 GUI 中被预览）才能接收帧数据。它每 2 秒轮询一次进度，并在 `--duration`、 `Ctrl+C`，或当录像机自动结束时。

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--array-id ID` | 仅数组 | 目标数组 （若仅连接一个则省略）。 |
| `-o, --output DIR` | `output` | 输出目录（后端本地）。 |
| `--fps F` | `10` | 录制帧率。 |
| `--duration S` | 直到按下 Ctrl+C | `S` 秒后自动停止。 |
| `--gif` | 关闭 | 同时写入动画 GIF。 |
| `--gif-only` | 关闭 | 仅生成 GIF 文件（不生成 `.avi`）。 |

### `array-burst` — 原始拜耳高帧率连拍（分析级）

直接读取抓取循环的同步组缓冲区 — **无需校准链、无需 exiftool、无需实时预览** — 因此可达到相机的全帧抓取速率。写入原始帧 + 每帧清单 + 在 `<output>/bursts/<base>/` 下，针对每个不同的 DLS 读数写入一个 `.daq`。离线重新处理（下一条命令），或传递 `--build` 文件，以便在停止时立即进行处理。

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--array-id ID` | 仅数组 | 目标数组。 |
| `-o, --output DIR` | `output` | 输出目录（突发数据存入 `<DIR>/bursts/<base>/`）。 |
| `--duration S` | 直到按下 Ctrl+C | 在 `S` 秒后自动停止。 |
| `--max-frames N` | 无限制 | 处理 `N` 个原始帧后自动停止。 |
| `--build` | 关闭 | 停止后，立即重新处理该帧序列 （与 `array-build-video` 相同）。 |
| `--products …` | `combined:index` | 配合 `--build` 使用：指定要构建的视频（见下文）。 |
| `--fps F` | `10` | 配合 `--build` 使用：输出视频的帧率。 |
| `--save-tiffs` | 关闭 | 启用 `--build`：同时按帧校准的 TIFF 文件。 |
| `--gif` | 关闭 | 启用 `--build`：同时写入动画 GIF 文件。 |

### `array-build-video` — 离线重新处理已保存的连拍数据

将每个原始帧与最近的已保存 `.daq` 读数进行时间匹配，并将其通过**与导入管道相同的辐射度/反射率/折射率处理链**进行处理， 从而渲染一个或多个视频。

`--products` 是由 `kind:level` 项组成的逗号分隔列表，其中 `kind` ∈ `per_cam` | `combined`，且 `level` ∈ `radiance` | `reflectance` | `index`。一个单独的 `level`（不包括 `kind:`）默认值为 `per_cam`。默认值为 `combined:index`。

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| 标志 | 默认值 | 说明 |
| --- | --- | --- |
| `--burst-dir DIR` | （必填） | 突发数据文件夹的路径（`…/bursts/<base>/`）。 |
| `--products …` | `combined:index` | `kind:level` 列表， 如上所述。 |
| `--fps F` | `10` | 输出视频帧率。 |
| `--save-tiffs` | 关闭 | 同时保存每帧校准后的 TIFF 文件。 |
| `--gif` | 关闭 | 同时写入动画 GIF 文件。 |

> **选择合适的录制器。** `array-record` 是 *监控级*——它捕获显示器上呈现的实时合成图像，需要保持数据流打开。`array-burst` → `array-build-video` 是 *分析级*——它以全速率保存原始传感器数据，并在后续重建校准后的辐射度/反射率/指数视频，无需实时预览。

### 单色（M3M）单波段相机

**M3M**系列是拜耳阵列**M3C**的单色版本：每台相机配备一个窄带干涉滤光片 （例如 `M3M-<lens>-F<wavelength>`、`M3M-L87-F685`），因此传感器可输出**单个灰度波段**，不包含拜耳马赛克。 无需进行去马赛克处理，无需消除通道间串扰，也无需设置白平衡——整个RGB显示颜色处理流程在此完全不适用。

这对CLI意味着：

- **`lattice white-balance`、`lattice color-profile`、`lattice color`**会检测到单色摄像头，并**跳过该设备并显示一行提示信息**，而不是推送毫无意义的设置。 在同一会话中，它们仍能正常运行于RGB或Bayer M3C 摄像头时仍能正常运行。
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** 依然有效——辐射度和反射度是 *按波段* 划分的辐射度 图，且针对单个波段定义明确。单色帧携带**单位**传感器响应矩阵（无3×3解混），因此该平面在校准计算中保持不变。
- **单台单色相机无法生成植被指数。**NDVI / NDRE 等公式至少需要两个波段（例如 Red + NIR）。若要通过单色硬件生成植被指数，需将**多台**不同波长的 M3M 相机对准目标，将其对齐组合成一个多波段堆栈，然后对*该堆栈*进行指数计算：

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` 符号必须与预设的通道名称**完全**一致 （区分大小写；NDVI 对应的小写形式为 `red`、`nir` —— 参见 `--list-presets`），且波段侧名称对应对齐堆栈中的某个波段 （离线模式也接受以 0 为起点的频段索引，例如 `--channel red=0 --channel nir=1`）。

整个堆栈中的区分标志是模型字符串中的 `M3M` 标记（该标记绝不会出现在 `M3C` 字符串中），并在 GUI/SDK 中显示为 `is_mono`。

---

## 主机网卡设置与调优（LATTICE 阵列）

LATTICE 摄像机通过主机的以太网适配器传输 GVSP，因此对于多摄像机阵列，适配器的 **驱动程序**和**接收环大小** 与链路速率同样重要。错误的设置会在“阵列设置”面板中以 `FRAMES WILL DROP` / `Reduce ROI to enable` 告警的形式显示 （以及在 `lattice network-analysis` / SDK 的 `analyze_array_network()` 中）出现 `rtump64x64.inf` 故障，即使摄像机本身运行正常。

### USB 10GbE 适配器 — Realtek RTL8157（“Realtek USB 10GbE 系列控制器”）

| 项目 | 必需值 | 为何重要 |
| --- | --- | --- |
| **驱动程序版本**|**≥ v10.67（2026年1月）**，INF `rtump64x64sta.inf` | 旧版**2016**驱动程序（v10.65，`rtump64x64.inf`）在关机/重启/休眠时无法正确处理断电及**`DRIVER_POWER_STATE_FAILURE` (蓝屏 `0x9F`)**。系统转换过程会卡死（约5分钟超时），用户被迫强制关机，而反复发生的不正常关机将**导致WMI存储库损坏**（PowerShell/工具开始因 `Invalid class` 报错而失效），并在下次启动时**导致 USB 堆栈卡死**（网卡无法启用；USB 设备停止枚举）。在依赖干净重启之前，请先从 realtek.com（或 USB 适配器厂商）获取更新。 |
| **接收缓冲区**— 关键词 `ReceiveBufferLen` |**256**（驱动程序最大值） | 网卡的 RX 环。驱动程序默认值**32**仅留下约 0.26 MB 的可用环——对于多摄像头突发传输而言远不够用——因此数组面板会报告 `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` 并阻止连接。设置为**256**时，环容量较大（**在实验室 10GbE 主机上实测约 13.5 MB**）， 为接收管道提供了处理多机位 GVSP 突发数据的实际余量。（特定配置是否能实际 *建立连接* 由两项检查决定——**考虑缓冲区耗尽情况的**接入检查和**聚合超过订阅量**检查——而非单纯的突发流量与环形缓冲区容量的比较；参见 [阵列 fps 与突发模型](#array-fps--burst-model)。) |
| **接收 URB**— 关键字 `PendingReceives` |**64** (最大值) | 传输中的 USB 请求块；需与接收缓冲区配合使用以吸收突发流量。 |
| **巨型帧** — 关键字 `*JumboPacket` | **9014** | 用于 9000 字节的 GVSP 数据包 （每帧数据包数量比1500字节帧少6倍）。 |

> ⚠️ **网络适配器驱动程序更新会将这些高级属性重置为默认值。**更新或更换网卡驱动程序后，请**重新应用** `ReceiveBufferLen=256` 和 `PendingReceives=64`，否则即使“硬件未发生任何变化”，阵列面板仍会再次进入锁定状态。这是原本正常运行的设备突然拒绝 连接的首要原因。

请通过**管理员权限**的 PowerShell 进行应用 （替换为您的适配器名称，例如 `"Ethernet 5"`）：

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` 适用于 USB 10GbE 适配器。** 现在它会检测网卡类型并调整正确的接收环关键字：对于 PCIe 网卡（如 Intel I219 等），`*ReceiveBuffers`→2048；对于 Realtek **USB** 10GbE 控制器（不暴露 `*ReceiveBuffers`），则 `ReceiveBufferLen`→256 + `PendingReceives`→64 用于 Realtek **USB** 10GbE 控制器（该控制器不暴露 `*ReceiveBuffers`）。 目标值会被限制在各驱动程序报告的最大值（`NumericParameterMaxValue`）内，因此绝不会写入超出范围的值。 请在**提升权限**的终端中运行；与任何基于注册表的调整一样，该更改需在网卡重启或系统重启后生效。上述手动 `Set-NetAdapterAdvancedProperty` 命令仍然是一个不错的选择——它们可在不重启的情况下实时 （重新绑定适配器）。

### 网络基础知识（所有 LATTICE 链路）

- **地址分配：**链路本地 `169.254.0.0/16`（GigE Vision LLA）。主机采用静态 `169.254.x.x/16`；摄像机和数据采集卡-E 设备在同一范围内自动分配。无需 DHCP/网关。
- **数据包大小：**优先使用巨型数据包（9000），但应让自动探测功能自行确定——该功能在每次连接时都会重新测量，并且已通过 GVSP探测，因此只要线路实际支持，数据包最终会以巨包形式传输。仅在您比探测器更了解情况时才通过 `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` 进行固定设置，且优先采用按命令设置而非永久设置： 该设置会绕过探测，因此如果路径实际上无法承载 9000，则使用 `SC_ERR_TIMEOUT -1011` 时**每次**捕获都会超时（参见 [环境变量](#environment-variables)）。
- **RX环带宽随`ReceiveBufferLen`调整：**在默认`32`设置下，可用环带宽约为0.26 MB （对于任何多摄像头突发传输而言都太小）；在最大 `256` 设置下，其容量较大（在实验室 10GbE 主机上测得约 13.5 MB），提供了真正的余量。 配置能否建立连接，取决于“考虑带宽消耗的接入检查”**以及**下文所述的聚合超订阅检查——而非单纯比较原始突发速率与环带宽。

### 阵列帧率与突发模型

如何解读“阵列设置”面板（以及 `lattice analyze-array` / SDK 的 `analyze_array_network`）：

- **突发数据按每台摄像头的实际像素格式进行累加。**单色**M3M**相机输出**Mono12（2 位/像素）**；**M3C**拜耳相机输出 8 位或 12 位（TRI032S 即使被请求输出拜耳 RG8，也会默认输出拜耳 RG12）。 因此，一个由 4 台相机组成的全分辨率帧，**若均为 8 位则约为 12.6 MB，但若包含三台 12 位单色相机则约为 25 MB**。该投影功能会根据相机型号（身份缓存）解析每台相机的格式，因此数据包内容与线缆实际传输的数据相符——而非采用“统一的 BayerRG8”这一假设。
- **无论铭牌标注如何，USB 以太网适配器的传输速率上限均为 200 MB/s。** 将链路速率转换为持续传输速率的效率表源自 PCIe； USB 网卡虽标称其 *以太网* 链路速率，但实际受限于 USB 总线及其驱动程序。一款 USB 10GbE 适配器曾显示约 1063 MB/s 的“持续传输速率”——该数值从未经过实际测试——而由此产生的速率调控导致 6–18 % 帧，但仍报告正常的帧率目标值。目前，通过 USB 连接的网卡速度上限为 **200 MB/s**（该限制由总线决定，因此不会随标称速率线性增长；USB 1 GbE 适配器可达到约 80 MB/s，且不受此影响）。能力记录中的 `wire_ceiling_source` 以文字形式明确说明了这一点，而 `nic_is_usb` 标志会对此进行标记。无论哪种情况，均可通过 `--wire-ceiling-mbps` 进行覆盖。
- **通量控制是基于数据流的，而非基于整个突发与环形缓冲区的比较。** 同时发生的突发数据只需适应 *瞬态积压* = `max(0, Σ per-cam arrival − host drain) × emit_window`，而非整个突发数据。 在主机快/摄像头慢的架构中（**PCIe**10G主机 + 4× 1 GbE摄像头：到达速率 ≈ 320 MB/s，排出速率 ≈ 1063 MB/s），主机的排出速度快于摄像头的填充速度， 积压量 ≈ 0，因此全分辨率模拟发射会**放行**，尽管 25 MB 的突发数据量超过了 13.5 MB 的环容量。若将这四台摄像头连接到**USB**10GbE 适配器后，数据排出速率为 200 MB/s， 而非 1063 ——数据到达速度超过了数据排出速度，且丢失的数据表现为帧损坏，而非帧率降低。在 1 GbE 主机上，相机 31.25 MB/s 的 DLThr 下限导致数据到达速度超过数据排出速度 → 系统会正确地**阻止**（针对*此类*阻塞情况，请缩小ROI或使用≥2的像素合并）。允许通过的条件是**两个**连接门中的一个——另一个是下方的订阅不足检查。
- **预测帧率是保守的串行读取上限。**主机抓取循环目前以**串行**方式拉取每台摄像机的缓冲区 （每个摄像头约一个发射窗口），因此该周期受 `max(readout+emit, N × emit)` 限制，且每个摄像头的发射速率被限制在摄像头的**访问链路**带宽内（1 GbE ≈ 80 MB/s）， 而非主机的上行链路。对于 4 台全分辨率 12 位摄像头的阵列，帧率为**~2.8 fps**， 与实测的约2.7–3.0 fps相符。帧率被刻意设计为**与曝光时间无关**，因此在光线较暗的场景中，随着曝光时间延长，实际帧率可能会略微低于上限。串行读取才是真正的帧率限制因素；若将其并行化，上限将提升至接近单帧发射速率。
- **聚合过载订阅是硬性连接阻塞点。**每台摄像头的带宽分配下限为**8 MB/s**（`ARRAY_PER_CAM_FLOOR_BPS`）， 因此一旦下限被锁定，聚合需求（`per_cam × N`）就可能超过**防碰撞安全线速上限**（`sustained × sim_emit_factor`）。1 GbE 环境下实际全分辨率上限为：**6 台摄像头（MTU 为 1500）或 9 台（启用巨型帧）**。该上限仅由线路带宽和下限共同决定——它**与帧大小无关**，因此**帧合并和缩小 ROI 区域均无助于解决问题**（这些措施仅降低了每个 *帧*的字节数，而非GevSCPD控制的每*秒*字节数）；唯一的解决方法是减少摄像头数量、端到端启用巨型帧，或使用更快的网卡。症状表现为GVSP数据包丢失，而非平滑的帧率降低，因此`analyze-array`会将可实现的帧率（fps）数值并输出 `**OVER-SUBSCRIBED**`，而当分辨率被锁定时，`array-connect` 会**拒绝连接**（否则“走下”机制会将帧进行合并处理，但这同样无法消除此类阻塞）。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` 将“拒绝连接”降级为针对基准测试的醒目警告——参见 [环境变量](#environment-variables)。

### 数组健康状况——哪个子系统正在丢失帧

已连接数组的 `GET /api/camera/array/<array_id>/capability` 携带一个实时
`health` 数据块，该数据块基于滚动 **10 秒** 时间窗口进行重新评估。它将帧丢失
拆分为两种需要采取相反修复措施的原因，而不是报告一个既不指明原因的“不完整”
率：

| 字段 | 含义 | 涉及子系统 |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct`（按串行端口） | 帧**已到达但结构损坏**— GVSP 数据包丢失。 |**网络**：带宽预算、速率控制、网卡接收环、MTU |
| `never_arrived_rate_pct`（按串行号） | 帧**从未到达**——相机未触发，或未发送任何数据。 |**触发/同步**： M8 电缆、`--line`、`TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | 每台摄像机的最差传输速率。 | — |
| `per_cam_rate_pct` | 每台相机的综合不完整率（两种原因合计）。 | — |
| `stable_for_seconds` | 每台摄像头处于 0.01% 以下的时间长度。 | — |

当该值超过 5% 时，后端会记录一条 `[array-health <id>] WARN` 日志行，其中会注明分段情况——在
首次超标时、在严重性等级区间发生变化时， 在该状态持续期间每分钟记录一次，以及在
状态消除时记录一次。 受损的半部分会在每个摄像头首次触发时输出 `[gvsp-corrupt <SN>]` 并
注明原因，随后每 60 秒汇总一次。每次评估结果仍会写入后端日志文件；
计数器会随着每个缓冲区更新，无论输出内容如何。

同一条记录还会报告整个分配 所关联的编号：

| 字段 | 含义 |
| --- | --- |
| `wire_ceiling_mbps` | 主机当前生效的持续带宽配额，MB/s。 |
| `wire_ceiling_source` | 该数值来源的文字说明——例如 `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` 或 `user override 120 MB/s (auto said 200)`。 |
| `wire_ceiling_is_user_set` | 当 `--wire-ceiling-mbps`（或 GUI 中的 **带宽预算** 字段）将其设置为 `true` 时。 |
| `nic_is_usb` | 对于 USB 以太网适配器，`true` —— 参见上文提到的 200 MB/s 上限。 |

**解读：** 当 `gvsp_corrupt_rate_pct` 不为零且 `never_arrived_rate_pct` 为 0 时
表示触发和电缆同步完美无缺，100% 的丢包发生在网络
路径上 — 降低 `--wire-ceiling-mbps`，然后重新连接。反向模式则表明问题出在
同步线缆或触发线上。

> **`--target-fps` 并非帧损坏的控制因素。** GevSCPD 时序在连接时仅写入
> 一次，因此降低触发率只会改变占空比，而不会改变
> 同时发射突发速率。经测量，将需求削减5倍未见改善；
> 将线速上限从240 MB/s降至200 MB/s后，同一测试平台的数据包损率从10.4%
> 降至0.00%。

> **TRI03S固件不支持**流自动缩减功能在 TRI032S 固件中不可用。** 运行中的阵列
> 无法自行修复此问题；请断开并重新连接，以便连接时选择器能够
> 根据新的上限重新规划。

### 症状 → 解决方法

| 症状（阵列设置 / 连接 / `analyze_array_network`） | 原因 | 解决方法 |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` 重置为 32 （通常发生在驱动程序更新后） | 将 `ReceiveBufferLen` 设为 256，将 `PendingReceives` 设为 64； 重新打开控制面板（如果后端缓存了旧的环大小，则重启后端） |
| 重启/关机卡死；随后出现 `Invalid class` WMI 错误， 网卡无法启用，USB 设备缺失 | 旧版 2016 Realtek USB 10GbE 驱动程序 → 蓝屏 `0x9F` → 强制关机 | 将适配器驱动程序更新至 ≥ v10.67（2026），然后重新-应用上述接收环设置 |
| 连接成功但返回低于原生分辨率 | Smart-prep 自动缩小帧大小以适应线缆 | 升级链路 / 接受缩减 / `--force-tier slip-emit-and-capture` |
| 阵列报告的目标帧率正常，但实际输出仅为其一小部分；`health.gvsp_corrupt_rate_pct` 不为零，`never_arrived_rate_pct` 为 0 | 主机推断出的 线速预算值高于其实际可维持的速率（常见于 USB 以太网适配器、带宽较窄的 PCIe 通道或共享互连结构） | 采用较低的 `--wire-ceiling-mbps` 值重新连接，并重新检查健康状态块。 **非** `--target-fps` — GevSCPD 速率控制在连接时已固定 |
| 已发布组中缺少摄像头；`health.never_arrived_rate_pct` 不为零，`gvsp_corrupt_rate_pct` 0 | 触发/同步路径——摄像头未触发，非网络问题 | 检查 M8同步线缆及`--line`；确认所有成员均已启用（`TriggerMode=On`） |
| `**OVER-SUBSCRIBED**` / `Wire budget` 值在 `analyze-array` 中超限，或出现带固定解决方式的连接拒绝（`array over-subscribes the wire`） | 每台摄像头的聚合需求（8 MB/s 下限 × N 台摄像头）超过了 防冲突线速上限——1 GbE @1500 MTU 条件下 6 台全分辨率摄像头，使用巨型帧时为 9 台 | 减少摄像头数量、端到端使用巨型帧，或使用更快的网卡。 **ROI/像素合并无法解决问题**（该上限与帧大小无关）。`CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` 在测试平台上会覆盖此限制（接受数据包丢失） |

---

## `chloros-cli daq`

Spectral-sensor命令。分为两类：
- **`pool-*`**— 通过后端持久化池驱动传感器的轻量级HTTP客户端。**这是受支持的路径，也是出厂版CLI中唯一存在的路径。** 后端拥有传输通道，因此图形界面、CLI和SDK脚本均可共享一个活动句柄，而无需争夺串行端口。
- **其余所有情况**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`、`serve`、`ws`、`udp`、 `mqtt`、`reflectance`、`login`、`logout`、`status`） — 直接硬件访问，为完整起见，下文将对此进行说明。这些功能需要 `daq` Python 软件包，该软件包**未包含在任何已发布的构建产物中**：编译后的 CLI 不包含该软件包 （`scripts/Build-CLI.ps1` 会设置 `--nofollow-import-to=daq`，而传输层 `pyserial` / `bleak` / `zeroconf` 传输也包含该文件)，且 PyPI 上的 SDK 包中同样不包含该文件。这些文件仅能从源代码检出后运行， 因此请将其视为MAPIR内部的开发路径，而非可供外部使用的方案。
- **`discover` / `list`** 兼具两者特性：它们是直接硬件命令，但已发布的构建版本会回退到 `pool-discover`，由后端执行扫描。因此扫描功能在任何环境下均可正常工作——这一点至关重要，因为这是获取 DAQ-M 设备 BLE MAC 地址的唯一途径。

> **`chloros-cli daq --help`**（以及 `-h` / `help`）列出了 `pool-*` 的子命令——帮助信息被有意路由至池客户端，以便反映 实际执行的命令。若在正式发布版本中调用直接硬件子命令，程序会以明确的错误信息退出，指出缺失的包并引导您返回`pool-*`；绝不会发生无提示的静默失败。 （`discover` / `list` 是例外——它们会重定向至 `pool-discover` 并正常运行。）
>
> **客户所需的一切功能均可通过 `pool-*` 实现**——连接、流式传输、记录经过校准的 `.daq` 文件，以及切换电容曲线。 该数据采集设备还可通过 Python 配合 `chloros_sdk.connect_daq_sensor()` 进行控制，该命令使用相同的池化路径。

### DAQ 传感器首次连接工作流

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` 参考

| 子命令 | 用途 |
| --- | --- |
| `daq pool-connect` (智能检测) | 在后端池中打开一个传感器。 |
| `daq pool-connect --port PORT` | 在特定串行端口上进行 DAQ-U。 |
| `daq pool-connect --ble` | 通过 BLE 进行 DAQ-M，自动扫描 MAC 地址。 |
| `daq pool-connect --mac MAC` | 通过 BLE 连接至已知 MAC 地址的 DAQ-M（隐含 `--ble`）。 |
| `daq pool-connect --eth-host HOST` | 通过以太网连接至已知主机的 DAQ-E。 |
| `daq pool-connect --eth` | 通过以太网连接的 DAQ-E，主机自动发现（mDNS + ARP 备用方案；在 Windows 和 Linux 上，即使 ARP 缓存为空也能正常工作）。 |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | 调整积分窗口 / AE状态。 |
| `daq pool-connect --no-stream` | 已连接但尚未开始流式传输 （通过 `pool-stream --start` 恢复）。 |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | 速率校正配置文件。后端默认值为 `sunshine_cosine`。 |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | 扫描每个传输层，查找可连接但暂不连接的传感器。**这是查找 DAQ-M 的 BLE MAC 地址的方法。** `daq discover` / `daq list` 路径在出厂版本中会自动跳转至此处。传感器池中已打开的传感器不会被列出 ——已连接的 DAQ-M 会停止广播——因此请使用 `pool-list` 查询这些传感器。 |
| `daq pool-list` | 显示后端池中的所有传感器。 |
| `daq pool-disconnect --sensor-id ID [--all]` | 释放。 |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | 最近的 N 个频谱帧。 |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | 恢复 / 暂停流式传输。 |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | 开始/停止 .daq 录制。 |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | 在运行时切换电容校正配置文件。 |

### 直接硬件子命令（仅限源代码检出——未包含在发布版本中）

> 此处列出仅为完整起见。这些命令需要 `daq` Python 软件包以及 `pyserial` / `bleak` / `zeroconf`，上述包均未包含在编译版的 CLI 或 PyPI SDK 中——它们仅可在 MAPIR 源代码检出环境中运行。**如果您使用的是已发布的 Chloros 构建版本，请改用上述 `pool-*` 命令**；这些命令涵盖连接、流传输、录制和捕获源选择功能。

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

打开、连接并运行已保存的Chloros项目 （即包含 `cameras.json`、`sensors.json` 和 `project.json` 的文件夹）。所有操作均通过后端进行路由，因此图形界面和 `CLI` 命令生成的硬件状态完全一致。

### 子命令参考

| 子命令 | 用途 |
| --- | --- |
| `project open PATH` | 打印项目的设备清单（摄像头、阵列、传感器）。 |
| `project devices PATH [--reconnect]` | 列出或重新运行设备发现。 |
| `project connect PATH [--cameras-only] [--sensors-only]` | 连接所有已保存的摄像头/阵列/传感器。 |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 从指定名称的摄像头或阵列进行单帧捕获。 |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | 从指定摄像头或阵列进行 N 帧连拍（`-n/--count` 默认值为 5；`-i/--interval` 帧间间隔（秒），默认值为 0）。阵列连拍会-重复同步组（过期监视器），因此部分循环的相机阵列无法返回同一帧的 N 个副本；打印每次迭代的结果。 |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | 通过后台任务将数据流写入磁盘。`--poll-interval` = `/stats`轮询之间的秒数（默认 2.0）。 |
| `project sensor read PATH NAME [--json]` | 最新光谱帧。 |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | 记录 .daq 文件。 |
| `project run PATH RECIPE.yaml` | 执行 YAML/JSON 捕获配方。`--dry-run` 进行验证而不执行。 |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | 计算数组的对齐参数 — 参见 [下方的标志表](#project-align-calibrate-options)。 |
| `project align status PATH NAME [--json]` | 打印当前对齐配置文件。 |
| `project align clear PATH NAME` | 清除缓存的配置文件。 |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | 微调一个从属节点的变换。 |
| `project align export PATH NAME --to FILE` | 将配置文件保存至JSON。 |
| `project align import PATH NAME --from FILE [--no-validate]` | 加载已保存的配置文件。 |

#### `project align calibrate` 选项

| 标志 | 默认值 | 描述 |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | 对齐方法。**这些拼写与 `lattice align-calibrate`** 不同，后者接受简写形式 `orb` / `akaze` / `phase`；这两个命令 在此标志下不可互换。 |
| `--model {translation, rigid, affine, homography}` | `affine` | 变换模型以适配。 |
| `--frames N` | `1` | 将帧快照同步到平均值。 |
| `--reference SN` | 主摄像机 | 参考摄像机序列号；其余所有成员均被变形到该主摄像机上。 |
| `--max-features N` | `5000` | ORB特征计数上限。 |
| `--ratio-threshold F` | `0.75` | Lowe 比率检验。 |
| `--ransac-threshold-px F` | `3.0` | RANSAC 内点阈值。 |
| `--min-matches N` | `15` | **质量阈值** — 若符合的内点数量低于此值，则拒绝该解。 |
| `--max-reproj-err-px F` | `4.0` | **质量阈值** — 若重投影均方根误差超过此值，则拒绝该求解结果。 |
| `--checkerboard RxC` | — | `--method checkerboard` 的板级几何信息，例如 `9x6`。 |
| `--name PROFILE` | 空 | 嵌入到已保存的JSON中的轮廓名称。**并非数组名称** — 即位置参数 `NAME`。 |

这两个质量门是校准虽然成功求解却仍
拒绝保存的原因：若配置文件未能通过其中任何一项检查，将导致后续每次
捕获都出现无提示的定位偏差，因此系统会拒绝保存而非将其保留。

### 示例

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### 配方 DSL

`project run RECIPE.yaml` 接受 YAML 或 JSON 格式的文件，用于描述一系列操作：

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

支持的操作：`apply`、`wait`、`capture`、`stream`、`burst`、`sensor`。 `burst` 操作需要 `name`（必填）、`count`（默认值为 5）、`interval` （秒，默认值为 0）、`output`、`format` 以及 `settings`（与-camera设置形式，如`apply`）；数组连拍使用与`project burst`相同的“新同步组”看门狗。

运行命令：

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## 环境变量

| 变量 | 作用 |
| --- | --- |
| `CHLOROS_BACKEND_URL` | 覆盖后端URL（默认值为 `http://127.0.0.1:5000`）—— **仅由 `lattice`、`project`、 以及 `daq pool-*` 命令族支持此功能。** 核心命令（`process`、 `login`、`logout`、`status`、`export-status`、`time-sync`、 `selftest`) 引脚 `http://127.0.0.1:<port>` 并忽略此变量（IPv4 字面量绕过了 Windows `localhost`→`::1` 约 次/请求的惩罚），因此它们始终将目标指向本地机器。|
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` 将数组超订阅连接拒绝（每个 CAM 的聚合需求 &gt; `pin_resolution` 下的碰撞安全带宽上限）降级为响亮的警告-并继续处理，接受 GVSP 数据包丢失。仅限基准测试使用——参见 [阵列 fps 与突发模型](#array-fps--burst-model)。 |
| `CHLOROS_CLI_MODE` | 由CLI本身设置； 指示后端启用并行处理。 |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` 跳过 GVSP 备用探测（仅 ICMP 结果）。**这会禁用巨包，它仅仅是抑制日志输出**——摄像头在每条路径上仅对大小不超过1500的DF ping做出响应，因此该探测是检测巨包的唯一途径。每次连接可为每台摄像头节省约1秒；如果网络*能够*传输巨包，则开销约为线速上限的1.45倍。 设置此选项时，SDK会发出警告。 |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | 将GVSP数据包大小固定为N字节；完全跳过探测。建议使用命令（`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`）而非永久设置：固定大小将导致无法适应前端网络，且在无法承载巨包的路径上将大小固定为9000会导致**每次**捕获在`SC_ERR_TIMEOUT -1011`。 |
| `TMPDIR` (Linux) | 覆盖 Nuitka 的单文件提取目录。若存在 CLI，则自动使用 `/mnt/ssd/tmp`。 |

---

## 退出代码

| 代码 | 含义 |
| --- | --- |
| `0` | 成功。 |
| `1` | 通用错误（大多数子命令错误）。 |
| `2` | 参数错误。 |
| `130` | 被 Ctrl+C 中断。 |

---

## 故障排除提示

- **“需要登录”** → 在本机上运行一次 `chloros-cli login EMAIL PASSWORD`。
- **“无法连接后端”** → 启动 Chloros 桌面应用程序，或直接运行后端二进制文件（`chloros-backend`），若为远程连接，请检查 `CHLOROS_BACKEND_URL`。
- **`lattice` 命令因“未找到 LATTICE 摄像头驱动程序”而失败** → 未安装 Arena SDK 运行时； CLI 随附的 `win32api` 已打包在 Windows 上，但 C 运行时是 GUI 安装程序的一部分。
- **数组连接/数组设置中显示“FRAMES WILL DROP”或“Reduce ROI to enable”** → 主机网卡的接收环大小过小（通常在网卡驱动程序更新后会重置为 32）。 请参阅 [主机网卡设置与调优](#host-nic-setup--tuning-lattice-arrays) — 设置 `ReceiveBufferLen=256`、`PendingReceives=64`。
- **系统在重启/关机时卡死，随后出现 WMI 错误 `Invalid class` / 网卡无法启用 / USB 驱动器缺失** → 过时的 USB 10GbE 适配器驱动程序导致 `DRIVER_POWER_STATE_FAILURE`（蓝屏 `0x9F`）。请更新适配器驱动程序 — 参见 [主机网卡设置与调优](#host-nic-setup--tuning-lattice-arrays)。
- **Jetson 交换分区警告** → 添加基于文件的交换分区；CLI会输出确切的 `fallocate` / `swapon` 命令。
- **缺少 DAQ 直接命令** → 预期情况：随附的 `chloros-cli` 刻意排除了 `daq` 包，因此仅包含 `pool-*`（PyPI 上的 SDK 也没有提供该包）。 请使用 `pool-*`， 该包通过后端驱动相同的传感器；或者使用来自 Python 的 `chloros_sdk.connect_daq_sensor()`。

---

## 参见

- [Python SDK 参考文档](sdk-reference.md) — 与所有 CLI 命令功能等效的编程接口。
- [DAQ 传感器指南](../daq/README.md) — 针对特定传感器的接线与校准说明。
- 在线文档：`https://mapir.gitbook.io/chloros/cli`
