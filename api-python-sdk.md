# API : Python SDK

**Chloros Python SDK** 提供对Chloros图像处理引擎的程序化访问，支持自动化操作、自定义工作流，并与您的Python应用程序及研究流程无缝集成。

### 核心特性

* 🐍 **原生Python** - 简洁的Python式图像处理
* 🔧 **完整访问** - 全面掌控图像处理流程
* 🚀 **自动化** - 构建定制化批处理工作流
* 🔗 **集成** - 将Chloros嵌入现有Python应用程序
* 📊 **科研就绪** - 完美适配科学分析管道
* ⚡ **并行处理** - 扩展至CPU核心（Chloros+）

### 系统要求

| 要求          | 详细说明                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros桌面版**  | 必须本地安装                                           |
| **许可证**          | Chloros+ ([需付费方案](https://cloud.mapir.camera/pricing)) |
| **操作系统** | Windows 10/11 (64位)                                              |
| **Python**           | Python 3.7或更高版本                                                |
| **内存**           | 最低8GB RAM（推荐16GB）                                  |
| **网络连接**         | 需联网激活许可证                                     |

{% 提示 style=&quot;warning&quot; %}
**许可证要求**：Python SDK 需付费订阅 Chloros+ 才能访问 API。 标准（免费）方案不包含API/SDK访问权限。请访问[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)进行升级。
{% endhint %}

## 快速入门

### 安装

通过pip安装：

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**首次设置**：使用SDK前，请通过打开Chloros激活Chloros+许可证， Chloros（浏览器）或 Chloros CLI 并使用您的凭据登录。此操作仅需执行一次。
{% endhint %}

### 基础用法

处理仅含少量内容的文件夹：

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### 全面控制

用于高级工作流：

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

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

安装 SDK 前请确保：

1. 已安装 **Chloros Desktop** ([下载](download.md))
2. 已安装 **Python 3.7+** ([python.org](https://www.python.org))
3. **有效的 Chloros+ 许可证** ([升级](https://cloud.mapir.camera/pricing))

### 通过 pip 安装

**标准安装：**

```bash
pip install chloros-sdk
```

**带进度监控支持：**

```bash
pip install chloros-sdk[progress]
```

**开发环境安装：**

```bash
pip install chloros-sdk[dev]
```

### 安装验证

测试SDK是否正确安装：

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## 初始配置

### 许可证激活

SDK与Chloros、Chloros（浏览器版）及Chloros CLI共享同一许可证。 通过图形界面或CLI激活一次：

1. 打开**Chloros或Chloros（浏览器版）**，在用户 <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> 标签页登录。或直接打开**CLI**。
2. 输入Chloros+凭证并登录
3. 许可证本地缓存（重启后仍有效）

{%提示 style=&quot;success&quot; %}
**一次性设置**：通过GUI或CLI登录后，SDK将自动使用缓存许可证。无需额外认证！
{% endhint %}

### 测试连接

验证SDK能否连接至Chloros：

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

本地图像处理的核心类。

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
| `api_url`                 | 字符串 | `"http://localhost:5000"` | 本地Chloros后端的URL          |
| `auto_start_backend`      | 布尔值 | `True`                    | 需要时自动启动后端 |
| `backend_exe`             | str  | `None` (自动检测)      | 后端可执行文件路径            |
| `timeout`                 | int  | `30`                      | 请求超时时间（秒）            |
| `backend_startup_timeout` | int  | `60`                      | 后端启动超时时间（秒） |

**示例：**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### 方法

#### `create_project(project_name, camera=None)`

创建新的Chloros项目。

**参数：**

| 参数         | 类型 | 是否必填 | 描述                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | 是      | 项目名称                                     |
| `camera`       | 字符串 | 否       | 相机模板（例如&quot;Survey3N\_RGN&quot;、&quot;Survey3W\_OCN&quot;） |

**返回值：** `dict` - 项目创建响应

**示例：**

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

| 参数         | 类型       | 必填       | 描述                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | 字符串/路径 | 是      | 图片所在文件夹路径         |
| `recursive`   | 布尔值 | 否       | 搜索子文件夹（默认：False） |

**返回值：** `dict` - 包含文件数量的导入结果

**示例：**

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
| `debayer`                 | str  | &quot;高质量（更快）&quot; | 去马赛克方法                  |
| `vignette_correction`     | bool | `True`                  | 启用暗角校正      |
| `reflectance_calibration` | 布尔 | `True`                  | 启用反射率校准      |
| `indices`                 | 列表 | `None`                  | 待计算植被指数 |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | 输出格式                   |
| `ppk`                     | bool | `False`                 | 启用PPK校正                  |
| `custom_settings`         | 字典 | `None`                  | 高级自定义设置        |

**导出格式：**

* `"TIFF (16-bit)"` - 推荐用于GIS/摄影测量
* `"TIFF (32-bit, Percent)"` - 科学分析
* `"PNG (8-bit)"` - 目视检查
* `"JPG (8-bit)"` - 压缩输出

**可用指数：**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 等。

**示例：**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
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
| `mode`              | str      | `"parallel"` | 处理模式：&quot;parallel&quot; 或 &quot;serial&quot;   |
| `wait`              | 布尔值     | `True`       | 等待处理完成                       |
| `progress_callback` | 可调用对象 | `None`       | 进度回调函数(progress, msg) |
| `poll_interval`     | 浮点型    | `2.0`        | 进度轮询间隔（秒）   |

**返回值：** `dict` - 处理结果

{% 提示 style=&quot;warning&quot; %}
**并行模式**：需Chloros+许可证。自动扩展至CPU核心数（最多16个工作进程）。
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

**返回值：** `dict` - 当前项目配置

**示例：**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

获取后端状态信息。

**返回值：** `dict` - 后端状态

**示例：**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

关闭后端（若由SDK启动）。

**示例：**

```python
chloros.shutdown_backend()
```

***

### 便捷函数

#### `process_folder(folder_path, **options)`

处理文件夹的单行便捷函数。

**参数：**

| 参数                 | 类型     | 默认值         | 描述                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | 字符串/路径 | 必填        | 含图像的文件夹路径     |
| `project_name`            | 字符串      | 自动生成  | 项目名称                   |
| `camera`                  | 字符串 | `None`          | 相机模板                |
| `indices`                 | 列表     | `["NDVI"]`      | 待计算索引           |
| `vignette_correction`     | 布尔值     | `True`          | 启用晕影校正                 |
| `reflectance_calibration` | 布尔值     | `True`          | 启用反射率校准                |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | 输出格式                  |
| `mode`                    | str      | `"parallel"`    | 处理模式                |
| `progress_callback`       | 可调用对象 | `None`          | 进度回调函数              |

**返回值：** `dict` - 处理结果

**示例：**

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

SDK支持上下文管理器实现自动清理：

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

### 示例1：基础处理

使用默认设置处理文件夹：

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### 示例 2：自定义工作流

对处理管道实现全局控制：

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
    debayer="High Quality (Faster)",
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

### 示例 4：研究管道集成

将 Chloros 集成至数据分析：

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

带日志记录的高级进度跟踪：

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

面向生产环境的健壮错误处理：

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
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
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

### 示例 7：命令行工具

基于 SDK 构建自定义 CLI 工具：

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
    
    args = parser.parse_args()
    
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
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## 异常处理

SDK为不同错误类型提供特定异常类：

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
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

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

处理大型数据集时采用批处理模式：

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

### 后端启动失败

**问题：** SDK 无法启动后端

**解决方案：**

1. 确认已安装Chloros桌面版：

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. 检查Windows防火墙是否阻塞
3. 尝试手动指定后端路径：

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### 未检测到许可证

**问题：**SDK 提示许可证缺失

**解决方案：**

1. 打开Chloros、Chloros（浏览器）或Chloros CLI并登录
2. 验证许可证是否已缓存：

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. 联系支持：info@mapir.camera

***

### 导入错误

**问题：** `ModuleNotFoundError: No module named 'chloros_sdk'`

**解决方案：**

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

### 处理超时

**问题：**处理超时

**解决方案：**

1. 延长超时时间：

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. 分批处理较小数据量
3. 检查可用磁盘空间
4. 监控系统资源

***

### 端口已被占用

**问题：**后端端口5000被占用

**解决方案：**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

或查找并关闭冲突进程：

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## 性能优化建议

### 提升处理速度

1. **启用并行模式**（需Chloros+）

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **降低输出分辨率**（若可接受）

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **禁用非必要索引**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **在SSD上处理**（非HDD）

***

### 内存优化

针对大型数据集：

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### 后台处理

释放Python资源供其他任务使用：

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## 集成示例

### Django集成

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

### Flask

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

### 问：SDK是否需要联网？

**答：**仅需初始许可证激活时连接。通过Chloros、Chloros（浏览器）或Chloros CLI登录后，许可证将缓存至本地，支持离线使用30天。

***

### 问：能否在无图形界面的服务器上使用SDK？

**答：**可以！要求如下：

* Windows Server 2016或更高版本
* 已安装Chloros（仅需安装一次）
* 在任意设备上激活许可证 （缓存许可证已复制至服务器）

***

### 问：桌面版、CLI和SDK有何区别？

| 功能         | 桌面图形界面 | CLI 命令行 | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **界面类型**   | 点击式操作 | 命令行模式    | Python API  |
| **适用场景**    | 可视化工作 | 脚本编写        | 系统集成     |
| **自动化程度**  | 有限         | 良好             | 卓越       |
| **灵活性** | 基础       | 良好             | 最高     |
| **许可证**     | Chloros+    | Chloros+         | Chloros+    |

***

### 问：能否分发使用SDK构建的应用程序？

**答：**SDK代码可集成至您的应用程序，但需满足：

* 终端用户需安装Chloros
* 终端用户需持有有效的Chloros+许可证
* 商业分发需获取OEM授权

有关OEM授权事宜，请联系info@mapir.camera。

***

### 问：如何更新SDK？

```bash
pip install --upgrade chloros-sdk
```

***

### 问：处理后的图像保存在何处？

默认保存在项目路径：

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### 问：能否通过定时运行的Python脚本处理图像？

**答：**可以！请结合Python脚本使用Windows任务计划程序：

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

通过任务计划程序设置每日运行。

***

### 问：SDK是否支持异步/等待？

**答：**当前版本为同步模式。如需异步行为，请使用`wait=False`或在独立线程中运行：

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## 获取帮助

### 文档

* **API 参考文档**：本页面

### 支持渠道

* **电子邮件**：info@mapir.camera
* **官网**：[https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **定价**：[https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### 示例代码

此处列出的所有示例均经过测试且可直接投入生产环境。请根据您的使用场景复制并调整这些代码。

***

## 许可协议

**专有软件** - 版权所有 (c) 2025 MAPIR 有限公司

SDK 需有效 Chloros+ 订阅支持。禁止未经授权的使用、分发或修改。
