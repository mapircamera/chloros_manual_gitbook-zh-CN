# 如何在 AI 助手中使用 Chloros

本手册面向两类读者：人类，以及人类日益依赖的 AI 助手。 每页都提供了精确的数值、默认值以及可直接复制粘贴的命令，以便助手（如 Claude、ChatGPT、Copilot、编程代理等）能够一次成功编写出可运行的 Chloros 自动化脚本。

Chloros 版本：**

1.2.0**。CLI/SDK 平台： Windows 10/11 x64 以及 Linux（x86_64 / Jetson aarch64）。

## 向您的助手提供什么

| 资源 | URL | 用途 |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | 本手册中每页的机器可读索引。 |
| **CLI 参考** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | 完整的 `chloros-cli` 命令界面：包含所有命令、参数、默认值、退出代码及输出文件夹规则。 专为大型语言模型（LLM）设计。 |
| **SDK 参考手册** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | 完整的 `chloros_sdk` Python API：类、签名、异常及示例。 专为 LLM 设计。 |
| **将任意页面导出为原始 Markdown 格式** | 在页面 URL 末尾添加 `.md` | 例如 `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` 将页面以原始 Markdown 格式返回——非常适合粘贴到上下文窗口或从代理中获取。 |

手册内链接：[CLI 参考](reference/cli-reference.md) · [SDK 参考](reference/sdk-reference.md)。

{% hint style="info" %}
这两个参考页面内容自成体系：阅读过其中一个页面的助手无需参考手册的其他部分，即可编写正确的脚本。
{% endhint %}

## 提示脚本

复制、填写 `<placeholders>`，然后粘贴到您的助手中。

### 1. 将飞行文件夹处理为 NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. 批量监视捕获目录

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. 连接 LATTICE 阵列并进行捕获

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. 记录 DAQ 光传感器光谱

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
通过命令行进行的DAQ脚本操作始终通过`daq pool-*`系列命令（`pool-connect`、`pool-list`、`pool-latest`、 `pool-stream`、`pool-record`、`pool-set-cap`、`pool-disconnect`）。 您的助手可能发明的其他 `daq` 子命令在发布的版本中不可用，并会因错误而退出。
{% endhint %}

## 为什么 AI 编写的脚本能与 Chloros 良好兼容

这些都是 Chloros 1.2.0 版本中经过验证的真实行为——它们消除了机器编写自动化脚本中常见的故障模式：

* **无需繁琐的初始化步骤。**SDK 的智能连接辅助工具（`connect_camera`、`connect_array`、`connect_daq_sensor`）以及处理入口点 （`ChlorosLocal`、`process_folder`）**会自动启动本地后端**。 生成的脚本无需打开图形界面，也无需手动启动服务器——只需安装 desktop/CLI 软件包即可。
* **整个流程仅需一次调用。** `chloros_sdk.process_folder("path", indices=["NDVI"])` 将导入 → 校准 → 反射率 → 折射率导出等步骤端到端执行。处理流程更精简，生成的脚本出错的可能性更小。
* **无输出运行会进行自诊断。** 在 `process()` 之后，运行摘要将附加到结果中，且每条处理提示（例如 *为何*运行未产生输出）也会作为 Python `UserWarning` 重新输出——因此，即使脚本从未检查结果字典，也会显示诊断信息。
* **CLI 会发出明显错误提示。**一个请求生成结果但未写入任何结果的 `chloros-cli process` 运行会输出 `Processing finished but wrote no image products.` 并**以非零退出码结束**，因此 shell 脚本和 CI 只需通过简单的退出码检查即可检测到该情况。 成功运行会报告 `Image products written: N`。

助理应知晓的一点差异：SDK 的 `process()` 会在无产出运行时故意**不**触发异常——而是通过摘要/提示进行报告。 如果某个 Python 管道在空运行时必须停止，请检查摘要（配方 2 会这样做）。

## 注意事项

* **Chloros+ 需要登录。**CLI 和 SDK 需要**付费** Chloros+ 层级，该限制由服务器端强制执行：未登录时请求将返回 `401 AUTH_REQUIRED` 错误，若使用免费层级则返回 `403 PLAN_UPGRADE_REQUIRED` 错误。 在运行生成的脚本前，请在每台机器上执行一次 `chloros-cli login`。参见 [Chloros+ 登录](chloros+-login.md)。
* **捕获命令会驱动真实硬件。** `lattice` / `daq` / `project` 命令以及 SDK 会话对象用于连接、流式传输以及触发物理摄像头和传感器。 在首次运行生成的脚本之前，请先对其进行审查，并在有人值守的情况下运行该脚本。
* **对输出结果进行抽查。** 在发布结果之前，请核对产品文件夹及部分像素值。特别需要注意的是，反射率 TIFF 文件的缩放比例因光源而异——请阅读 `Chloros:PixelScale` 的 XMP 标签 （LATTICE：32768 = 1.0 反射率；Survey3：65535），而非自行假设除数。两份参考文档均在“读取反射率像素”部分对此进行了说明。
* **导致生成的代码出错的小陷阱：**`pool-record` 会写入**后端主机** 的文件系统（默认 `~/Documents/DAQ Live View/`）； 在具有多个网络接口的机器上，请优先使用 `daq pool-connect --eth-host <ip-or-hostname>` 而不是自动发现； 并且在任何出现后端 URL 的地方，请使用 `http://127.0.0.1:5000`（切勿使用 `localhost`）。
