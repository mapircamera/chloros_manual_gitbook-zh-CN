# API：Python SDK

{% hint style="info" %}
**正在寻找完整的 API 文档吗？** 本页面是一份动手实践教程。所有公开类、方法、精确签名以及可直接复制粘贴的示例均收录在 [SDK 参考文档](reference/sdk-reference.md) 中，该文档已针对 AI 助手进行了优化。**正在使用 AI 助手吗？** 将此 URL 粘贴到聊天窗口中，即可获取完整的最新 Chloros 1.3.0 API：

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

本手册的每一页均可通过其小写 slug + `.md` 以原始 Markdown 格式获取，整本手册的索引位于 `https://mapir.gitbook.io/chloros/llms.txt`。
{% endhint %}

**Chloros Python SDK**（PyPI 上的 `chloros-sdk`）通过 Python 实现了桌面应用程序的所有功能：批量图像处理、LATTICE 摄像头和阵列的实时控制、DAQ 光传感器会话以及保存项目的自动化操作。 它是在GUI和CLI所使用的同一本地后端（HTTP，详见`127.0.0.1:5000`）之上构建的一层轻量级接口，因此这三种操作界面上的行为完全一致。

## 安装

该 SDK随 Chloros 桌面软件包一同提供，该软件包还包含处理后端和硬件运行时，且其版本始终与 SDK 保持一致。

**步骤 1 — 安装 Chloros。**Windows：从 [下载](download.md) 页面运行桌面安装程序（默认路径 `C:\Program Files\Chloros\`）。 Linux：使用 `sudo apt install ./chloros_1.3.0_<arch>.deb` 安装 `.deb` 软件包（[Linux 安装指南](linux/linux-installation.md)）。**步骤 2 — 检查 SDK**（Python 3.7 及以上版本）。通常无需安装任何内容：

* **Windows：** 安装程序会将捆绑的 SDK wheel 安装到系统的 Python 中（如果未安装 Python，则跳过此步骤）。
* **Linux：** `.deb` 会将系统 SDK `python3` 安装到 `/usr/lib/python3/dist-packages` 中。 该软件包推荐的依赖项 `python3-numpy` 和 `python3-requests`：`apt install` 会一并安装，而 `dpkg -i` 则不会。
* **虚拟环境**无法识别系统中的软件包；请将捆绑的 wheel 文件安装到其中： 在 Linux 上安装 `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`，或在 Windows 安装目录的 `sdk` 文件夹中安装 `chloros_sdk-*.whl`（默认位于 `C:\Program Files\Chloros\sdk\`）。

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

在未安装 Chloros 软件包的机器上，若其 SDK 驱动着另一台机器上的后端（参见 [仅使用 pip 安装会得到什么](#what-a-pip-only-install-gets)），请从 PyPI 安装：`pip install chloros-sdk`。 PyPI 会在发布版本时更新，因此始终与最新的稳定版本保持一致。

**步骤 3 — 每台机器登录一次：**

```bash
chloros-cli login user@example.com 'YourPassword'
```

凭据会缓存到 `~/.chloros/` 中（两个平台均适用）。在 Windows 上，您也可以通过桌面应用的“用户”<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 选项卡进行登录。该 SDK 需要付费的 Chloros+ 套餐 —— 请参阅下方的 [许可要求](#license-requirement)。

| 要求 | 详细信息 |
| --- | --- |
| **已安装 Chloros** | Windows：桌面安装程序；Linux：`.deb` 软件包（提供后端二进制文件和 SDK） |
| **Python** | 3.7 或更高版本（在 3.10 上开发/测试） |
| **操作系统** | Windows 10/11 64 位、Ubuntu 22.04 LTS 或更高版本，或 NVIDIA Jetson（JetPack 6） |
| **许可** | 有效的 Chloros+ 登录账号，任意付费套餐（Copper 或更高） |

## 60 秒搞定

仅需一次调用即可创建项目、导入文件夹、配置处理流程并运行管道——若后端尚未运行，系统将自动启动：

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

（在 Linux 上，请使用 Linux 路径：`/home/user/drone_images/flight001`。该 SDK 在两个平台上的工作原理完全相同。）

正在处理 LATTICE 捕获文件夹？ 请使用 LATTICE 专用封装脚本——它会应用正确的默认设置（不检测面板目标，使用标准去拜耳算法）：

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` —— 全面控制处理管道

对于超过一行命令的任何操作，请使用 `ChlorosLocal`。它会使用已运行的后端（即桌面应用程序的后端），或在首次使用时启动已安装的后端（`auto_start_backend=True`； 在 Windows 系统中，默认从 Chloros 安装目录下的 `backend` 文件夹启动，默认使用 `C:\Program Files\Chloros\backend\`），用于创建和配置项目、监控进度，并在运行后返回摘要。

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
请保留默认的 `http://127.0.0.1:5000`，而非替换为 `localhost` ——在 Windows 系统上，`localhost` 会首先解析为 `::1`，且针对仅支持 IPv4 的后端，每次请求约需 2 秒。
{% endhint %}

将其用作上下文管理器。退出代码块后，该实例对后端的使用将终止，但后端本身不会停止：它将继续运行以供下一个脚本使用，并在闲置 30 分钟后自动停止。`shutdown_backend()` 可停止该实例启动的后端。

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` 支持以下关键字： `debayer`、`vignette_correction`、`reflectance_calibration`、`indices`、`export_format`、 `ppk`、`daq_log_path`、`input_level`、`radiometric_output`、`array_alignment`、 `array_alignment_crop`、`array_alignment_interpolation` 以及 `custom_settings`。主要数值：

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE 专用的调节参数（`input_level`、`radiometric_output` 以及 `array_alignment*` 系列）及其完整的值表已在 [SDK 参考](reference/sdk-reference.md#supported-values)中已详细记录，并附有完整的值表。

### 监控进度

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### 读取运行后摘要——并捕获空运行

运行完成后，`process()` 会将后端的处理摘要作为 `result["summary"]` 附加。`summary["hints"]` 中的每一条目都是一句完整的说明，解释任何值得注意的情况——例如， 为何某次运行未产生任何输出——而且每条提示都会以Python格式重新输出为`UserWarning`，因此即使您从未检查过该字典，空运行也能实现自我诊断：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**当运行未生成任何图像时，`process()` 不会触发异常。**这是 SDK 和 CLI 之间唯一一个刻意设计的差异： `chloros-cli process` 将“请求了产品，但未写入任何产品”视为失败并返回非零状态码，而SDK则正常返回，并通过 `summary` / 提示信息报告该情况。 如果您的管道在空运行时应停止，请自行检查——检查 `summary`（或统计项目文件夹下的文件数量），而不是依赖异常。
{% endhint %}

## Smart Connect — 实时硬件

三个辅助程序会在后端硬件池中打开持久会话——该池与CLI、其他脚本以及GUI的相机选项卡所使用的池相同，因此SDK脚本可与它们共存，无需争夺串行端口或网络带宽。如果未运行任何后端，这三个程序都会自动启动本地后端。 GUI 的 DAQ 面板不属于该池：请先在 GUI 中关闭光传感器，然后再通过脚本打开它。

### 单个 LATTICE 摄像头 — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**相对路径指向脚本运行的位置（在 1.3.0 版本中已更改）。** 使用本地后端时，相对路径 `output_dir`（包括默认的 `"output"`）——以及后端打开的其他路径 （`burst_dir`、`profile_path`、`save_path`、 `mask_path`）——将根据您的脚本的工作目录进行解析，因此 `cam.capture("output/")` 会写入 `<current folder>/output/`。 在 1.2.0 版本中，已安装的后端会在其自身的安装文件夹中解析这些路径，并因“访问被拒绝”而失败。对于远程后端，路径将按输入的形式发送。
{% endhint %}

### 同步数组 — `connect_array`

`connect_array` 是多机位拍摄系统的推荐入口点。它运行与 GUI 相同的智能预备流程：网络分析、同步层自动选择、PTP 时间同步、按摄像机选择像素格式、AE 初始化以及 GPIO 触发就绪。 **第一个串行端口为主设备**（它触发硬件触发脉冲）；其余为从设备。

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

在任何数组捕获中添加 `smart=True`，可在触发前等待所有摄像机的自动曝光稳定。

1.3.0 版本新增的 `ArraySession` 还能直接向您提供同步帧，无需写入文件：

* `arr.grab()` 通过单次触发，为每台相机返回一帧图像，均以 numpy 数组形式输出（`{serial: array}`）； `.group_id` 是共享触发器，`.errors` 表示被排除在外的任何相机。
* `arr.frame_stream(fps=5.0)` 每 tick 返回一组此类数据，供您在自己的处理管道中进行实时处理。
* `arr.continuous("output/")` 运行后端自身的连续捕获（即 GUI 中的“连续”模式），并返回一个包含 `ContinuousHandle` 和 `status()` 的 `arr.continuous("output/")`， `wait()` 和 `stop()` — 当您需要以高帧率保存每个触发事件时，请使用该对象代替捕获循环。

有关采集模式（单次/连续/间隔/最快）、记录器、突发视频和数组对齐，请参阅 [SDK 参考](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep)。

### DAQ 光传感器 — `connect_daq_sensor`

若不带参数，`connect_daq_sensor()` 将智能检测传输协议（优先级：以太网 → BLE → USB）：

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

每个帧包含 135 个数据点的 `spectrum`（校准后为 W/m²/nm）、一个 `is_saturated` 标志以及 CIE `x`， `y`、`z`。若要指定特定传感器或传输协议，请传递一个显式提示：

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

请注意，电容校正配置文件（`cap_id`）**并非** SDK 控制项——请改用 `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` 进行选择。

### 已保存的项目 — `open_project`

已保存的Chloros项目会保留其已连接的硬件（`cameras.json` + `sensors.json` 以及 `project.json`）， 而 `chloros_sdk.open_project(path)` 可以一次性重新连接其摄像头、阵列和光传感器，并通过设备名称触发捕获操作。 光传感器通过正在运行的Chloros后端重新连接，并保留其已保存的设置和捕获能力。请参阅参考文档中的[项目自动化](reference/sdk-reference.md#project-automation--chlorosproject)。

## 仅安装 pip 时的功能

在使用硬件界面之前，请检查模块级别的可用性标志：

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

在**仅**安装了 `pip install chloros-sdk` 且未安装 Chloros 桌面包的主机上：

* `ChlorosLocal`、`process_folder` 和 `process_lattice_capture` **无法**正常工作——它们需要桌面安装程序中附带的后端二进制文件。
* 智能连接辅助程序（`connect_camera`、`connect_array`、`connect_daq_sensor`）是纯粹的HTTP客户端，因此可与位于另一台机器上的后端配合使用 ——但随附的后端仅绑定回环地址，因此您必须自行进行端口转发（例如 `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`），并将 `backend_url="http://127.0.0.1:5000"` 与 `auto_start_backend=False` 配合使用。 参见 [远程后端模式](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)。
* 直接硬件 LATTICE 类（`LatticeCamera`、 `CameraPool`, …) 可以导入，但需要来自桌面包的 Arena SDK 运行时——若缺少该运行时，`CAMERA_AVAILABLE` 将变为 `False`。
* DAQ 传感器仅通过（隧道化的）后端驱动——`connect_daq_sensor()`，或项目的 `proj.sensors`；后端会在其自身机器上打开传感器。

## AI 助手 — `chloros-mcp`

该SDK包含`chloros-mcp`，这是一个MCP服务器，允许Claude Desktop、Claude Code、Cursor及其他MCP客户端驱动Chloros：处理图像，并运行LATTICE相机、阵列和DAQ光传感器。 其工具涵盖了本SDK的功能。它随SDK一同安装，因此对于Claude Code，配置只需一行代码：

```bash
claude mcp add chloros -- chloros-mcp
```

其他客户端：[如何在AI助手中使用Chloros](ai-assistants.md)。 选项及所有工具：[MCP 服务器（AI 助手）](reference/sdk-reference.md#mcp-server-ai-assistants)。

## 许可要求

访问 SDK 需要拥有任何付费层级的有效 Chloros+ 登录账号——**Copper 或更高**（Copper / Bronze / Silver / Gold）； 免费的 Iron 套餐不提供 SDK/CLI 访问权限。该限制在**服务器端** 执行：每个 SDK 请求必须同时包含有效会话和付费套餐，否则后端将返回 `403` / `PLAN_UPGRADE_REQUIRED` （由 `ChlorosLocal` 引发的错误代码为 `ChlorosLicenseError`，由 `connect_*` 辅助函数引发的错误代码为 `ChlorosConnectError`）。 已注销的调用者会收到 `401` / `AUTH_REQUIRED` （`ChlorosAuthenticationError`）——重新运行 `chloros-cli login` 可解决第一种情况，但无法解决第二种情况。

在套餐的宽限期内，离线使用是可行的：服务等级信息将从服务器验证缓存（5 分钟）或已签名、与机器绑定的许可证缓存（月度套餐为 30 天；年度套餐为订阅到期日）中读取。 当宽限期结束时，套餐将转为免费状态，且SDK访问将暂停，直到设备与服务器建立一次连接为止。`chloros-cli status`在免费套餐下仍可访问，因此原因始终可见。 请参阅 [Chloros+ 登录](chloros+-login.md)。

## 异常

捕获基类以处理“Chloros 出现的任何错误”：

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

所有管道异常（`ChlorosBackendError`、`ChlorosConnectionError`、`ChlorosLicenseError`、`ChlorosAuthenticationError`、 `ChlorosConfigurationError`、`ChlorosProcessingError`）均继承自 `ChlorosError`。 一个注意事项：`ChlorosConnectError` —— 仅由 `connect_camera` / `connect_array` / `connect_daq_sensor` 触发 ——源自普通的 `Exception`，**而非** `ChlorosError`，因此 `except ChlorosError` 无法捕获该异常。 完整的层次结构请参见 [SDK 参考](reference/sdk-reference.md#exceptions)。

## 另请参阅

* [SDK 参考](reference/sdk-reference.md) — 完整的 API 接口，针对 AI 助手进行了优化。
* [CLI 参考](reference/cli-reference.md) — 每个 CLI 子命令都对应一个 SDK 调用。
* [在 AI 助手中使用 Chloros](ai-assistants.md) —— `chloros-mcp` 的配置和提示词示例。
* [下载](download.md) —— 适用于 Windows 和 Linux 的安装程序。
