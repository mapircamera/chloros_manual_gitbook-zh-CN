# CLI：命令行

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** 提供了对 Chloros 图像处理引擎的强大命令行访问功能，可为您的图像处理工作流实现自动化、脚本化和无头操作。

### 主要功能

* 🚀 **自动化** - 通过脚本批量处理多个数据集
* 🔗 **集成** - 嵌入现有工作流和管道
* 💻 **无头运行** - 无需图形界面即可运行
* 🌍 **多语言支持** - 支持 38 种语言
* ⚡ **并行处理** - [动态计算适配](processing-architecture/dynamic-compute-adaptation.md) 可根据您的硬件自动进行优化

### 系统要求

| 要求                        | 详细信息                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **操作系统** | Windows 10/11 (64 位), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **许可证**          | Chloros+ ([需付费计划](https://cloud.mapir.camera/pricing)) |
| **内存**           | 至少 8GB RAM（建议 16GB）                                  |
| **网络**         | 许可证激活必备                                     |
| **磁盘空间**       | 视项目大小而定                                              |

{% hint style="warning" %}
**许可证要求**：CLI 需要付费的 Chloros+ 订阅。 标准（免费）套餐不包含 CLI 访问权限。请访问 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) 进行升级。
{% endhint %}

## 快速入门

### 安装

#### Windows

CLI 已自动包含在 Chloros 安装程序中：

1. 下载并运行 **Chloros Installer.exe**

2. 完成安装向导
3. CLI 安装路径：`C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
安装程序会自动将 `chloros-cli` 添加到系统 PATH 中。安装完成后请重启终端。
{% endhint %}

#### Linux

请安装适用于您系统架构的 `.deb` 软件包：

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

有关 Linux 的详细设置，请参阅 [Linux 安装](linux/linux-installation.md)。

### 首次设置

在使用 CLI 之前，请激活您的 Chloros+ 许可证：

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### 基本用法

使用默认设置处理文件夹：

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## 命令参考

### 通用语法

```
chloros-cli [global-options] <command> [command-options]
```

***

## 命令

### `process` - 处理图像

对文件夹中的图像进行校准处理。

**语法：**

```bash
chloros-cli process <input-folder> [options]
```

**示例：**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### 处理命令选项

| 选项                | 类型    | 默认值        | 描述                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | 路径    | _必填_     | 包含 RAW/JPG 多光谱图像的文件夹                                         |
| `-o, --output`        | 路径    | 与输入相同  | 处理后图像的输出文件夹                                                     |
| `-n, --project-name`  | 字符串  | 自动生成 | 自定义项目名称                                                                    |
| `--vignette`          | 标志    | 已启用        | 启用暗角校正                                                             |
| `--no-vignette`       | 标志    | -              | 禁用暗角校正                                                            |
| `--reflectance`       | 标志    | 已启用        | 启用反射率校准                                                         |
| `--no-reflectance`    | 标志    | -              | 禁用反射率校准                                                        |
| `--ppk`               | 标志    | 禁用       | 应用来自 .daq 光传感器数据的 PPK 校正                                      |
| `--format`            | 选项  | TIFF (16 位)  | 输出格式：`TIFF (16-bit)`、`TIFF (32-bit, Percent)`、`PNG (8-bit)`、`JPG (8-bit)` |
| `--min-target-size`   | 整数 | 自动           | 校准面板检测的最小目标像素尺寸                          |
| `--target-clustering` | 整数 | 自动           | 目标聚类阈值 (0-100)                                                    |
| `--debayer`           | 选项  | `standard`     | 去拜耳化方法：`standard` 或 `texture-aware`（仅限 Chloros+）                          |
| `--target`, `--targets` | 标志  | 禁用       | 仅在“target”或“targets”子文件夹中搜索校准目标（加快处理速度） |
| `--indices`           | 列表    | 无           | 待计算的植被指数（例如：`--indices NDVI NDRE GNDVI`）                    |
| `--exposure-pin-1`    | 字符串  | 无           | 锁定相机型号的曝光参数（引脚 1）                                                 |
| `--exposure-pin-2`    | 字符串  | 无           | 相机型号的曝光锁定（引脚 2）                                                 |
| `--recal-interval`    | 整数 | 自动           | 重新校准间隔（秒）                                                      |
| `--timezone-offset`   | 整数 | 0              | 时区偏移（小时）                                                               |

***

### `login` - 账户认证

请使用您的 Chloros+ 凭据登录，以启用 CLI 处理。

**语法：**

```bash
chloros-cli login <email> <password>
```

**示例：**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**特殊字符**：若密码包含 `$`、`!` 或空格等字符，请使用单引号将其括起来。
{% endhint %}

**输出：**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - 清除凭据

清除存储的凭据并退出您的账户。

**语法：**

```bash
chloros-cli logout
```

**示例：**

```bash
chloros-cli logout
```

**输出：**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK 用户**：Python SDK 还提供了一种编程方法，可在 Python 脚本中清除凭据。 详情请参阅 [Python SDK 文档](api-python-sdk.md#logout)。
{% endhint %}

***

### `status` - 检查许可证状态

显示当前许可证和身份验证状态。

**语法：**

```bash
chloros-cli status
```

**示例：**

```bash
chloros-cli status
```

**输出：**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - 检查导出进度

在处理过程中或处理结束后监控线程 4 的导出进度。

**语法：**

```bash
chloros-cli export-status
```

**示例：**

```bash
chloros-cli export-status
```

**用例：** 在处理运行期间调用此命令以检查导出进度。***

### `language` - 管理界面语言

查看或更改 CLI 界面语言。

**语法：**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**示例：**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### 支持的语言（共 38 种）

| 代码    | 语言              | 本地名称      |
| ------- | --------------------- | ---------------- |
| `en`    | 英语               | English          |
| `es`    | 西班牙语               | Español          |
| `pt`    | 葡萄牙语            | Português        |
| `fr`    | 法语                | Français         |
| `de`    | 德语                | Deutsch          |
| `it`    | 意大利语              | Italiano         |
| `ja`    | 日语              | 日本語              |
| `ko`    | 韩语                | 한국어              |
| `zh`    | 简体中文        | 简体中文             |
| `zh-TW` | 繁体中文        | 繁體中文             |
| `ru`    | 俄语               | Русский          |
| `nl`    | 荷兰语                | Nederlands       |
| `ar`    | 阿拉伯语                | العربية          |
| `pl`    | 波兰语                | Polski           |
| `tr`    | 土耳其语               | Türkçe           |
| `hi`    | 印地语                 | हिंदी            |
| `id`    | 印尼语            | Bahasa Indonesia |
| `vi`    | 越南语            | Tiếng Việt       |
| `th`    | 泰语                  | ไทย              |
| `sv`    | 瑞典语               | Svenska          |
| `da`    | 丹麦语                | Dansk            |
| `no`    | 挪威语             | Norsk            |
| `fi`    | 芬兰语               | Suomi            |
| `el`    | 希腊语                 | Ελληνικά         |
| `cs`    | 捷克语                | Čeština          |
| `hu`    | 匈牙利语             | Magyar           |
| `ro`    | 罗马尼亚语              | Română           |
| `uk`    | 乌克兰语             | Українська       |
| `pt-BR` | 巴西葡萄牙语  | Português Brasileiro |
| `zh-HK` | 粤语             | 粵語             |
| `ms`    | 马来语                 | Bahasa Melayu    |
| `sk`    | 斯洛伐克语                | Slovenčina       |
| `bg`    | 保加利亚语             | Български        |
| `hr`    | 克罗地亚语              | Hrvatski         |
| `lt`    | 立陶宛语            | Lietuvių         |
| `lv`    | 拉脱维亚语               | Latviešu         |
| `et`    | 爱沙尼亚语              | Eesti            |
| `sl`    | 斯洛文尼亚语             | Slovenščina      |

{% hint style="success" %}
**自动保存**：您的语言偏好设置将保存至 `~/.chloros/cli_language.json`，并在所有会话中保持有效。
{% endhint %}

***

### `set-project-folder` - 设置默认项目文件夹

更改默认项目文件夹位置（与 Windows 中的 GUI 共享）。

**语法：**

```bash
chloros-cli set-project-folder <folder-path>
```

**示例：**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - 显示项目文件夹

显示当前默认项目文件夹的位置。

**语法：**

```bash
chloros-cli get-project-folder
```

**示例：**

```bash
chloros-cli get-project-folder
```

**输出：**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - 恢复默认设置

将项目文件夹重置为默认位置。

**语法：**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - 运行系统诊断

运行 7 项诊断检查以验证系统配置。

**语法：**

```bash
chloros-cli selftest
```

**执行的诊断项目：**

1. 版本检查
2. 端口可用性 (5000)
3. 后端启动
4. API 连接性测试
5. 系统信息和 GPU 检测
6. 降噪模型验证
7. CUDA 可用性检查

{% hint style="info" %}
**故障排除提示**：安装完成后运行 `selftest` 以验证系统配置是否正确，特别是在 Linux/Jetson 平台上，可能需要验证 GPU 和 CUDA 的设置。
{% endhint %}

***

### `update` - 检查更新（仅限 Linux）

在 Linux 系统上检查并安装 CLI 更新。

**语法：**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| 选项    | 描述                        |
| --------- | ---------------------------------- |
| `--check` | 仅检查更新，不安装 |

{% hint style="info" %}
此命令仅在 Linux 系统上可用。在 Windows 系统上，更新通过安装程序提供。
{% endhint %}

***

## 全局选项

以下选项适用于所有命令：

| 选项            | 类型    | 默认值       | 描述                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | 路径    | 自动检测 | 后端可执行文件的路径                       |
| `--port`          | 整数 | 5000          | 后端 API 端口号                          |
| `--restart`       | 标志    | -             | 强制重启后端（终止现有进程） |
| `--version`       | 标志    | -             | 显示版本信息并退出                |
| `--help`          | 标志    | -             | 显示帮助信息并退出                   |

{% hint style="info" %}
**后端自动检测**：`--backend-exe` 路径会根据平台自动检测：
* **Windows**：`C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (手动)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**全局选项示例：**

**Windows：**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux：**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## 处理设置指南

### 并行处理与动态计算适配

Chloros 1.1.0 包含 [动态计算适配](processing-architecture/dynamic-compute-adaptation.md) — 处理引擎会 **自动检测您的硬件** 并选择最佳策略：

| 平台 | 策略 | 工作线程 | 管道 | 备注 |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | 内存高效，串行化 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | 并行 GPU 处理 |
| **配备 8GB GPU 的台式机** | `GPU_SINGLE` | 3 | `tiled_gpu` | 良好的台式机性能 |
| **配备 12GB+ GPU 的台式机** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | 最佳台式机性能 |
| **仅CPU系统** | `CPU_PARALLEL` | 核心数 - 1 | `cpu_fallback` | 无需GPU |

{% hint style="success" %}
**无需手动配置！** Chloros 会自动检测您的 CPU、GPU、内存以及（在 Jetson 上）温度传感器，然后自动配置最佳处理流程。
{% endhint %}

### 去拜耳滤波方法

| 方法 | CLI 标志 | 质量 | 速度 | 许可证 |
| --- | --- | --- | --- | --- |
| **标准 (快速，中等质量)** | `--debayer standard` | 良好 | 快速 | 免费 / Chloros+ |
| **纹理感知 (慢速，最高质量)** | `--debayer texture-aware` | 最高 | 慢速 | 仅限 Chloros+ |

默认去拜耳化方法为 **标准**。**纹理感知**方法采用 AI/ML 降噪模型以获得最高质量的输出，但需要 Chloros+ 许可证和 NVIDIA GPU。

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### 暗角校正

**功能说明：** 校正图像边缘的光线衰减（相机图像中常见的暗角现象）。

* **默认启用** - 大多数用户应保持此功能启用
* 使用 `--no-vignette` 禁用

{% hint style="success" %}
**建议**：始终启用暗角校正，以确保整个画面的亮度均匀。
{% endhint %}

### 反射率校准

利用校准板将原始传感器值转换为标准化的反射率百分比。

* **默认启用** - 植被分析必不可少
* 图像中需包含校准目标板
* 使用 `--no-reflectance` 禁用

{% hint style="info" %}
**要求**：确保校准面板在图像中曝光正确且可见，以实现准确的反射率转换。
{% endhint %}

### PPK 校正

**功能说明：** 利用 DAQ-A-SD 日志数据应用后处理动态校正，以提高 GPS 精度。

* **默认禁用**
* 使用 `--ppk` 启用
* 项目文件夹中需包含来自 MAPIR DAQ-A-SD 光传感器的 .daq 文件。

### 输出格式

<table><thead><tr><th width="197">格式</th><th width="130.20001220703125">位深度</th><th width="116.5999755859375">文件大小</th><th>最适合</th></tr></thead><tbody><tr><td><strong>TIFF (16位)</strong> ⭐</td><td>16 位整数</td><td>大</td><td>GIS 分析、摄影测量（推荐）</td></tr><tr><td><strong>TIFF (32 位，百分比)</strong></td><td>32 位浮点数</td><td>超大</td><td>科学分析、研究</td></tr><tr><td><strong>PNG (8 位)</strong></td><td>8 位整数</td><td>中</td><td>目视检查、网络共享</td></tr><tr><td><strong>JPG (8 位)</strong></td><td>8 位整数</td><td>小</td><td>快速预览、压缩输出</td></tr></tbody></table>***

## 自动化与脚本编写

### PowerShell 批处理 (Windows)

在 Windows 上自动处理多个数据集文件夹：

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows 批处理脚本 (Windows)

Windows 上的简单循环批处理：

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash 批处理 (Linux)

在 Linux 上处理多个数据集文件夹：

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python 自动化脚本（跨平台）

带错误处理的高级自动化（适用于 Windows 和 Linux）：

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## 处理工作流

### 标准工作流

1. **输入**：包含 RAW/JPG 图像对的文件夹
2. **检测**：CLI 自动扫描支持的图像文件
3. **处理**：并行模式根据您的 CPU 核心数进行扩展（Chloros+）
4. **输出**：创建包含处理后图像的相机型号子文件夹

### 输出结构示例

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### 处理时间估算

100 张图像（每张 12MP）的典型处理时间：

| 平台 | 模式 | 预计时间 | 备注 |
| --- | --- | --- | --- |
| **台式机 12GB+ GPU** | `GPU_PARALLEL` | 5-10 分钟 | 最快选项 |
| **台式机 8GB GPU** | `GPU_SINGLE` | 10-15 分钟 | 性能良好 |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 15-25 分钟 | 边缘计算 |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 30-60 分钟 | 内存受限 |
| **仅 CPU** | `CPU_PARALLEL` | 20-40 分钟 | 无需 GPU |

{% hint style="info" %}
**性能提示**：处理时间因图像数量、分辨率、去拜耳化方法和硬件而异。纹理感知去拜耳化比标准去拜耳化耗时显著更长。详情请参阅 [动态计算自适应](processing-architecture/dynamic-compute-adaptation.md)。
{% endhint %}

***

## 故障排除

### 未找到 CLI

**Windows 错误：**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows 解决方案：**

1. 验证安装位置：

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. 若未在 PATH 中，请使用完整路径：

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. 手动添加到 PATH：
   * 打开系统属性 → 环境变量
   * 编辑 PATH 变量
   * 添加：`C:\Program Files\Chloros\resources\cli`
   * 重启终端

**Linux 错误：**

```
chloros-cli: command not found
```

**Linux 解决方案：**

1. 验证安装：

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. 重新加载 shell：

```bash
source ~/.bashrc
```

3. 检查权限：

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### 后端启动失败**错误：**

```

Backend failed to start within 30 seconds
```

**解决方案：**

1. 检查后端是否已运行（先关闭它）
2. 检查防火墙是否阻挡（Windows）或检查端口可用性（Linux: `lsof -i :5000`）
3. 尝试其他端口：

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. 强制重启后端：

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. 在 Linux 上，检查后端可执行文件是否存在：

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### 许可证/身份验证问题**错误：**

```

Chloros+ license required for CLI access
```

**解决方案：**

1. 确认您拥有有效的 Chloros+ 订阅
2. 使用您的凭据登录：

```bash
chloros-cli login user@example.com 'password'
```

3. 检查许可证状态：

```bash
chloros-cli status
```

4. 联系支持：info@mapir.camera

***

### 未找到图像**错误：**

```

No images found in the specified folder
```

**解决方案：**

1. 确认文件夹中包含受支持的格式（.RAW、.TIF、.JPG）
2. 检查文件夹路径是否正确（路径中含空格时请使用引号）
3. 确保您对该文件夹具有读取权限
4. 检查文件扩展名是否正确

***

### 处理卡顿或死机**解决方案：**

1. 检查可用磁盘空间（确保有足够空间存储输出文件）
2. 关闭其他应用程序以释放内存
3. 减少图像数量（分批处理）

***

### 端口已被占用**错误：**

```

Port 5000 is already in use
```

**解决方案：**

**Windows：**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux：**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## 常见问题

### 问：使用 CLI 需要许可证吗？

**答：**需要！CLI 需要付费的**Chloros+ 许可证**。

* ❌ 标准（免费）套餐：CLI 已禁用
* ✅ Chloros+（付费）套餐：CLI 已完全启用

订阅地址：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### 问：我可以在没有图形界面的服务器上使用 CLI 吗？**答：** 可以！CLI 完全支持无头运行。这是 Linux 的主要使用场景。**Windows 服务器：**
* Windows Server 2016 或更高版本
* 已安装 Visual C++ 再分发包

**Linux 服务器：**
* Ubuntu 20.04+ / Debian 11+ (amd64) 或 JetPack 6 (arm64)
* 通过 `.deb` 包安装

**两平台共通：**
* 至少 8GB 内存 （推荐 16GB）
* 一次性许可证激活：`chloros-cli login user@example.com 'password'`

***

### 问：处理后的图像保存在哪里？**答：**默认情况下，处理后的图像保存在与输入文件相同的文件夹中，位于相机型号的子文件夹内（例如：`Survey3N_RGN/`）。

使用 `-o` 选项指定其他输出文件夹：

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### 问：能否同时处理多个文件夹？**A:** 无法通过单条命令直接实现，但您可以通过脚本依次处理文件夹。请参阅 [自动化与脚本](CLI.md#automation--scripting) 章节。***

### Q: 如何将 CLI 的输出保存到日志文件中？**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**批处理：**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash：**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### 问：处理过程中按下 Ctrl+C 会发生什么？**A:** CLI 将：

1. 正常停止处理
2. 关闭后端
3. 以代码 130 退出

部分处理过的图像可能会保留在输出文件夹中。

***

### Q: 我可以自动化 CLI 的处理吗？**A:** 当然可以！CLI 专为自动化设计。请参阅 [自动化与脚本](CLI.md#automation--scripting) 以了解 PowerShell (Windows)、 批处理 (Windows)、Bash (Linux) 以及 Python（跨平台）示例。***

### 问：如何查看 CLI 的版本？**答：**

```bash
chloros-cli --version
```

**输出：**

```

Chloros CLI 1.1.0
```

***

## 获取帮助

### 命令行帮助

直接在 CLI 中查看帮助信息：

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### 支持渠道

* **电子邮件**：info@mapir.camera
* **网站**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **定价**：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## 完整示例

### 示例 1：基本处理

使用默认设置进行处理（晕影、反射率）：

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### 示例 2：高质量科学输出

32 位浮点数 TIFF:**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### 示例 3：快速预览处理

8 位 PNG（未校准，用于快速审查）：

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### 示例 4：PPK 校正处理

应用基于反射率的 PPK 校正：

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### 示例 5：自定义输出位置

以特定格式处理并输出到不同位置：

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### 示例 6：身份验证工作流

完整的身份验证流程（所有平台均相同）：

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 示例 7：多语言使用

更改界面语言（所有平台均相同）：

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
