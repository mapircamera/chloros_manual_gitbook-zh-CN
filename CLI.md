# CLI : 命令行

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** 为 Chloros 图像处理引擎提供强大的命令行访问功能，支持自动化、脚本编写及无头操作，助力您的成像工作流程。

### 核心特性

* 🚀 **自动化** - 实现多数据集批量处理脚本化
* 🔗 **集成性** - 嵌入现有工作流与数据管道
* 💻 **无界面运行** - 无需图形界面即可执行
* 🌍 **多语言支持** - 覆盖38种语言
* ⚡ **并行处理** - 动态扩展至您的CPU（最多支持16个并行工作进程）

### 系统要求

| 要求          | 详细说明                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **操作系统** | Windows 10/11 (64位)                                              |
| **许可证**          | Chloros+ ([需付费方案](https://cloud.mapir.camera/pricing)) |
| **内存**           | 最低8GB RAM（推荐16GB）                                  |
| **网络连接**         | 许可证激活必需                                     |
| **磁盘空间**       | 根据项目大小而定                                              |

{% 提示 style=&quot;warning&quot; %}
**许可证要求**：CLI 需订阅付费版 Chloros+。 标准（免费）方案不包含CLI访问权限。请访问[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)进行升级。
{% endhint %}

## 快速入门

### 安装指南

CLI已随Chloros安装程序自动包含：

1. 下载并运行**Chloros安装程序.exe**
2. 完成安装向导
3. CLI 安装路径：`C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% 提示 style=&quot;success&quot; %}
安装程序会自动将 `chloros-cli` 添加至系统 PATH 环境变量。安装完成后请重启终端。
{% endhint %}

### 初始设置

使用CLI前，请激活您的Chloros+许可证：

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### 基本用法

使用默认设置处理文件夹：

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## 命令参考

### 基本语法

```
chloros-cli [global-options] <command> [command-options]
```

***

## 命令列表

### `process` - 处理图像

使用校准处理文件夹中的图像。

**语法：**

```bash
chloros-cli process <input-folder> [options]
```

**示例：**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### 处理命令选项

| 选项                | 类型    | 默认值        | 描述                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | 路径    | _必需_     | 包含RAW/JPG多光谱图像的文件夹                                         |
| `-o, --output`        | 路径    | 与输入相同  | 处理后图像的输出文件夹                                                     |
| `-n, --project-name`  | 字符串  | 自动生成 | 自定义项目名称                                                                    |
| `--vignette`          | 标志    | 启用        | 启用晕影校正                                                             |
| `--no-vignette`       | 标志    | -              | 禁用晕影校正                                                            |
| `--reflectance`       | 标志    | 已启用        | 启用反射率校准                                                         |
| `--no-reflectance`    | 标志    | -              | 禁用反射率校准                                                        |
| `--ppk`               | 标志    | 禁用       | 应用来自.daq光传感器数据的PPK校正                                      |
| `--format`            | 选择  | TIFF (16位)  | 输出格式：`TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | 整数 | 自动           | 校准面板检测的最小目标尺寸（像素）                          |
| `--target-clustering` | 整数 | 自动           | 目标聚类阈值（0-100）                                                    |
| `--exposure-pin-1`    | 字符串 | 无           | 相机型号曝光锁定（引脚1）                                                 |
| `--exposure-pin-2`    | 字符串 | 无           | 相机型号曝光锁定（引脚2）                                                 |
| `--recal-interval`    | 整数 | 自动           | 重新校准间隔（秒）                                                      |
| `--timezone-offset`   | 整数 | 0              | 时区偏移（小时）                                                               |

***

### `login` - 账户认证

使用您的Chloros+凭证登录以启用CLI处理。

**语法：**

```bash
chloros-cli login <email> <password>
```

**示例：**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% 提示 style=&quot;warning&quot; %}
**特殊字符**：密码中若包含`$`、`!`或空格等字符，请使用单引号包裹。
{% endhint %}

**输出：**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - 清除凭证

清除存储凭证并退出账户。

**语法：**

```bash
chloros-cli logout
```

**示例：**

```powershell
chloros-cli logout
```

**输出：**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - 检查许可证状态

显示当前许可证及认证状态。

**语法：**

```bash
chloros-cli status
```

**示例：**

```powershell
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

在处理过程中或处理后监控线程4的导出进度。

**语法：**

```bash
chloros-cli export-status
```

**示例：**

```powershell
chloros-cli export-status
```

**使用场景：** 在处理运行期间调用此命令以检查导出进度。

***

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

```powershell
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### 支持语言（共38种）

| 代码    | 语言               | 原生名称      |
| ------- | --------------------- | ---------------- |
| `en`    | 英语               | English          |
| `es`    | 西班牙语         | Español          |
| `pt`    | 葡萄牙语         | Português        |
| `fr`    | 法语                | Français         |
| `de`    | 德语                | Deutsch          |
| `it`    | 意大利语               | Italiano         |
| `ja`    | 日语               | 日本語              |
| `ko`    | 韩语                | 한국어              |
| `zh`    | 简体中文             | 简体中文             |
| `zh-TW` | 繁體中文             | 繁體中文             |
| `ru`    | 俄语               | Русский          |
| `nl`    | 荷兰语                 | Nederlands       |
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
| `cs`    | 捷克语                 | Čeština          |
| `hu`    | 匈牙利语             | Magyar           |
| `ro`    | 罗马尼亚语              | Română           |
| `uk`    | 乌克兰语             | Українська       |
| `pt-BR` | 巴西葡萄牙语  | Português Brasileiro |
| `zh-HK` | 粤语             | 粤语             |
| `ms`    | 马来语                 | 马来语    |
| `sk`    | 斯洛伐克语                | 斯洛伐克语       |
| `bg`    | 保加利亚语         | Български        |
| `hr`    | 克罗地亚语         | Hrvatski         |
| `lt`    | 立陶宛语            | Lietuvių         |
| `lv`    | 拉脱维亚语         | Latviešu         |
| `et`    | 爱沙尼亚语         | Eesti            |
| `sl`    | 斯洛文尼亚语       | Slovenščina      |

{% 提示 style=&quot;success&quot; %}
**自动持久化**：您的语言偏好已保存至 `~/.chloros/cli_language.json` 并将在所有会话中持续生效。
{% endhint %}

***

### `set-project-folder` - 设置默认项目文件夹

更改默认项目文件夹位置（与GUI共享）。

**语法：**

```bash
chloros-cli set-project-folder <folder-path>
```

**示例：**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - 显示项目文件夹

显示当前默认项目文件夹位置。

**语法：**

```bash
chloros-cli get-project-folder
```

**示例：**

```powershell
chloros-cli get-project-folder
```

**输出：**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - 恢复默认值

将项目文件夹重置为默认位置。

**语法：**

```bash
chloros-cli reset-project-folder
```

***

## 全局选项

以下选项适用于所有命令：

| 选项          | 类型    | 默认值       | 说明                                                 |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | 路径    | 自动检测 | 后端可执行文件路径                       |
| `--port`        | 整数 | 5000          | 后端端口号                          |
| `--restart`     | 标志    | -             | 强制重启后端（终止现有进程） |
| `--version`     | 标志    | -             | 显示版本信息并退出                |
| `--help`        | 标志    | -             | 显示帮助信息并退出                   |

**全局选项示例：**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## 处理设置指南

### 并行处理

Chloros+ CLI **自动扩展**并行处理以匹配计算机性能：

**工作原理：**

* 检测CPU核心数与内存容量
* 分配工作进程：**2× CPU核心数**（启用超线程）
* **上限：16个并行工作进程**（保障稳定性）

**系统分级：**

| 系统类型   | CPU        | 内存      | 工作进程  | 性能     |
| ---------| | | | | | |
| **高端**  | 16+核  | 32+ GB   | 最高16个  | 极致速度   |
| **中端**   | 8-15核   | 16-31 GB   | 8-16个   | 卓越速度   |
| **低端**   | 4-7核   | 8-15 GB   | 4-8个   | 良好速度   |

{% 提示 style=&quot;success&quot; %}
**自动优化**：CLI可自动检测系统配置并配置最佳并行处理方案，无需手动设置！
{% endhint %}

### 去拜耳化算法

CLI默认采用**高品质（更快）**作为推荐去拜耳化算法：

| 方法                      | 画质 | 速度 | 描述                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **高品质（更快）** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | 边缘感知算法（默认推荐） |

### 暗角校正

**功能说明：**修正图像边缘的光线衰减（相机成像中常见的暗角现象）。

* **默认启用** - 大多数用户应保持启用状态
* 使用`--no-vignette`禁用

{% 提示 style=&quot;success&quot; %}
**建议**：始终启用暗角校正以确保画面亮度均匀。
{% endhint %}

### 反射率校准

通过校准面板将原始传感器值转换为标准化反射率百分比。

* **默认启用** - 植被分析必备功能
* 需图像中存在校准目标面板
* 使用`--no-reflectance`禁用

{%提示 style=&quot;info&quot; %}
**要求**：确保校准面板在图像中曝光准确且可见，以实现精确反射率转换。
{% endhint %}

### PPK校正

**功能：**利用DAQ-A-SD日志数据应用后处理动态校正，提升GPS精度。

* **默认禁用**
* 使用`--ppk`启用
* 需项目文件夹内包含MAPIR DAQ-A-SD光传感器生成的.daq文件。

### 输出格式

<table><thead><tr><th width="197">格式</th><th width="130.20001220703125">位深度</th><th width="116.5999755859375">文件大小</th><th>最佳适用场景</th></tr></thead><tbody><tr><td><strong>TIFF (16位)</strong> ⭐</td><td>16位整数</td><td>大</td><td>GIS分析、摄影测量（推荐）</td></tr><tr><td><strong>TIFF（32位，百分比）</strong></td><td>32位浮点</td><td>超大</td><td>科学分析、研究</td></tr><tr><td><strong>PNG（8位）</strong></td><td>8 位整数</td><td>中等</td><td>目视检查、网络共享</td></tr><tr><td><strong>JPG（8位）</strong></td><td>8位整数</td><td>小</td><td>快速预览，压缩输出</td></tr></tbody></table>***

## 自动化与脚本编写

### PowerShell批量处理

自动处理多个数据集文件夹：

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

### Windows批处理脚本

批量处理的简单循环：

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

### Python自动化脚本

带错误处理的高级自动化：

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

1. **输入**：包含RAW/JPG图像对的文件夹
2. **检测**：CLI自动扫描支持的图像文件
3. **处理**：并行模式可扩展至CPU核心数量（Chloros+）
4. **输出**：创建相机型号子文件夹并存放处理后的图像

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

100张图像（每张1200万像素）典型处理时长：

| 模式              | 时间      | 硬件                                     |
| ----------------- | --------- | -------------------------------------------- |
| **并行模式** | 5-10 分钟 | i7/锐龙7处理器，16GB内存，SSD（最多16个工作进程） |
| **并行模式** | 10-15 分钟 | i5/锐龙5处理器，8GB内存，机械硬盘（最多8个工作进程） |

{% 提示 style=&quot;info&quot; %}
**性能提示**：处理时间因图像数量、分辨率及计算机配置而异。
{% endhint %}

***

## 故障排除

### 未找到CLI

**错误：**

```
'chloros-cli' is not recognized as an internal or external command
```

**解决方案：**

1. 验证安装路径：

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. 若未添加至PATH环境变量，请使用完整路径：

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. 手动添加至PATH环境变量：
   * 打开系统属性 → 环境变量
   * 编辑PATH变量
   * 添加：`C:\Program Files\Chloros\resources\cli`
   * 重启终端

***

### 后端启动失败

**错误：**

```
Backend failed to start within 30 seconds
```

**解决方案：**

1. 检查后端是否已运行（先关闭）
2. 检查防火墙是否阻断
3. 尝试使用其他端口：

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. 强制重启后端：

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### 许可证/认证问题

**错误：**

```
Chloros+ license required for CLI access
```

**解决方案：**

1. 确认您拥有有效的 Chloros+ 订阅
2. 使用凭证登录：

```powershell
chloros-cli login user@example.com 'password'
```

3. 检查许可证状态：

```powershell
chloros-cli status
```

4. 联系支持：info@mapir.camera

***

### 未找到图像文件

**错误：**

```
No images found in the specified folder
```

**解决方案：**

1. 确认文件夹内包含支持的格式（.RAW, .TIF, .JPG）
2. 检查文件夹路径是否正确（路径含空格时请使用引号）
3. 确保您对文件夹拥有读取权限
4. 检查文件扩展名是否正确

***

### 处理卡顿或挂起

**解决方案：**

1. 检查可用磁盘空间（确保足够存储输出文件）
2. 关闭其他应用程序以释放内存
3. 减少图像数量（分批处理）

***

### 端口已被占用

**错误代码：**

```
Port 5000 is already in use
```

**解决方案：**

指定其他端口：

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## 常见问题

### 问：CLI是否需要许可证？

**答：**需要！CLI需付费获取**Chloros+许可证**。

* ❌ 标准（免费）方案：CLI禁用
* ✅ Chloros+（付费）方案：CLI功能完全启用

订阅地址：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### 问：能否在无GUI的服务器上使用CLI？

**答：**可以！CLI支持完全无头运行。要求：

* Windows Server 2016或更高版本
* 已安装Visual C++再发行包
* 充足内存（最低8GB，推荐16GB）
* 任意设备上的一次性GUI许可证激活

***

### 问：处理后的图像保存在何处？

**答：**默认情况下，处理后的图像保存在**与输入文件相同的目录**下，分类于相机型号子文件夹中（例如`Survey3N_RGN/`）。

使用`-o`选项可指定其他输出文件夹：

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### 问：能否同时处理多个文件夹？

**A:** 无法通过单条命令直接实现，但可通过脚本实现文件夹顺序处理。详见[自动化与脚本](CLI.md#automation--scripting)章节。

***

### Q: 如何将CLI输出保存至日志文件？

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**批处理：**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### 问：处理过程中按下Ctrl+C会怎样？

**答：** CLI将执行：

1. 优雅终止处理
2. 关闭后端进程
3. 以状态码130退出

部分处理的图像可能仍保留在输出文件夹中。

***

### 问：能否自动化CLI处理？

**答：**当然可以！CLI专为自动化设计。 请参阅[自动化与脚本](CLI.md#automation--scripting)获取PowerShell、批处理及Python示例。

***

### 问：如何检查CLI版本？

**答：**

```powershell
chloros-cli --version
```

**输出：**

```
Chloros CLI 1.0.2
```

***

## 获取帮助

### 命令行帮助

在CLI中直接查看帮助信息：

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### 支持渠道

* **电子邮件**：info@mapir.camera
* **官网**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **定价**：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## 完整示例

### 示例 1：基础处理

采用默认设置处理（晕影、反射率）：

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### 示例 2：高品质科研输出

32位浮点TIFF：

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### 示例3：快速预览处理

8位PNG（无校准）用于快速审阅：

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### 示例4：PPK校正处理

应用反射率的PPK校正：

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### 示例5：自定义输出位置

以特定格式处理至不同驱动器：

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### 示例6：认证工作流

完成完整认证流程：

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 示例7：多语言使用

更改界面语言：

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
