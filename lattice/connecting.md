# 连接相机

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>在连接任何设备之前，“摄像头”选项卡的显示</p></figcaption></figure>Chloros 会自动在链路上发现 LATTICE 摄像头——无论是通过 GUI 的“摄像头”选项卡、`chloros-cli lattice`， 或通过 Python 和 SDK 进行检测。摄像机的型号字符串将驱动后续所有操作： Chloros 会根据摄像机的 `DeviceUserID` + `DeviceSerialNumber` 数据解析传感器配置文件、波段布局和出厂校准，因此 **无需针对每台摄像机进行配置**。

连接前，请确保主机网络已配置完毕——包括链路本地寻址、巨帧以及（对于阵列）网卡接收缓冲区设置。 这些属于硬件方面的设置，相关内容详见 LATTICE 手册：[**网络设置**](https://mapir.gitbook.io/lattice-camera/setup/network-setup)。

## 通过图形用户界面 (GUI) 连接

在 Chloros 侧边栏中打开 **摄像机**选项卡（后端启动完成后，硬件选项卡才会显示），或通过主菜单 →**连接到摄像机**。这两种方式都会打开**连接摄像机** 对话框。

### “连接摄像头”对话框

该对话框在打开的瞬间会扫描网络（“正在扫描网络...”），并列出所有检测到的摄像头。 每行显示摄像头的 **型号**（例如 `LATT-M3M-L41-F550`）、**序列号**和**IP 地址**。

* **点击某一行即可选中它**（绿色高亮）。您可以选择**多台摄像头** 并一次性连接它们——Chloros 表示按顺序连接。
* 带有 **“已连接”** 标记的行表示已连接，无法再次选中。
* 带有 **“在阵列中”** 标记的行属于当前已连接的摄像头阵列。若要将该摄像头作为独立设备使用，请先断开该阵列的连接。
* **连接** — 连接所选摄像头；当选中多台时，按钮上会显示数量，例如“连接 (3)”。
* **重新扫描** — 再次执行设备发现。
* **关闭** — 关闭对话框。
* 如果扫描结束时未找到结果，对话框将显示 **“网络上未找到摄像头”** — 请参阅下方的 [故障排除](connecting.md#troubleshooting)。

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>“连接摄像头”对话框 — 此处显示的是网络上无摄像头的状态</p></figcaption></figure>### 首次连接：校准包下载

当某台摄像机**首次**连接到计算机时，Chloros 会通过 GigE 从摄像机本身获取其出厂校准包（约 3.8 MB）。 在此过程中，对话框会显示一个绿色的**“正在从摄像机下载校准数据”**面板，其中包含按序列号划分的进度条——预计每台摄像机需要大约**70 秒**。该包会被缓存到主机上，因此后续连接同一台摄像机时将完全跳过下载步骤（且不会显示该面板）。

### 分析系统

对话框中的 **分析系统** 按钮将检测主机和网络（运行时标签显示为“正在分析...”），并生成一份诊断报告：

* **主机** — CPU 核心数和内存； GPU 名称和内存，或显示“GPU：未检测到”。
* **网络接口** — 每个网卡的名称、链路速度、MTU（启用时带有“jumbo”标签）、上行/下行状态，以及是否位于 USB 总线上。
* **摄像头**— 序列号、型号、IP 地址，以及**每台摄像头连接的网卡**。
* **性能** — 每台摄像头在当前像素格式下的实际帧率与理想帧率对比；当理想帧率超过实际帧率时，会显示一条绿色的“潜力：可提升 N 倍”提示线。
* **警告和编号建议** —— 若无需修复的问题，则显示“系统状态良好，可支持当前摄像头数量”。

当设备发现或流媒体传输出现异常时，请运行此工具——它可在不离开对话框的情况下，识别出大多数网卡端的问题（MTU 设置错误、摄像头连接到错误的接口、USB 适配器限制等）。

### 连接摄像头阵列

若要将两台或更多摄像头连接为**同步阵列**，请改用阵列连接向导（**连接摄像头阵列**）： 该向导将引导您完成主从选择（由 GPIO 接线探针预先填充）、显示模式选择（独立拼图与合并拼图），并在您确认之前，通过阵列设置场景实时预览可实现的帧率（fps）和线带宽。 本手册的“多摄像头阵列”章节中介绍了该向导及阵列工作流； CLI 对应的流程是 [CLI 参考手册](../reference/cli-reference.md) 中的“LATTICE 摄像机首次连接工作流”。

## 从 CLI 和 SDK

访问 CLI 和 SDK 需要付费的 Chloros+ 层级并已登录； 此限制由服务器端强制执行（未登录时为 `401 AUTH_REQUIRED`，免费层级下为 `403 PLAN_UPGRADE_REQUIRED`）。

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

完整的签名、选项和捕获工作流：[CLI 参考](../reference/cli-reference.md) § `chloros-cli lattice`， [SDK 参考](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`。

## 连接时校准问题的解决方式

每台 LATTICE 摄像头都**内置**了出厂校准包，而且当摄像头连接时，Chloros 还会检查 MAPIR 的云端数据：

| 情况        | Chloros 使用什么                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **在线**|**该序列号发布的最新校准数据** —— 云端副本优先于相机内置副本。因此，由 MAPIR 重新校准或更新的相机将自动完成更新；无需用户操作。 |
| **离线**|**相机内的校准包**保持原样。完全离线的工作流程仍可正常运行；它们仅在相机首次联网（或进行工厂级固件重刷）后，才会采用更新的校准数据。                                                  |

在拍摄时，实际应用的系数会被**固化到每张图像的XMP元数据中**。后续的校准更新绝不会在用户不知情的情况下更改您已经拍摄的图像——重新处理旧照片时，系统会使用其XMP中记录的系数，而非当前最新的系数。

## 故障排除

* **“网络上未发现相机”**— 请验证 [网络设置](https://mapir.gitbook.io/lattice-camera/setup/network-setup) 中的链路本地配置：主机网卡静态 IP 为 `169.254.x.x/16`，相机位于同一链路上，不使用 DHCP/网关。 然后在连接对话框中使用**分析系统**功能，检查每台摄像机在哪个网卡上可见（或不可见）。进行任何线缆连接或网卡更改后，请执行**重新扫描**。
* **先前可正常工作的设备无法连接**（带有 `FRAMES WILL DROP` / `Reduce ROI to enable` 的阵列面板门）——网卡驱动程序更新已悄然重置了接收环设置。 请重新应用这些设置，或在具有管理员权限的终端中运行 `chloros-cli lattice network --fix`；请参阅 [网络设置](https://mapir.gitbook.io/lattice-camera/setup/network-setup)。
* **摄像头显示“In Array”** —— 表示该摄像头属于已连接的阵列会话。若要独立使用该摄像头，请断开阵列连接。
