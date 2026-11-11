# 如何将Chloros与AI助手结合使用

本手册面向两类读者：人类，以及人类日益依赖的 AI 助手。每页都提供了精确的数值、默认值以及可直接复制粘贴的命令，以便助手（如 Claude、ChatGPT、Copilot、编程助手等）能够一次成功编写出可运行的 Chloros 自动化脚本。

Chloros 版本：**

1.3.0**。CLI/SDK 平台：Windows 10/11 x64 和 Linux（x86_64 / Jetson aarch64）。

## 应向助手提供哪些内容

| 资源 | URL | 用途 |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | 本手册中每页的机器可读索引。 |
| **命令行界面（CLI）参考** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | 完整的 `chloros-cli` 命令集：包含所有命令、参数、默认值、退出代码及输出文件夹规则。专为大型语言模型（LLM）处理而编写。 |
| **SDK 参考** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | 完整的 `chloros_sdk` Python API：类、签名、异常及示例代码。专为大型语言模型（LLM）处理而编写。 |
| **将任意页面转换为原始 Markdown 格式** | 在页面 URL 后附加 `.md` | 例如，`https://mapir.gitbook.io/chloros/reference/sdk-reference.md` 会将页面以原始 Markdown 格式返回——非常适合粘贴到上下文窗口或由智能代理获取。 |

手册内链接：[CLI 参考](reference/cli-reference.md) · [SDK 参考](reference/sdk-reference.md)。

{% hint style="info" %}
这两个参考页面内容自成体系：阅读其中一个页面的助手无需参考手册的其他部分，即可编写正确的脚本。
{% endhint %}

## 让您的助手直接运行 Chloros：`chloros-mcp`

上述所有内容都要求您的助手*编写*脚本或命令。 借助 `chloros-mcp`，支持 [模型上下文协议](https://modelcontextprotocol.io)（MCP）的助手（例如 Claude Desktop、Claude Code 或 Cursor）可直接驱动 Chloros 运行。 您只需用自然语言提出请求，它便会调用服务器的工具：

* **图像处理：** 创建项目、导入捕获图像的文件夹、选择设置、进行处理，并报告运行结果。
* **运行硬件：**连接 LATTICE 相机、阵列和 DAQ 光传感器，预览，设置曝光和色彩，拍摄，录制，并更新固件。

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

服务器端工具涵盖了 [Python SDK](api-python-sdk.md) 的所有功能。 这些工具随SDK一同安装，因此通常无需额外安装：在Windows系统上，Chloros安装程序会将`chloros-mcp.exe`放置在您的Python的`Scripts`文件夹中， 而Linux的`.deb`会将`chloros-mcp`放置在`/usr/bin`中。

**Claude代码：**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop：**将其添加到 `claude_desktop_config.json` 中（通过**设置 &gt; 开发者 &gt; 编辑配置**可打开该文件），然后重启 Claude Desktop。**Cursor** 在 `~/.cursor/mcp.json` 中采用相同的条目。

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

如果客户端提示服务器启动失败，可能是因为 `chloros-mcp` 未出现在其识别到的 `PATH` 中： 请改用完整路径（Windows 系统中为 `where chloros-mcp`，Linux 系统中为 `which chloros-mcp`）。

您需要在同一台计算机上安装 Chloros，并在该计算机上通过应用或 `chloros-cli login` 登录任意付费的 Chloros+ 套餐（Copper 或更高）。服务器无法代您登录。

{% hint style="info" %}
各应用程序的客户端设置、选项、所有工具以及运行行为说明：请参阅 SDK 参考文档中的 [MCP 服务器（AI 助手）](reference/sdk-reference.md#mcp-server-ai-assistants)。
{% endhint %}

## 提示词模板

复制、填写 `<placeholders>`，然后粘贴到您的助手中。

### 1. 将飞行数据文件夹处理为 NDVI

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
通过命令行进行的DAQ脚本操作始终通过`daq pool-*`系列命令（`pool-connect`、`pool-list`、`pool-latest`、 `pool-stream`、`pool-record`、`pool-set-cap`、`pool-disconnect`）。 您的助手可能发明的其他 `daq` 子命令在正式发布版本中不可用，并会因错误而退出。
{% endhint %}

## 为何 AI 编写的脚本能与 Chloros 良好兼容

这些都是 Chloros 1.3.0 中真实且经过验证的行为——它们消除了机器编写自动化脚本中常见的故障模式：

* **无需繁琐的初始化步骤。**SDK 的智能连接辅助函数（`connect_camera`、`connect_array`、`connect_daq_sensor`）以及处理入口点 (`ChlorosLocal`、`process_folder`) 会**自动启动本地后端**。生成的脚本无需打开图形界面，也无需手动启动服务器——只需安装桌面版或 CLI 软件包即可。
* **整个处理流程仅需一次调用。** `chloros_sdk.process_folder("path", indices=["NDVI"])` 会端到端地执行导入 → 校准 → 反射率计算 → 折射率导出。流程更简洁，生成的脚本出错的可能性更小。
* **无输出运行会进行自诊断。** `process()` 运行结束后，运行摘要将附加到结果中，并且每条处理提示（例如 *为何*运行未产生输出）也会以Python脚本`UserWarning`的形式重新输出——因此，即使脚本从未检查结果字典，也会显示诊断信息。
* **CLI会明确报错。**请求生成结果但未写入任何数据的运行会输出`Processing finished but wrote no image products.`并**以非零退出代码退出**，因此shell脚本和CI只需通过简单的退出代码检查即可检测到该情况。 成功运行会报告 `Image products written: N`，若写入了掩码文件，还会报告 `(plus M clamp masks)`。捕获命令同样会列出所有无法写入的文件，并以 `1` 退出。
* **相对路径的解析方式取决于脚本的预期。** 使用本地后端时，相对输出路径（CLI 中为 `-o out/`，SDK 中为 `output_dir="out/"`）将以命令或脚本的运行目录为基准进行解析。

助手应了解的一点差异：SDK 中的 `process()` 在无产出运行时会刻意**不**触发异常——而是通过摘要/提示进行报告。 如果 Python 管道在空运行时必须停止，请检查摘要（示例 2 会这样做）。

## 注意事项

* **需要 Chloros+ 登录。**CLI和SDK需要**付费**的Chloros+套餐，该限制由服务器端强制执行：未登录时请求会因`401 AUTH_REQUIRED`失败，使用免费套餐时则会因`403 PLAN_UPGRADE_REQUIRED`失败。 在运行生成的脚本之前，请在每台机器上执行一次 `chloros-cli login`。请参阅 [Chloros+ 登录](chloros+-login.md)。
* **捕获命令可控制真实硬件。** `lattice` / `daq` / `project` 命令、SDK会话对象以及 `chloros-mcp` 的工具可连接、流式传输并触发物理摄像头和传感器。在首次运行生成的脚本前请先进行审查，并确保在有人值守的情况下运行该脚本。
* **对输出结果进行抽查。** 在发布结果之前，请核对产品文件夹及部分像素值。 特别需要注意的是，反射率 TIFF 文件的缩放比例因数据源而异——请查阅 `Chloros:PixelScale` 的 XMP 标签（LATTICE 文件包含该信息：32768 = 1.0 反射率； Survey3文件不包含该标签，且采用65535 = 1.0的标准），切勿自行假设除数。两份参考文档均在“读取反射率像素”部分对此进行了说明。
* **导致生成的代码出错的小陷阱：**`pool-record` 会写入**后端主机** 的文件系统（默认 `~/Documents/DAQ Live View/`）； 在拥有多个网络接口的机器上，建议优先使用 `daq pool-connect --eth-host <ip-or-hostname>` 而非自动发现；凡出现后端 URL 的位置，均应使用 `http://127.0.0.1:5000`（切勿使用 `localhost`）； 遍历项目输出树的脚本应跳过 `clamp_masks/` 文件夹（其中的 `*_clampmask.tif` 文件是掩码，而非图像——参见 [输出图像格式](output-image-formats.md#clamp-masks))；在Windows系统中若已禁用长路径功能，请保持项目文件夹名称简短——项目文件夹路径若超过约130个字符，将在处理开始前被拒绝。
