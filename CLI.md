# 命令行界面（CLI）

> **完整参考：**[CLI参考](reference/cli-reference.md) 记录了**每个子命令的每个参数**，并针对 AI 助手进行了优化——将该 URL 粘贴到您的助手中，并请求一个可用的命令：`https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **AI 工具使用提示：**本手册的任意页面均可通过在URL后添加 `.md`（例如 `https://mapir.gitbook.io/chloros/reference/cli-reference.md`）获取原始 Markdown 格式， 而 `https://mapir.gitbook.io/chloros/llms.txt` 则为大型语言模型（LLM）提供了整本手册的索引。

> **希望您的助手直接运行 Chloros 吗？** `chloros-mcp` 是一个随 Python SDK 安装的 MCP 服务器，可与 Claude Desktop、Claude Code、Cursor 及其他 MCP 客户端配合使用：请参阅 [如何在 AI 助手中使用 Chloros](ai-assistants.md)。

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## 什么是 CLI


`chloros-cli` 是 Chloros
桌面应用所使用的同一处理引擎的命令行前端。 它是一个基于 Chloros
后端（位于 `127.0.0.1:5000` 上的本地服务器）的轻量级 HTTP
客户端——大多数命令会自动启动后端，因此脚本只需调用一次 `chloros-cli process …` 即可。 随后，后端将保持运行以等待下一个命令： 它会在执行 `chloros-cli logout` 时停止（除非应用程序处于打开状态，或正在进行运行或构建），在执行 `chloros-cli backend stop` 时停止，或者在 30 分钟内没有客户端、运行任务或连接设备的情况下自动停止。 `chloros-cli backend status` 会显示该状态。

它可在 **Windows
10/11 (x64)**和**Linux
(x86_64，以及 JetPack 6 上的 NVIDIA Jetson arm64)** 上运行，支持任何终端，无需图形界面。 请使用以下命令验证安装：

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

命令家族一览：

* **处理与账户** — `process`、`login`、`logout`、`status`、 `export-status`、`language`（38种语言——参见[支持的语言](supported-languages.md)）， `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` （仅限 Linux
/Jetson）
* **实时硬件** — `lattice`（LATTICE 摄像头控制，45+ 个子命令），`daq pool-*`（DAQ 光传感器），`time-sync` (PTP)
* **自动化** — `project`（无头模式下运行已保存的Chloros
项目，包括YAML捕获配方）

值得了解的全局选项：`--port N`（后端端口，默认值为 `5000`）、`-v/--verbose`、 `--restart`（强制重启后端）、`--backend-exe PATH`。完整列表请参阅 [CLI
参考](reference/cli-reference.md)。

***

## 安装

CLI
**包含在所有平台的 Chloros
安装程序中** —— 无需单独下载 CLI
。请从 [下载](download.md) 页面获取安装程序。

### Windows


安装程序会将 CLI
放置在：

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

并将该文件夹添加到您的系统 `PATH` 中一次（升级时也会移除早期安装程序留下的重复条目）——安装后请 **打开一个新的终端**，以便识别更新后的 `PATH`。 安装程序还会在安装根目录下放置启动脚本（`Chloros_CLI.bat` / `Chloros_CLI.ps1`），并添加一个**Chloros
CLI
** 开始菜单快捷方式，点击任意一个都会打开一个终端，其中已预先加载好可用的 `chloros-cli`。

### Linux


请使用 `apt` 安装适用于您架构的 `.deb`（请勿使用 `dpkg -i`，该版本会跳过 Python
SDK
的推荐依赖项）：

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

这将安装 `chloros-cli` 至 `/usr/bin/chloros-cli` （`PATH`中已包含），后端和CLI
程序解压在`/usr/lib/chloros/backend/`和`/usr/lib/chloros/cli/`中 （`/usr/lib/chloros/chloros-backend` 是一个用于启动后端的启动器），后端运行 LATTICE 摄像头所需的 Arena SDK
运行时，以及系统所需的 Python
SDK
（`python3`）。 详情请参阅 [Linux
安装指南](linux/linux-installation.md)。

### 验证

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## 登录与授权

访问 CLI
（及 Python
SDK
）需要 **付费的 Chloros
+ 套餐**—— 任何付费套餐均包含此功能；免费套餐则不包含。该限制由后端在**服务器端** 强制执行，而非由 CLI
二进制文件执行：未登录的调用将被 `401 AUTH_REQUIRED` 拒绝， 而在免费套餐下，已登录的调用将返回错误代码 `403 PLAN_UPGRADE_REQUIRED`，无论该调用来自 `chloros-cli`、SDK
还是自定义的 HTTP
客户端。 请在 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) 处进行升级。

**每台机器仅需登录一次**：

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**包含特殊字符的密码**（`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$`会被 shell 篡改——CLI
在遇到 401 错误时会检测到此问题并自动重试，但使用单引号可完全避免该问题）。
{% endhint %}

会话缓存于 `~/.chloros/user_session.json` 中，并在套餐的宽限期内持续离线工作（月度套餐为 30 天，年度套餐则持续至到期）。 即使没有付费套餐，`chloros-cli status` 也能正常工作，因此拒绝的原因始终可见。

{% hint style="danger" %}
**要调度无界面任务？请先登录。**若在**无缓存会话**的情况下运行后端进程生成命令（`process`、`status`、 `export-status` 等）若在**无缓存会话**的情况下运行，不会快速报错——而是会切换到交互式 `Email:` / `Password:` 提示符，并通过标准输入（stdin）接收命令。 因此，无人值守的 cron 任务或 CI 步骤将**因等待输入而挂起**。在安排任何任务之前，请先在该机器上运行一次 `chloros-cli login EMAIL 'PASSWORD'`。
{% endhint %}

***

## 首次处理运行

将 `process` 指向捕获文件夹——它会自动检测 Survey3
（`.raw` + `.jpg`）、 LATTICE（`.tif`/`.tiff`）、`.dng`，或上述格式的混合：

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

进度流按管道线程实时更新（检测、分析、处理、导出），成功运行后会报告写入的图像产品数量： `Image products written: N`，若运行过程中还写入了 [钳位掩膜](output-image-formats.md#clamp-masks)，则随后会输出 `(plus M clamp masks)`（钳位掩膜不计入产品数量）。

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


如果Chloros
后端已处于运行状态（例如桌面应用程序的后端），`process`将使用该后端并在其中打开其项目。当应用程序正在处理其已打开的项目时，该命令会拒绝执行：请从应用程序中处理该任务，或关闭该项目。 如果之前的运行仍在结束中，它会等待最多 15 秒，之后才会拒绝。`--restart` 会强制启动一个新的后端。 在 Linux
上，由 systemd 服务运行的后端绝不会被重启：`process` 会直接使用该后端，而在原本应重启后端的情况下（如 `--restart`，或导入操作未找到任何内容时）， 它会输出 `sudo systemctl restart <unit>` 并退出 `1`。

### 输出结果的存储位置

`process` 将文件写入 **项目文件夹**，而非您的输入文件夹：

* 若未指定 `-o`：项目将创建在您的默认项目文件夹下（与 GUI 共享；可通过 `get-project-folder` / `set-project-folder` 进行管理； 若未设置，则位于您的用户文件夹中的 `Documents/MAPIR/Chloros Projects`），若未指定名称，则以 `-n/--project-name` 或时间戳（`YYYYMMDD_HHMMSS`）命名。
* 使用 `-o PATH` 时：该文件夹 **即** 为项目文件夹。如果该文件夹中已包含 `project.json`，则会创建后缀为 `_1`/`_2`… 后缀的同级文件，而非覆盖原有文件。相对路径 `PATH` 是以您运行 `chloros-cli` 的文件夹为基准的。

{% hint style="info" %}
**相对路径（在 1.3.0 版本中已更改）。** 后端打开的每个路径——无论在任何命令中（包括捕获命令的默认路径 `output`），如 `-o`/`--output`， `--input`、`--burst-dir`、`--profile`、`--save` 和 `--mask` ——将根据您运行 `chloros-cli` 的文件夹进行解析，因此 `-o captures/` 会写入 `<current folder>/captures/`。 在 1.2.0 版本中，后端会在其自身的安装文件夹中解析这些路径，这会导致“访问被拒绝”的错误。对于远程后端（`CHLOROS_BACKEND_URL`），路径将按输入形式发送，并在该机器上进行解析。
{% endhint %}

{% hint style="warning" %}
**Windows
：除非启用了长路径，否则请保持项目文件夹名称简短。** 当 Windows
关闭长路径功能（Windows
默认设置）时，任何文件路径的长度不得超过 260 个字符，且子文件夹名称在项目文件夹下的层级深度不得超过约 130 个字符。 `process` 在启动前会进行此项检查：如果项目文件夹层级过深，将显示 **“该项目文件夹层级对这台计算机而言过深”** 并拒绝处理；同时会提示最长的符合要求的项目文件夹名称以及如何启用长路径功能——在此之前不会进行任何处理。 请使用更短的 `-o`（或 `set-project-folder`），或启用 Windows
长路径（安装程序中的 **启用 Windows
长路径** 选项，默认勾选），然后重新启动 Chloros
。
{% endhint %}

在项目内部，产品是**按相机分类，然后按文件格式分类**的：

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

对于 LATTICE，相机文件夹为 `LATT-<sensor>-<lens>-F<filter>`（与拍摄文件的 EXIF 信息 `Model` 匹配），而 Survey3 的相机文件夹为 `<model>_<filter>` （例如 `Survey3N_RGN`）对应Survey3
。格式文件夹的命名规则为 `--format`：`tiff16`、`tiff8`、 `png8`、`jpg8` 或 `tiff32` 对应 `TIFF (32-bit, Percent)`。

对于多光谱 M3C 相机，格式文件夹中除产品文件夹外，还可能出现一个 `clamp_masks/` 文件夹（例如 `tiff16/clamp_masks/Reflectance_Calibrated_Images/` 和 `tiff32/clamp_masks/Radiance_Images/`）。 该文件夹包含辐射度和反射率产品的逐像素掩膜，在这些产品中，波段解混将负值限制为零——参见 [限制掩膜](output-image-formats.md#clamp-masks)。任何产品文件夹中都不包含掩膜。

{% hint style="info" %}
**每个导出的产品都保留源文件的名称。**`capture_..._raw.tif` 的辐射度导出文件仍命名为 `capture_..._raw.tif` —— 它只是位于 `tiff32/Radiance_Images/` 目录下。**文件夹用于标识产品，而非文件名**，因此应使用通配符匹配目录，而非 `*radiance*` 这样的后缀。
{% endhint %}

### 您实际会用到的选项

| 标志 | 默认值 | 功能说明 |
| --- | --- | --- |
| `-o, --output PATH` | 默认项目文件夹 | 项目文件夹位置（参见上文）。 |
| `-n, --project-name NAME` | 时间戳 | 项目名称。 |
| `--format FMT` | `TIFF (16-bit)` | 以下选项之一：`TIFF (16-bit)`、 `TIFF (8-bit)`、`TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)` 中的一个。 Chloros
不进行图像放大：对于所有8位图像，更深的格式会被写为`TIFF (8-bit)`，且仅提示一次。 |
| `--indices NAME [NAME ...]` | 无 | 待导出的植被指数（参见 [植被指数](#vegetation-indices)）。 |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = 神经网络去拜耳化，速度较慢， 最高质量（Chloros
+，需要 CUDA GPU：若无 CUDA GPU，则运行时使用 `standard`；LATTICE 捕获数据不支持此选项，该类数据始终采用 `standard`）。 |
| `--vignette / --no-vignette` | 启用 | 暗角校正。 |
| `--reflectance / --no-reflectance` | 启用 | 反射率校准；对于 LATTICE，此选项同时用于切换反射率乘积功能。 |
| `--input-level {auto,raw,debayered,processed}` | `auto` | 强制 LATTICE TIFF 文件的处理管道入口点。 |

关于其他所有内容——目标检测调整、PPK、曝光定位点、阵列对齐标志——请参阅 [CLI
参考手册中`process`部分](reference/cli-reference.md)。

***

## 选择导出内容（LATTICE 产品）

LATTICE 处理会在 **单次处理中**扩展到所有适用产品。每个产品的四个开关**默认均处于开启状态**；使用 `--no-` 格式可禁用其中一个：

| 开关 | 产品 |
| --- | --- |
| `--debayered` | 线性去马赛克 → `Debayered_Images/`： 移除校准包中的暗偏移，随后进行边缘感知去马赛克处理，不进行线性校正、平场校正或分色处理——即与捕获设备在 `debayered` 级别保存的文件相同。保持传感器原生方向（记录在 XMP 中）。 |
| `--preview` | 显示预览（白平衡 + 伽马校正；多光谱图像的伪彩色拉伸）→ `Preview_Images/` |
| `--radiance` | float32 辐射度，W/m²/sr/nm → `Radiance_Images/`（始终为 `tiff32/`） |
| `--reflectance` | uint16 反射率，Pix4D 兼容 → `Reflectance_Calibrated_Images/` |

仅 RGB
主相机始终输出去拜耳化 + 预览数据——对于宽带传感器而言，各波段的辐射度/反射率没有实际意义，因此这些开关对它们而言是无操作的。Survey3
`.raw` 会忽略这些开关，并遵循标准的反射率/目标路径。

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`**（默认值为 `auto`）选择反射率参考： `auto` 将通过质量保证（QA）检测的帧内 [校准目标](calibration-targets.md) 设为绝对参考，并在无目标存在时回退至数据采集（DAQ）光传感器下行光分界值 (ρ = π·L/E)（带有通过质量保证标志的目标会与数据采集模块在测量分数上产生竞争）；`target` 采用严格模式（不进行数据采集替换）； `daq`以DAQ结果为准。可通过`--target-reflectance-dir`提供按单元测量的目标扫描数据。

{% hint style="info" %}
**读取反射率像素：**表示 ρ = 1.0 的 DN 是**按光源计算的** —— LATTICE 文件会在 XMP 中添加 `Chloros:PixelScale=32768` 标记； Survey3
文件使用65535，且不包含`Chloros:PixelScale`标签（但包含其他`Chloros:*`处理标签）。 请读取该标签并以此为基准进行除法运算，而非假设其为常数。详细信息及一个刻意设计的无比例尺边界情况请参见[CLI
参考](reference/cli-reference.md)。
{% endhint %}

**处理始终从 `raw` 开始。** 衍生产品（去拜耳化/辐射度/反射率导出）绝不会被重新输入到处理管道中——重新导入并处理它们会导致校准运算被重复应用，因此 Chloros
会跳过它们并明确提示。 `--input-level`是当您确实需要强制指定入口点时，特意设计的“后门”。

***

## 运行失败时

从 1.2.0 版本开始，`process` 会明确报错，而不是“成功”却没有任何输出：

* 若运行**请求了产物但未写入任何产物**——仅包含 `project.json` 和 `calibration_data.json` ——则会输出 `Processing finished but wrote no image products.` 并**以非零状态退出**， 因此脚本可以检测到此情况。常见原因包括：输入文件夹未被识别为捕获文件夹（请检查布局和 `--input-level`），或者所有请求的产出对于这些相机均不适用（例如，从仅支持 RGB
的相机中请求辐射度/反射率）。
* **刻意仅生成元数据的运行**（所有产品选项均关闭，无 `--indices`）仍被视为成功——此时输出空图像即为正确结果。
* 请启用 `--verbose` 重新运行，并检查后端日志中是否存在 `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` 相关行，这些日志条目会说明每台相机的跳过原因。
* 在Windows
系统中，若已禁用长路径功能，当项目文件夹路径深度超出260个字符的限制时，系统会在处理开始**之前**拒绝该路径，并使用符合限制的最长项目文件夹名称（参见[输出文件存放位置](#where-the-outputs-land)）。
* 捕获命令（`lattice capture`、`lattice array-capture` 及其他捕获系列）会列出所有无法写入的文件，并以 `1` 状态退出；当磁盘空间已满时，重复捕获操作将停止。

退出代码：`0` 成功 · `1` 通用错误 · `2` 参数错误 · `130` 被 Ctrl+C 中断。

***

## 植被指数

向 `--indices` 传递一个或多个预设名称；每个指数将保存在各自的 `<INDEX>_Index_Images/` 文件夹中：

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

`process --indices` 支持的 22 个预设名称：

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

向项目中添加了一个该项目尚未列出的名称。如果名称不是预设名称，或者摄像机频段无法生成该名称，则会跳过该名称并发出警告，同时标明该名称。 如果CLI
无法读取项目当前的索引列表，它将保持列表不变并发出警告，而非替换该列表。

{% hint style="warning" %}
**共有三个索引列表——请勿混淆。**GUI 的“项目设置”下拉菜单中有 27 个公式（包括 `FCI1`、`FCI2`、`GARI`、 `GEMI`、`LCI`——这五个仅限图形用户界面使用，且**不**适用于 `--indices`）。 实时/离线命令 `lattice index --preset` 使用其独立的 22 个预设列表。公式和波段运算详见 [多光谱指数公式](project-settings/multispectral-index-formulas.md)。
{% endhint %}

***

## DAQ 光传感器：快速入门

`daq pool-*`系列通过后端持久化池驱动MAPIR
DAQ光谱传感器（通过USB连接的DAQ-U、通过BLE连接的DAQ-M、通过以太网连接的DAQ-E）——GUI、CLI
和SDK
均共享一个实时句柄。 **`pool-*`是出厂版CLI
中支持的DAQ路径**；您可能看到的其他 `daq` 子命令，属于 MAPIR
内部的仅源接口，执行后会返回显式错误并引导您使用 `pool-*`。

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

`pool-record`（不带 `--duration`）将运行至 `pool-record --stop`； 默认输出目录为 `~/Documents/DAQ Live View/` **位于后端机器上**。 电容校正曲线在连接时选择（`--cap-id`，后端默认 `sunshine_cosine`），并可通过 `pool-set-cap` ——电平校正配置文件及传感器的校准范围在本手册的 DAQ 章节中有详细说明。

{% hint style="warning" %}
**多网卡主机上的 DAQ-E：** 即使传感器状态正常，启动后首次 `pool-connect --eth` 自动发现也可能失败。`--eth-host <ip-or-hostname>` 是可靠的形式——当发现操作未果时，请使用该配置。
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

此外：`chloros-cli time-sync` 会报告 Chloros
主机运行的 PTP 主时钟（LATTICE 相机和 DAQ-E 传感器以此为主时钟，以实现跨设备时间戳同步）， 而 `chloros-cli project` 则打开一个已保存的 Chloros
项目，并在无头模式下驱动其相机、阵列和传感器——包括通过脚本编写的 YAML 采集配方。

这三个系列（`lattice`、 `project`、`daq pool-*`）也是唯一支持通过 `CHLOROS_BACKEND_URL` 驱动 **远程** 后端的系列；核心命令始终针对本地机器。

完整的操作指南详见本手册的 LATTICE 章节；所有参数均收录于 [CLI
参考](reference/cli-reference.md)。

***

## 故障排除：前 5 大问题

| 症状 | 解决方法 |
| --- | --- |
| `Login required` 或计划任务在 `Email:` 提示符处卡住 | 在该机器上运行一次 `chloros-cli login EMAIL 'PASSWORD'` —— 没有缓存会话的命令将以交互方式执行，而非立即失败。 |
| `backend unreachable` | 启动 Chloros
桌面应用程序，或手动启动后端：Windows
系统上为 `"C:\Program Files\Chloros\backend\chloros-backend.exe"`（位于安装文件夹的 `backend` 文件夹中； 该文件夹不位于 `PATH` 中），Linux
系统上的 `chloros-backend`。`chloros-cli backend status` 显示是否正在运行； `lattice`/`project`/`daq pool-*` 绝不会启动后端。若将其指向远程后端，请检查 `CHLOROS_BACKEND_URL`。 |
| 数组连接受阻：`FRAMES WILL DROP` / `Reduce ROI to enable` | 主机网卡接收环路重置为默认值——这是此前正常运行的设备拒绝连接的首要原因，通常发生在网卡驱动程序更新之后。 请在**提升权限**的终端中运行 `chloros-cli lattice network --fix`（或设置 `ReceiveBufferLen=256`、`PendingReceives=64`）；请参阅参考手册中的 *主机网卡设置与调优*。 |
| `daq` 子命令退出：&quot;需要完整的 DAQ 软件包…&quot; | 请使用 `daq pool-*` 命令，这些命令通过后端驱动同一传感器：连接、流式传输、记录、捕获选择，以及针对 DAQ-E 的网络设置、重启和固件更新（或通过 Python
执行 `chloros_sdk.connect_daq_sensor()`）。 |
| Jetson 在处理大型文件夹前会打印交换空间警告 | 添加基于文件的交换空间——CLI
会打印出需要执行的确切 `fallocate`/`swapon` 命令。 |

***

## 获取帮助

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **所有标志、所有子命令：** [CLI
参考](reference/cli-reference.md)
* **Python
等效命令：** [Python
SDK
](api-python-sdk.md) 及 [SDK Reference](reference/sdk-reference.md)
* **支持：** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
