# API : Python SDK

**Chloros Python SDK** 提供了对 Chloros 图像处理引擎的编程访问，支持自动化、自定义工作流，并可与您的 Python 应用程序及研究流程无缝集成。

### 主要功能

* 🐍 **原生 Python** - 简洁、符合 Python 风格的 API 图像处理
* 🔧 **全面访问 API** - 对 Chloros 处理拥有完全控制权
* 🚀 **自动化** - 构建自定义批处理工作流
* 🔗 **集成** - 将 Chloros 嵌入现有 Python 应用程序
* 📊 **科研就绪** - 非常适合科学分析流程
* ⚡ **并行处理** - 根据您的 CPU 核心数进行扩展 (Chloros+)

### 系统要求

| 要求                        | 详细信息                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **已安装 Chloros** | Windows：桌面安装程序； Linux：`.deb` 软件包                  |
| **许可证**          | Chloros+ ([需付费套餐](https://cloud.mapir.camera/pricing)) |
| **操作系统** | Windows 10/11 (64 位), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 或更高版本                                                |
| **内存**           | 至少 8GB RAM（建议 16GB）                                  |
| **互联网**         | 许可证激活所需                                     |

{% hint style="warning" %}
**许可证要求**：Python SDK 需要付费的 Chloros+ 订阅才能访问 API。 标准（免费）套餐不包含 API/SDK 访问权限。请访问 [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) 进行升级。
{% endhint %}

## 快速入门

### 安装

通过 pip 安装：

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**首次设置**：在使用 SDK 之前，请通过打开 Chloros、Chloros （浏览器）或 Chloros CLI，并使用您的凭据登录。此操作仅需执行一次。在 Linux（无图形界面）上，请使用：`chloros-cli login user@example.com 'password'`
{% endhint %}

### 基本用法

仅需几行代码即可处理文件夹：

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**跨平台路径**：本页面的代码示例使用 Windows 格式的路径（例如 `C:\\DroneImages\\Flight001`）。 在 Linux 上，请改用 Linux 格式的路径（例如 `/home/user/drone_images/flight001` 或 `~/drone_images/flight001`）。SDK 在两个平台上的工作原理完全相同。
{% endhint %}

### 完全控制

适用于高级工作流：

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## 安装指南

### 先决条件

在安装 SDK 之前，请确保您已具备：

1. **已安装 Chloros** — Windows：桌面安装程序 ([下载](download.md))； Linux：`.deb` 软件包 ([Linux 安装](linux/linux-installation.md))
2. **Python 3.7+** 已安装 ([python.org](https://www.python.org))
3. **有效的 Chloros+ 许可证** ([升级](https://cloud.mapir.camera/pricing))

### 通过 pip 安装

**标准安装：**

```bash
pip install chloros-sdk
```

**支持进度监控：**

```bash
pip install chloros-sdk[progress]
```

**开发版安装：**

```bash
pip install chloros-sdk[dev]
```

### 验证安装

测试 SDK 是否安装正确：

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## 首次设置

### 许可证激活

SDK 与 Chloros、Chloros（浏览器版）以及 Chloros CLI 使用相同的许可证。 请通过图形界面或 CLI 进行一次激活：

**Windows：**打开**Chloros 或 Chloros（浏览器）**，并在“用户” <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 选项卡登录，或使用 CLI。**Linux：** 使用 CLI（无 GUI 可用）：

```bash
chloros-cli login user@example.com 'your_password'
```

许可证将缓存于本地，并在重启后保持有效。

{% hint style="success" %}
**一次性设置**：通过图形界面或 CLI 登录后，SDK 将自动使用缓存的许可证。无需额外认证！
{% endhint %}

{% hint style="info" %}
**注销**：SDK用户可通过`logout()`方法以编程方式清除缓存凭据。请参阅API参考文档中的[logout()方法](#logout)。
{% endhint %}

### 测试连接

验证 SDK 能否连接到 Chloros：

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API 参考文档

### ChlorosLocal 类

用于本地 Chloros 图像处理的主类。

#### 构造函数

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**参数：**

| 参数                 | 类型 | 默认值                   | 描述                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | 本地 Chloros 后端的 URL 实例          |
| `auto_start_backend`      | bool | `True`                    | 如有需要自动启动后端 |
| `backend_exe`             | 字符串  | `None` (自动检测)      | 后端可执行文件的路径            |
| `timeout`                 | 整数  | `30`                      | 请求超时时间（秒）            |
| `backend_startup_timeout` | int  | `60`                      | 后端启动超时（秒） |

**示例：**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**跨平台自动检测**： SDK 会自动尝试适用于您平台的正确后端路径：
* **Windows**：`C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**：`/usr/lib/chloros/chloros-backend`
* **Linux (手动)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### 方法

#### `create_project(project_name, camera=None)`

创建一个新的 Chloros 项目。

**参数：**

| 参数      | 类型 | 必填 | 描述                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | 字符串 | 是      | 项目名称                                     |
| `camera`       | 字符串  | 否       | 相机模板（例如，“Survey3N\_RGN”、“Survey3W\_OCN”） |

**返回值：** `dict` - 项目创建响应**示例：**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

从文件夹导入图像。

**参数：**

| 参数         | 类型     | 必填 | 描述                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | 是      | 包含图片的文件夹路径         |
| `recursive`   | bool     | 否       | 搜索子文件夹 (默认：False) |

**返回值：** `dict` - 包含文件数量的导入结果**示例：**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

配置处理设置。

**参数：**

| 参数                 | 类型 | 默认值                 | 描述                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | 字符串  | &quot;标准（快速，中等质量）&quot; | 去拜耳化方法            |
| `vignette_correction`     | 布尔值 | `True`                  | 启用暗角校正      |
| `reflectance_calibration` | bool | `True`                  | 启用反射率校准  |
| `indices`                 | list | `None`                  | 待计算的植被指数 |
| `export_format`           | 字符串  | &quot;TIFF (16位)&quot;         | 输出格式                   |
| `ppk`                     | 布尔值 | `False`                 | 启用PPK校正          |
| `custom_settings`         | 字典 | `None`                  | 高级自定义设置        |

**导出格式：**

* `"TIFF (16-bit)"` - 推荐用于GIS/摄影测量
* `"TIFF (32-bit, Percent)"` - 科学分析
* `"PNG (8-bit)"` - 目视检查
* `"JPG (8-bit)"` - 压缩输出

**可用索引：**NDVI, NDRE, GNDVI, OSAVI、CIG、EVI、SAVI、MSAVI、MTVI2 等。**示例：**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

处理项目图像。

**参数：**

| 参数           | 类型     | 默认值      | 描述                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | 字符串      | `"parallel"` | 处理模式：&quot;并行&quot; 或 &quot;串行&quot;   |
| `wait`              | bool     | `True`       | 等待处理完成                       |
| `progress_callback` | callable | `None`       | 进度回调函数(progress, msg) |
| `poll_interval`     | float    | `2.0`        | 进度轮询间隔（秒）   |

**返回值：** `dict` - 处理结果

{% hint style="warning" %}
**并行模式**：需要 Chloros+ 许可证。自动扩展至您的 CPU 核心（最多 16 个工作线程）。
{% endhint %}

**示例：**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

获取当前项目配置。

**返回值：** `dict` - 当前项目配置**示例：**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

获取后端状态信息，包括每个线程的处理进度。

**返回值：** `dict` - 后端状态，结构如下：

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**示例：**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

关闭后端（若由 SDK 启动）。

**示例：**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

清除本地系统中的缓存凭据。

**描述：**

通过移除缓存的身份验证凭据来实现程序化注销。这适用于：
* 在不同的 Chloros+ 账户之间切换
* 在自动化环境中清除凭据
* 安全目的（例如，在卸载前移除凭据）

**返回值：** `dict` - 注销操作结果**示例：**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**需重新认证**：调用 `logout()` 后，必须通过 Chloros、 Chloros（浏览器）或 Chloros CLI 进行重新登录，然后才能使用 SDK。
{% endhint %}

***

### 便捷函数

#### `process_folder(folder_path, **options)`

用于处理文件夹的一行便捷函数。

**参数：**

| 参数                 | 类型     | 默认值         | 描述                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | 必填        | 包含图像的文件夹路径     |
| `project_name`            | str      | 自动生成  | 项目名称                   |
| `camera`                  | 字符串      | `None`          | 相机模板                |
| `indices`                 | 列表     | `["NDVI"]`      | 待计算的索引         |
| `vignette_correction`     | 布尔     | `True`          | 启用暗角校正     |
| `reflectance_calibration` | 布尔值     | `True`          | 启用反射率校准 |
| `export_format`           | 字符串      | &quot;TIFF (16-bit)&quot; | 输出格式                  |
| `mode`                    | 字符串      | `"parallel"`    | 处理模式                |
| `progress_callback`       | 可调用     | `None`          | 进度回调              |

**返回值：** `dict` - 处理结果**示例：**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## 上下文管理器支持

SDK 支持上下文管理器以实现自动清理：

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## 完整示例

{% hint style="info" %}
**Linux 用户**： 以下所有示例均使用 Windows 路径。请将 `C:\\...` 路径替换为您自己的 Linux 路径（例如 `/home/user/...` 或 `~/...`）。 所有 SDK 功能在各平台上的表现完全一致。
{% endhint %}

### 示例 1：基本处理

使用默认设置处理文件夹：

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### 示例 2：自定义工作流

完全掌控处理流程：

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### 示例 3：批量处理多个文件夹

处理多个飞行数据集：

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### 示例 4：研究流程集成

将 Chloros 与数据分析集成：

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### 示例 5：自定义进度监控

通过日志记录实现高级进度跟踪：

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### 示例 6：错误处理

适用于生产环境的健壮错误处理：

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### 示例 7：账户管理与注销

通过编程方式管理凭据：

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### 示例 8：命令行工具

使用 SDK 构建自定义 CLI 工具：

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**用法：**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## 异常处理

SDK 为不同类型的错误提供了专门的异常类：

### 异常层次结构

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### 异常示例

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## 高级主题

### 自定义后端配置

使用自定义后端位置或配置：

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### 非阻塞处理

启动处理并继续执行其他任务：

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### 内存管理

对于大型数据集，请分批处理：

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## 故障排除

### 后端无法启动

**问题：** SDK 无法启动后端**解决方案：**

1. 验证是否已安装 Chloros：

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. 检查防火墙（Windows）或端口可用性（Linux：`lsof -i :5000`）
3. 尝试手动指定后端路径：

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### 未检测到许可证**问题：** SDK 提示缺少许可证**解决方案：**

1. 打开 Chloros、Chloros（浏览器）或 Chloros CLI 并登录。
2. 验证许可证是否已缓存：

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. 若遇到凭据问题，请清除缓存的凭据并重新登录：

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. 联系支持：info@mapir.camera

***

### 导入错误**问题：** `ModuleNotFoundError: No module named 'chloros_sdk'`**解决方案：**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### 处理超时**问题：** 处理超时**解决方案：**

1. 延长超时时间：

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. 处理更小的批次
3. 检查可用磁盘空间
4. 监控系统资源

***

### 端口已被占用**问题：** 后端端口 5000 已被占用**解决方案：**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

或查找并关闭冲突进程：

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## 性能提示

### 优化处理速度

1. **使用并行模式**（需 Chloros+ 版本）

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **降低输出分辨率**（若可接受）

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **禁用不必要的索引**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **在 SSD 上处理**（而非 HDD）***

### 内存优化

针对大型数据集：

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### 后台处理

为其他任务释放 Python：

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## 集成示例

### Django 集成

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## 常见问题

### 问：SDK 需要互联网连接吗？

**答：**仅在初始许可证激活时需要。通过 Chloros、Chloros（浏览器）或 Chloros CLI 登录后，许可证将缓存至本地，并在离线状态下有效期为 30 天。***

### 问：我可以在没有图形界面的服务器上使用 SDK 吗？**答：** 可以！SDK 可在 Windows 和 Linux 服务器上以无头模式运行。**Linux（推荐用于无头模式）：**
* 通过 `.deb` 包安装
* 激活许可证：`chloros-cli login user@example.com 'password'`

**Windows 服务器：**
* Windows Server 2016 或更高版本
* 已安装 Chloros（仅需安装一次）
* 通过 CLI 或在任意机器上激活许可证

***

### 问：桌面版、CLI 和 SDK 之间有什么区别？

| 功能         | 桌面图形界面 | CLI 命令行 | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **界面**   | 点选式 | 命令行        | Python API  |
| **最适合**    | 可视化工作 | 脚本编写        | 集成        |
| **自动化**  | 有限     | 良好             | 优秀     |
| **灵活性** | 基本       | 良好             | 最大     |
| **许可证**     | Chloros+    | Chloros+         | Chloros+    |***

### 问：我可以使用 SDK 构建的应用程序进行分发吗？**答：** SDK 代码可以集成到您的应用程序中，但：

* 最终用户需要安装 Chloros
* 最终用户需要有效的 Chloros+ 许可证
* 商业分发需要 OEM 许可

有关 OEM 事宜，请联系 info@mapir.camera。

***

### 问：如何更新 SDK？

```bash
pip install --upgrade chloros-sdk
```

***

### 问：处理后的图像保存在哪里？

默认保存在项目路径中：

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### 问：能否处理由按计划运行的 Python 脚本生成的图像？**答：** 可以！请使用操作系统计划程序配合 Python 脚本：

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows：** 通过任务计划程序设置为每日运行。**Linux：** 通过 cron 进行计划：

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### 问：SDK 是否支持 async/await？**答：**当前版本为同步模式。如需异步行为，请使用 `wait=False` 或在单独线程中运行：

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### 问：如何在不同的 Chloros+ 账户之间切换？**答：** 使用 `logout()` 方法清除缓存的凭据，然后使用新账户重新登录：

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

注销后，请通过 GUI、浏览器或 CLI 使用新账户进行身份验证，然后再使用 SDK。

***

## 获取帮助

### 文档

* **API 参考**：本页面

### 支持渠道

* **电子邮件**：info@mapir.camera
* **网站**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **定价**：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### 示例代码

此处列出的所有示例均经过测试且可用于生产环境。请复制并根据您的具体用例进行调整。

***

## 许可**专有软件** - 版权所有 (c) 2025 MAPIR Inc.

SDK 需要有效的 Chloros+ 订阅。禁止未经授权的使用、分发或修改。
