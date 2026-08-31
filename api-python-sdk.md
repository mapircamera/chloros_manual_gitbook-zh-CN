# API : Python SDK

{% hint style="info" %}
**正在寻找完整的 API 内容吗？** 本页面是一篇实操教程。 所有公共类、方法、精确签名以及可直接复制粘贴的示例均收录于 [SDK 参考指南](reference/sdk-reference.md)，该指南已针对 AI 助手进行了优化。**正在使用 AI 助手吗？** 将此 URL 粘贴到聊天窗口中，以便它获取完整的、最新的 Chloros 1.2.0 API：

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

本手册的每一页均可通过其小写别名 + `.md` 以原始 Markdown 格式获取，整本手册的索引位于 `https://mapir.gitbook.io/chloros/llms.txt`。
{% endhint %}

**Chloros Python SDK** （PyPI 上的 `chloros-sdk`）驱动了桌面应用程序从 Python 起所能实现的所有功能：批量图像处理、LATTICE 实时摄像头和阵列控制、DAQ 光传感器会话以及保存项目的自动化。 它是在 GUI 和 CLI 所使用的同一本地后端（位于 `127.0.0.1:5000` 上的 HTTP）之上构建的一层轻量级接口，因此这三个界面上的行为完全一致。

## 安装

安装分为两个步骤：首先安装 Chloros 桌面软件包（它提供处理后端和硬件运行时），然后安装 Python 软件包。

**步骤 1 — 安装 Chloros。** Windows：从 [下载](download.md) 页面运行桌面安装程序（默认路径为 `C:\Program Files\MAPIR\Chloros\`）。 Linux：安装 `.deb` 软件包（[Linux 安装](linux/linux-installation.md)）。**步骤 2 — 安装 SDK** (Python 3.7+)：

```bash
pip install chloros-sdk
```

您甚至可能不需要 pip：每个安装程序都附带一个匹配的 SDK wheel。Windows 安装程序会将其自动安装到您的系统 Python 中； 而 Linux `.deb` 则将其放置在 `/usr/lib/chloros/sdk/` 目录下，并显示确切的 `pip install --user` 命令。 PyPI 会在发布版本中更新，因此 `pip install chloros-sdk` 与最新稳定版保持一致。

**步骤 3 — 每台机器登录一次：**

```bash
chloros-cli login user@example.com 'YourPassword'
```

凭据将缓存至 `~/.chloros/`（两个平台均适用）。在 Windows 上，您也可以通过桌面应用的“用户”<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 选项卡进行登录。 SDK 需要付费的 Chloros+ 套餐——请参阅下方的 [许可证要求](#license-requirement)。

| 要求 | 详细信息 |
| --- | --- |
| **已安装 Chloros** | Windows：桌面安装程序； Linux：`.deb` 软件包（提供后端二进制文件） |
| **Python** | 3.7 或更高版本（在 3.10 上开发/测试） |
| **操作系统** | Windows 10/11 64 位、Ubuntu 22.04 LTS 或更新版本，或 NVIDIA Jetson（JetPack 6） |
| **许可** | 有效的 Chloros+ 登录账号，任意付费套餐（Copper 或更高） |

## 60 秒搞定

仅需一次调用即可创建项目、导入文件夹、配置处理流程并运行管道——若后端尚未运行，则会自动启动：

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

（在 Linux 上，请使用 Linux 路径：`/home/user/drone_images/flight001`。SDK 在两个平台上的工作原理完全相同。）

正在处理 LATTICE 捕获文件夹？请使用 LATTICE 兼容的封装程序——它会应用正确的默认设置（不检测面板目标，使用标准去拜耳算法）：

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — 完整的管道控制

对于超过一行命令的任何操作，请使用 `ChlorosLocal`。 它会在首次使用时启动后端（`auto_start_backend=True`），创建并配置项目，监控进度，并在运行后返回摘要。

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
请保留默认的 `http://127.0.0.1:5000`，而非替换为 `localhost` — 在 Windows 上， `localhost` 会首先解析为 `::1`，且在仅支持 IPv4 的后端上，每次请求耗时约 2 秒。
{% endhint %}

将其用作上下文管理器以确保资源清理：

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` 支持以下关键字： `debayer`、`vignette_correction`、`reflectance_calibration`、`indices`、`export_format`、 `ppk`、`daq_log_path`、`input_level`、`radiometric_output`、`array_alignment`、 `array_alignment_crop`、`array_alignment_interpolation` 以及 `custom_settings`。主要参数值：

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

LATTICE特有的调节旋钮 （`input_level`、`radiometric_output` 以及 `array_alignment*` 系列）及其完整的值表已在 [SDK 参考手册](reference/sdk-reference.md#supported-values) 中已附有完整的值表。

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

运行完成后，`process()` 会将后端的处理摘要作为 `result["summary"]` 附加。`summary["hints"]` 中的每条记录都是一句完整的解释，说明任何值得注意的情况——例如， 为何某次运行未产生任何输出——而且每条提示都会作为 Python `UserWarning` 重新输出，因此即使您从未检查过该字典，空运行也能自我诊断：

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**当运行未生成任何图像时，`process()` 不会触发异常。**这是 SDK 和 CLI 刻意设计上的唯一区别： `chloros-cli process` 将“请求了产物，但未写入任何产物”视为失败并返回非零状态，而 SDK 则正常返回，并通过 `summary` / hints 报告该情况。 如果您的管道在空运行时应停止，请自行检查——检查 `summary`（或统计项目文件夹下的文件数量），而不是依赖异常处理。
{% endhint %}

## Smart Connect — 实时硬件

三个辅助程序会在后端硬件池中打开持久会话——该池与 GUI 使用的池相同，因此 SDK 脚本可与桌面应用程序共存，而不会因串行端口或网络带宽发生冲突。如果未运行任何后端，这三个程序都会自动启动本地后端。

### 单台 LATTICE 摄像机 — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### 同步阵列 — `connect_array`

`connect_array` 是多摄像头系统的推荐入口。它运行与图形用户界面（GUI）相同的智能预处理流程：网络分析、同步层自动选择、PTP 时间同步、逐摄像头像素格式选择、自动曝光（AE）初始化以及 GPIO 触发准备。 **第一个串行端口为主设备**（它触发硬件触发脉冲）；其余为从设备。

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

在任何阵列采集任务中添加 `smart=True`，以在触发前等待所有摄像机自动曝光稳定。 有关捕获模式（单帧/连续/间隔/最快）、记录器、连拍转视频以及阵列对齐，请参阅 [SDK 参考文档](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep)。

### DAQ 光传感器 — `connect_daq_sensor`

若不带参数，`connect_daq_sensor()` 将智能检测传输协议（优先级：以太网 → BLE → USB）：

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

每个帧携带 135 点 `spectrum`（校准后为 W/m²/nm）、一个 `is_saturated` 标志以及 CIE `x`， `y`、`z`。若要指定特定的传感器或传输协议——在具有多个网络接口的主机上，这是可靠的选择（因为以太网自动发现功能在首次尝试时可能会遗漏状态正常的DAQ-E）——请传递一个明确的提示：

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

请注意，电平校正配置文件（`cap_id`）**并非** SDK 控制项——应通过 `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` 进行选择。

### 已保存的项目 — `open_project`

已保存的 Chloros 项目会保留其已连接的硬件（`cameras.json` + `sensors.json` 以及 `project.json`）， 而 `chloros_sdk.open_project(path)` 可以一次性重新连接所有设备，并通过设备名称驱动捕获。请参阅参考文档中的 [项目自动化](reference/sdk-reference.md#project-automation--chlorosproject)。

## 仅通过 pip 安装可获得的内容

在使用硬件表面之前，请检查模块级别的可用性标志：

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

在**仅**安装了 `pip install chloros-sdk` 而未安装 Chloros 桌面软件包的主机上：

* `ChlorosLocal`、`process_folder` 和 `process_lattice_capture` **无法**正常工作——它们需要桌面安装程序中附带的后端二进制文件。
* 智能连接辅助程序（`connect_camera`、`connect_array`、 `connect_daq_sensor`）是纯粹的 HTTP 客户端，因此可与另一台机器上的后端配合使用——但随附的后端仅绑定回环地址，因此您必须自行进行端口转发（例如 `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`），并将 `backend_url="http://127.0.0.1:5000"` 与 `auto_start_backend=False` 配合使用。请参阅 [远程后端模式](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)。
* 直接硬件 LATTICE 类（`LatticeCamera`、`CameraPool`、 …）可以导入，但需要来自桌面安装包的 Arena SDK 运行时——若缺少该运行时，`CAMERA_AVAILABLE` 将变为 `False`。
* `daq_sdk`（直接数据采集类）随桌面安装版提供，而非 PyPI 包， 因此在仅使用 pip 的主机上，`DAQ_AVAILABLE` 即为 `False` —— 请改用 `connect_daq_sensor()` 通过（隧道连接的）后端驱动 DAQ 传感器。

## 许可要求

访问 SDK 需要拥有任何付费套餐（**Copper 或更高**，即 Copper / Bronze / Silver / Gold）下有效的 Chloros+ 登录账号； 免费的 Iron 套餐不支持 SDK/CLI 访问。 该限制在**服务器端**执行：每个 SDK 请求必须同时包含有效会话和付费套餐，否则后端将返回 `403` / `PLAN_UPGRADE_REQUIRED` （由 `ChlorosLocal` 触发为 `ChlorosLicenseError`，由 `connect_*` 辅助函数触发为 `ChlorosConnectError`）。 已注销的调用者会收到 `401` / `AUTH_REQUIRED` （`ChlorosAuthenticationError`）—— 重新运行 `chloros-cli login` 可解决第一种情况，但无法解决第二种情况。

在计划的宽限期内，离线使用是可行的：层级信息从服务器验证缓存（5 分钟）或已签名、与机器绑定的许可证缓存（月度计划为 30 天；年度计划为订阅到期日）中读取。 当宽限期结束时，套餐将转为免费套餐，且 SDK 的访问权限将暂停，直到该设备与服务器建立一次连接为止。`chloros-cli status` 在免费套餐下仍可访问，因此原因始终可见。 请参阅 [Chloros+ 登录](chloros+-login.md)。

## 异常

捕获基类以处理“任何 Chloros 出错”的情况：

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

所有管道异常（`ChlorosBackendError`、`ChlorosConnectionError`、`ChlorosLicenseError`、`ChlorosAuthenticationError`、 `ChlorosConfigurationError`、`ChlorosProcessingError`）均源自 `ChlorosError`。 一个注意事项：`ChlorosConnectError` —— 仅由 `connect_camera` / `connect_array` / `connect_daq_sensor` 触发 ——源自普通的 `Exception`，**而非** `ChlorosError`，因此 `except ChlorosError` 无法捕获该异常。 完整的层次结构详见 [SDK 参考](reference/sdk-reference.md#exceptions)。

## 另请参阅

* [SDK 参考](reference/sdk-reference.md) — 完整的 API 接口，针对 AI 助手进行了优化。
* [CLI 参考](reference/cli-reference.md) — 每个 CLI 子命令都对应一个 SDK 调用。
* [下载](download.md) —— Windows 和 Linux 的安装程序。
