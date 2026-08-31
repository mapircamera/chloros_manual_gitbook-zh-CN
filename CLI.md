# CLI：命令行

> **完整参考：**[CLI 参考](reference/cli-reference.md) 记录了**每个子命令的每个参数**，并针对 AI 助手进行了优化——将它的 URL 粘贴到您的助手中，并询问有效的命令：`https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **AI 工具使用提示：** 本手册的任意页面均可通过在其 URL 后追加 `.md` （例如 `https://mapir.gitbook.io/chloros/reference/cli-reference.md`），而 `https://mapir.gitbook.io/chloros/llms.txt` 可为大语言模型（LLM）提供整本手册的索引。

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->
## 什么是 CLI

`chloros-cli` 是与 Chloros 桌面应用程序所使用的同一处理引擎对应的命令行前端。 它是一个基于 Chloros 后端（位于 `127.0.0.1:5000` 上的本地服务器）的轻量级 HTTP 客户端——大多数命令会自动启动后端， 因此脚本只需调用一次 `chloros-cli process …` 即可。

它可在 **Windows 10/11 (x64)**和**Linux (x86_64，以及运行于 JetPack 6 上的 NVIDIA Jetson arm64)** 上运行， 可在任何终端中运行，无需图形界面。请使用以下命令验证安装：

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

命令家族一览：

* **处理与账户** — `process`、`login`、`logout`、`status`、 `export-status`、`language`（支持 38 种语言——详见 [支持的语言](supported-languages.md)）， `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (仅限 Linux/Jetson)
* **实时硬件** — `lattice`（LATTICE 相机控制，45+ 个子命令），`daq pool-*`（DAQ 光传感器），`time-sync` (PTP)
* **自动化** — `project`（无头模式下运行已保存的 Chloros 项目，包括 YAML 捕获配方）

值得了解的全局选项：`--port N`（后端端口，默认 `5000`）、`-v/--verbose`、 `--restart`（强制重启后端），`--backend-exe PATH`。 完整列表请参阅 [CLI 参考文档](reference/cli-reference.md)。

***

## 安装

在所有平台上，CLI **均包含在 Chloros 安装程序中** —— 没有单独的 CLI 下载包。 请从 [下载](download.md) 页面获取安装程序。

### Windows

安装程序将 CLI 放置在：

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

并将该文件夹添加到系统的 `PATH` 目录中 —— 安装完成后请**打开一个新的终端**，以便系统能识别更新后的 `PATH`。安装程序还会在安装根目录下放置启动脚本 （`Chloros_CLI.bat` / `Chloros_CLI.ps1`）放置在安装根目录下，并添加了一个**Chloros CLI** 开始菜单快捷方式，每个快捷方式都会打开一个终端，其中已预装好 `chloros-cli` 并可立即使用。

### Linux

请安装适用于您系统架构的 `.deb`：

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

这将安装 `chloros-cli` 至 `/usr/bin/chloros-cli` （当前版本为 `PATH`），并将后端升级至 `/usr/lib/chloros/chloros-backend`，同时安装 LATTICE 相机所需的 Arena SDK 运行时。 详情请参阅 [Linux 安装指南](linux/linux-installation.md)。

### 验证

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## 登录与授权

访问 CLI（以及 Python 和 SDK）需要 **付费的 Chloros+ 套餐**——任何付费套餐均包含此功能；免费套餐则不包含。该限制由后端在**服务器端**强制执行，而非由 CLI 二进制文件执行： 未登录状态下的调用将被以 `401 AUTH_REQUIRED` 状态码拒绝，而免费套餐下已登录状态的调用将被以 `403 PLAN_UPGRADE_REQUIRED` 状态码拒绝，无论该调用来自 `chloros-cli`、 SDK，还是自定义的 HTTP 客户端。请在 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) 处进行升级。**每台机器登录一次**：

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->
{% hint style="warning" %}
**包含特殊字符的密码**（`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` 会被 shell 篡改（CLI 会在 401 错误时检测到此问题并自动重试，但使用单引号可完全避免该问题）。
{% endhint %}

会话被缓存于 `~/.chloros/user_session.json` 中，并在套餐的宽限期内保持离线运行（月度套餐为 30 天，年度套餐则持续至到期）。 即使没有付费套餐，`chloros-cli status` 也能正常工作，因此拒绝的原因始终可见。

{% hint style="danger" %}
**要调度无界面任务？请先登录。**若在**无缓存会话**的情况下运行后端进程生成命令（`process`、`status`、 `export-status` 等）若在**无缓存会话**的情况下运行，不会快速失败——而是会切换到交互式 `Email:` / `Password:` 提示符，并通过标准输入（stdin）接收命令。 因此，无人值守的 cron 任务或 CI 步骤将**因等待输入而挂起**。在安排任何任务之前，请先在机器上运行一次 `chloros-cli login EMAIL 'PASSWORD'`。
{% endhint %}

***

## 您的首次处理运行

将 `process` 指向捕获文件夹——它会自动检测 Survey3（`.raw` + `.jpg`）、 LATTICE（`.tif`/`.tiff`）、`.dng`，或混合类型：

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

每个管道线程（检测、分析、处理、导出）都会实时显示进度流，成功运行后会报告写入的图像产品数量（`Image products written: N`）。

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### 输出文件的位置

`process` 将输出写入 **项目文件夹**，而非您的输入文件夹：

* 若未指定 `-o`：项目将创建在您的默认项目文件夹下（与图形用户界面共享；可通过 `get-project-folder` / `set-project-folder`，备用方案为 `~/Chloros Projects`)，若未指定名称，则由 `-n/--project-name` 或时间戳（`YYYYMMDD_HHMMSS`）命名。
* 若使用 `-o PATH`：该文件夹 **即** 为项目文件夹。 如果该文件夹中已存在 `project.json`，则会创建后缀为 `_1`/`_2`… 的子文件夹，而非覆盖原有文件。

在项目内部，产品会**按相机分类，然后按文件格式分类**：

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

LATTICE 的相机文件夹为 `LATT-<sensor>-<lens>-F<filter>`（与拍摄文件的 EXIF 中的 `Model` 对应），而 `<model>_<filter>` （例如 `Survey3N_RGN`）对应 Survey3。 格式文件夹的命名遵循 `--format` 的格式：`tiff16`、`tiff8`、`png8`、 `jpg8`，或 `tiff32` 对应 `TIFF (32-bit, Percent)`。

{% hint style="info" %}
**每个导出的产品都保留源文件的名称。**`capture_..._raw.tif` 的 Radiance 导出文件仍命名为 `capture_..._raw.tif` —— 它只是位于 `tiff32/Radiance_Images/` 目录下。**文件夹而非文件名用于标识生成文件**，因此应使用通配符匹配目录，而非 `*radiance*` 这样的后缀。
{% endhint %}

### 您实际会用到的选项

| 标志 | 默认值 | 功能说明 |
| --- | --- | --- |
| `-o, --output PATH` | 默认项目文件夹 | 项目文件夹位置（参见上文）。 |
| `-n, --project-name NAME` | 时间戳 | 项目名称。 |
| `--format FMT` | `TIFF (16-bit)` | 选自 `TIFF (16-bit)`、 `TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)` 中的一个。 |
| `--indices NAME [NAME ...]` | 无 | 待导出的植被指数（参见 [植被指数](#vegetation-indices)）。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = 神经网络去拜耳化，速度较慢，质量最高（Chloros+，NVIDIA GPU）。 |
| `--vignette / --no-vignette` | 启用 | 暗角校正。 |
| `--reflectance / --no-reflectance` | 启用 | 反射率校准；对于 LATTICE 而言，此选项同时用于切换反射率产品。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | 强制 LATTICE TIFF 文件的处理管道入口点。 |

关于其他所有内容——目标检测调整、PPK、曝光定位点、阵列对齐标志——请参阅[`process`部分的CLI参考文档](reference/cli-reference.md)中的[`process`部分]。

***

## 选择要导出的内容（LATTICE 产品）

LATTICE 处理会在 **单次处理中**扩展到所有适用产品。每个产品的四个开关**默认均处于开启状态**；请使用 `--no-` 表单关闭其中一个：

| 开关 | 产品 |
| --- | --- |
| `--debayered` | 线性去马赛克 → `Debayered_Images/` |
| `--preview` | 显示预览（白平衡 + 伽马校正；多光谱图像的伪彩色拉伸） → `Preview_Images/` |
| `--radiance` | float32 辐射度，W/m²/sr/nm → `Radiance_Images/`（始终为 `tiff32/`） |
| `--reflectance` | uint16 反射率，Pix4D 兼容 → `Reflectance_Calibrated_Images/` |

RGB 主摄像头仅输出去拜耳化后的数据 + 预览数据 —— 对于宽带传感器而言，各波段的辐射度/反射率没有实际意义，因此这些开关对它们而言是无操作的。 Survey3 `.raw` 忽略这些切换选项，并遵循标准的反射率/目标路径。

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`**（默认 `auto`）选择反射率参考：`auto` 生成一个通过质量保证（QA）检测的帧内 [校准目标](calibration-targets.md) 作为绝对参考，当无目标存在时，则回退至数据采集（DAQ）光传感器向下辐射分界值（ρ = π·L/E）；`target` 采用严格模式（不进行数据采集替换）； `daq` 以数据采集（DAQ）结果为准。可通过 `--target-reflectance-dir` 提供按单位测量的目标扫描数据。

{% hint style="info" %}
**读取反射率像素：**表示 ρ = 1.0 的 DN 值是**按光源计算的** — LATTICE 文件会在 XMP 中标记 `Chloros:PixelScale=32768`；Survey3 文件使用 65535（且不包含 `Chloros:*` 标签）。 请读取该标签并以此为基准进行除法运算，而非假设其为常数。详细信息及一个刻意设计的无比例尺边界情况，请参见 [CLI 参考文档](reference/cli-reference.md)。
{% endhint %}

**处理始终从 `raw` 开始。** 衍生产品（去拜耳化/辐射度/反射率导出）绝不会被重新输入到处理管道中——重新导入并处理它们会导致校准运算被重复应用，因此 Chloros 会跳过它们并明确说明这一点。 `--input-level` 是当您确实需要强制指定入口点时，特意设置的“后门”。***

## 运行失败时

从 1.2.0 版本开始，`process` 会明确报错，而不是“成功”却没有任何输出：

* 那些**请求了产品但未写入任何产品**的运行——仅限 `project.json` 和 `calibration_data.json` ——会输出 `Processing finished but wrote no image products.` 并**以非零状态退出**， 因此脚本可以检测到该情况。常见原因包括：输入文件夹未被识别为采集任务（请检查文件夹结构及 `--input-level`），或者所有请求的产品均不适用于这些相机（例如，仅使用 RGB 相机时，却请求了辐射度/反射率等参数，而这些相机仅支持辐射度/反射率）。
* **刻意仅运行元数据**（所有产品选项均关闭，不使用 `--indices`）仍被视为成功——此时输出空图像即为正确结果。
* 重新运行时使用 `--verbose`，并检查后端日志中是否包含 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 相关行，这些行会说明各相机的跳过情况。

退出代码：`0` 成功 · `1` 通用错误 · `2` 参数错误 · `130` 被 Ctrl+C 中断。

***

## 植被指数

向 `--indices` 传递一个或多个预设名称；每个指数都会生成独立的 `<INDEX>_Index_Images/` 文件夹：

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

`process --indices` 支持的 22 个预设名称：

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**共有三个索引列表——请勿混淆。**GUI 的“项目设置”下拉菜单中有 27 个公式（新增了 `FCI1`、`FCI2`、`GARI`、 `GEMI`、`LCI` ——这五个仅限图形用户界面使用，且**不**适用于 `--indices`）。 在线/离线模式下的 `lattice index --preset` 命令使用其独立的 22 项预设列表。公式和波段运算在 [多光谱指数公式](project-settings/multispectral-index-formulas.md) 中有详细说明。
{% endhint %}

***

## DAQ 光传感器：简要介绍

`daq pool-*` 系列驱动 MAPIR DAQ 光谱传感器（通过后端持久化池——GUI、CLI 和 SDK——实现 USB 接口的 DAQ-U、 通过 BLE 连接的 DAQ-M、通过以太网连接的 DAQ-E），这些传感器通过后端的持久化池进行驱动——GUI、CLI 和 SDK 均共享一个活动句柄。 **`pool-*` 是出厂版 CLI 中受支持的 DAQ 路径**； 您可能看到的其他 `daq` 子命令，实际上是 MAPIR 内部的仅源表面，执行时会因显式错误而退出，并提示您使用 `pool-*`。

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record`（不带 `--duration`）将运行至 `pool-record --stop`； 默认输出目录位于 **后端机器上** 的 `~/Documents/DAQ Live View/`。 电容校正配置文件在连接时选择（`--cap-id`，后端默认 `sunshine_cosine`），并可通过 `pool-set-cap` ——关于电容校准配置文件和传感器的校准范围，请参阅本手册的 DAQ 章节。

{% hint style="warning" %}
**多网卡主机上的 DAQ-E：**即使传感器状态正常，启动后首次 `pool-connect --eth` 自动发现也可能失败。`--eth-host <ip-or-hostname>` 是可靠的配置方式——当发现失败时请使用该配置。
{% endhint %}

***

## LATTICE 相机、PTP 与项目自动化

`lattice` 系列（45 多个子命令）涵盖了 LATTICE 相机的端到端操作：发现、单次拍摄、通过 GUI 的“智能预备连接”流程实现的持久同步数组、浏览器实时预览、对齐、索引计算以及主机网卡诊断。 示例：

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

此外：`chloros-cli time-sync` 会报告由 Chloros 主机运行的 PTP 主时钟（LATTICE 相机和 DAQ-E 传感器以此为主时钟，以实现跨设备时间戳同步）， 而 `chloros-cli project` 则打开了一个已保存的 Chloros 项目，并在无头模式下驱动其中的相机、阵列和传感器——包括通过脚本调用的 YAML 采集配方。

这三个系列（`lattice`、`project`、 `daq pool-*`）也是唯一支持使用 `CHLOROS_BACKEND_URL` 驱动 **远程** 后端的系列；核心命令始终针对本地机器。

完整的操作指南详见本手册的 LATTICE 章节；所有参数均收录于 [CLI 参考文档](reference/cli-reference.md)。

***

## 故障排除：前 5 大问题

| 症状 | 解决方法 |
| --- | --- |
| `Login required` 或计划任务在 `Email:` 提示符处挂起 | 在该机器上运行一次 `chloros-cli login EMAIL 'PASSWORD'` —— 没有缓存会话的命令将以交互方式执行，而非立即失败。 |
| `backend unreachable` | 启动 Chloros 桌面应用程序，或直接运行后端二进制文件（`chloros-backend`）。 若将 `lattice`/`project`/`daq pool-*` 指向远程后端，请检查 `CHLOROS_BACKEND_URL`。 |
| 数组连接被阻塞：`FRAMES WILL DROP` / `Reduce ROI to enable` | 主机网卡接收环被重置为默认值——这是此前正常运行的设备拒绝连接的首要原因，通常发生在网卡驱动程序更新之后。 请在**提升权限**的终端上运行 `chloros-cli lattice network --fix`（或设置 `ReceiveBufferLen=256`、`PendingReceives=64`）；参见参考手册中的 *主机网卡设置与调优*。 |
| `daq` 子命令退出：&quot;需要完整的数据采集（DAQ）软件包…&quot; | 出厂预装版本中应包含此功能——编译好的 CLI 仅包含 `daq pool-*` 系列，该系列涵盖连接、流传输、记录和采样点选择功能。 请使用 `pool-*`（或来自 Python 的 `chloros_sdk.connect_daq_sensor()`）。 |
| Jetson 在处理大型文件夹前会显示交换警告 | 添加基于文件的交换空间 — CLI 会精确打印出需要执行的 `fallocate`/`swapon` 命令。 |

***

## 获取帮助

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **所有标志、所有子命令：** [CLI 参考](reference/cli-reference.md)
* **Python 的等效命令：** [Python SDK](api-python-sdk.md) 以及 [SDK 参考](reference/sdk-reference.md)
* **支持：** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
