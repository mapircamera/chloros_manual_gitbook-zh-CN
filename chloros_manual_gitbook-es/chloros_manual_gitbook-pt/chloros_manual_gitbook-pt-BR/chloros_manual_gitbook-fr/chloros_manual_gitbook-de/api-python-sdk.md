# API : Python SDK

The **Chloros Python SDK** provides programmatic access to the Chloros image processing engine, enabling automation, custom workflows, and seamless integration with your Python applications and research pipelines.

### Key Features

* 🐍 **Native Python** - Clean, Pythonic API for image processing
* 🔧 **Full API Access** - Complete control over Chloros processing
* 🚀 **Automation** - Build custom batch processing workflows
* 🔗 **Integration** - Embed Chloros in existing Python applications
* 📊 **Research-Ready** - Perfect for scientific analysis pipelines
* ⚡ **Parallel Processing** - Scales to your CPU cores (Chloros+)

### Requirements

| Requirement          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Must be installed locally                                           |
| **License**          | Chloros+ ([paid plan required](https://cloud.mapir.camera/pricing)) |
| **Operating System** | Windows 10/11 (64-bit)                                              |
| **Python**           | Python 3.7 or higher                                                |
| **Memory**           | 8GB RAM minimum (16GB recommended)                                  |
| **Internet**         | Required for license activation                                     |

{% hint style="warning" %}
**License Requirement**: The Python SDK requires a paid Chloros+ subscription for API access. Standard (free) plans do not have API/SDK access. Visit [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) to upgrade.
{% endhint %}

## Quick Start

### Installation

Install via pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**First-Time Setup**: Before using the SDK, activate your Chloros+ license by opening Chloros, Chloros (Browser) or Chloros CLI and logging in with your credentials. This only needs to be done once.
{% endhint %}

### Basic Usage

Process a folder with just a few lines:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Full Control

For advanced workflows:

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

## Installation Guide

### Prerequisites

Before installing the SDK, ensure you have:

1. **Chloros Desktop** installed ([download](download.md))
2. **Python 3.7+** installed ([python.org](https://www.python.org))
3. **Active Chloros+ license** ([upgrade](https://cloud.mapir.camera/pricing))

### Install via pip

**Standard installation:**

```bash
pip install chloros-sdk
```

**With progress monitoring support:**

```bash
pip install chloros-sdk[progress]
```

**Development installation:**

```bash
pip install chloros-sdk[dev]
```

### Verify Installation

Test that the SDK is installed correctly:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## First-Time Setup

### License Activation

The SDK uses the same license as Chloros, Chloros (Browser), and Chloros CLI. Activate once via the GUI or CLI:

1. Open **Chloros or Chloros (Browser)** and login on the User <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> tab. Or, open the **CLI**.
2. Enter your Chloros+ credentials and log in
3. License is cached locally (persists across reboots)

{% hint style="success" %}
**One-Time Setup**: After logging in via the GUI or CLI, the SDK automatically uses the cached license. No additional authentication needed!
{% endhint %}

### Test Connection

Verify the SDK can connect to Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API Reference

### ChlorosLocal Class

Main class for local Chloros image processing.

#### Constructor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parameters:**

| Parameter                 | Type | Default                   | Description                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL of local Chloros backend          |
| `auto_start_backend`      | bool | `True`                    | Automatically start backend if needed |
| `backend_exe`             | str  | `None` (auto-detect)      | Path to backend executable            |
| `timeout`                 | int  | `30`                      | Request timeout in seconds            |
| `backend_startup_timeout` | int  | `60`                      | Timeout for backend startup (seconds) |

**Examples:**

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

### Methods

#### `create_project(project_name, camera=None)`

Create a new Chloros project.

**Parameters:**

| Parameter      | Type | Required | Description                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Yes      | Name for the project                                     |
| `camera`       | str  | No       | Camera template (e.g., "Survey3N\_RGN", "Survey3W\_OCN") |

**Returns:** `dict` - Project creation response

**Example:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Import images from a folder.

**Parameters:**

| Parameter     | Type     | Required | Description                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Yes      | Path to folder with images         |
| `recursive`   | bool     | No       | Search subfolders (default: False) |

**Returns:** `dict` - Import results with file count

**Example:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configure processing settings.

**Parameters:**

| Parameter                 | Type | Default                 | Description                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | "High Quality (Faster)" | Debayer method                  |
| `vignette_correction`     | bool | `True`                  | Enable vignette correction      |
| `reflectance_calibration` | bool | `True`                  | Enable reflectance calibration  |
| `indices`                 | list | `None`                  | Vegetation indices to calculate |
| `export_format`           | str  | "TIFF (16-bit)"         | Output format                   |
| `ppk`                     | bool | `False`                 | Enable PPK corrections          |
| `custom_settings`         | dict | `None`                  | Advanced custom settings        |

**Export Formats:**

* `"TIFF (16-bit)"` - Recommended for GIS/photogrammetry
* `"TIFF (32-bit, Percent)"` - Scientific analysis
* `"PNG (8-bit)"` - Visual inspection
* `"JPG (8-bit)"` - Compressed output

**Available Indices:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2, and more.

**Example:**

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

Process the project images.

**Parameters:**

| Parameter           | Type     | Default      | Description                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Processing mode: "parallel" or "serial"   |
| `wait`              | bool     | `True`       | Wait for completion                       |
| `progress_callback` | callable | `None`       | Progress callback function(progress, msg) |
| `poll_interval`     | float    | `2.0`        | Polling interval for progress (seconds)   |

**Returns:** `dict` - Processing results

{% hint style="warning" %}
**Parallel Mode**: Requires Chloros+ license. Automatically scales to your CPU cores (up to 16 workers).
{% endhint %}

**Example:**

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

Get current project configuration.

**Returns:** `dict` - Current project configuration

**Example:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Get backend status information.

**Returns:** `dict` - Backend status

**Example:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Shutdown the backend (if started by SDK).

**Example:**

```python
chloros.shutdown_backend()
```

***

### Convenience Functions

#### `process_folder(folder_path, **options)`

One-line convenience function to process a folder.

**Parameters:**

| Parameter                 | Type     | Default         | Description                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Required        | Path to folder with images     |
| `project_name`            | str      | Auto-generated  | Project name                   |
| `camera`                  | str      | `None`          | Camera template                |
| `indices`                 | list     | `["NDVI"]`      | Indices to calculate           |
| `vignette_correction`     | bool     | `True`          | Enable vignette correction     |
| `reflectance_calibration` | bool     | `True`          | Enable reflectance calibration |
| `export_format`           | str      | "TIFF (16-bit)" | Output format                  |
| `mode`                    | str      | `"parallel"`    | Processing mode                |
| `progress_callback`       | callable | `None`          | Progress callback              |

**Returns:** `dict` - Processing results

**Example:**

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

## Context Manager Support

The SDK supports context managers for automatic cleanup:

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

## Complete Examples

### Example 1: Basic Processing

Process a folder with default settings:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Example 2: Custom Workflow

Full control over processing pipeline:

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

### Example 3: Batch Processing Multiple Folders

Process multiple flight datasets:

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

### Example 4: Research Pipeline Integration

Integrate Chloros with data analysis:

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

### Example 5: Custom Progress Monitoring

Advanced progress tracking with logging:

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

### Example 6: Error Handling

Robust error handling for production use:

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

### Example 7: Command-Line Tool

Build a custom CLI tool with the SDK:

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

**Usage:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Exception Handling

The SDK provides specific exception classes for different error types:

### Exception Hierarchy

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Exception Examples

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

## Advanced Topics

### Custom Backend Configuration

Use a custom backend location or configuration:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Non-Blocking Processing

Start processing and continue with other tasks:

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

### Memory Management

For large datasets, process in batches:

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

## Troubleshooting

### Backend Not Starting

**Issue:** SDK fails to start backend

**Solutions:**

1. Verify Chloros Desktop is installed:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Check Windows Firewall isn't blocking
3. Try manual backend path:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### License Not Detected

**Issue:** SDK warns about missing license

**Solutions:**

1. Open Chloros, Chloros (Browser) or Chloros CLI and login.
2. Verify license is cached:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Contact support: info@mapir.camera

***

### Import Errors

**Issue:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Solutions:**

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

### Processing Timeout

**Issue:** Processing times out

**Solutions:**

1. Increase timeout:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Process smaller batches
3. Check available disk space
4. Monitor system resources

***

### Port Already in Use

**Issue:** Backend port 5000 occupied

**Solutions:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Or find and close conflicting process:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Performance Tips

### Optimize Processing Speed

1. **Use Parallel Mode** (requires Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Reduce Output Resolution** (if acceptable)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Disable Unnecessary Indices**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Process on SSD** (not HDD)

***

### Memory Optimization

For large datasets:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Background Processing

Free up Python for other tasks:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integration Examples

### Django Integration

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

## FAQ

### Q: Does the SDK require an internet connection?

**A:** Only for initial license activation. After logging in via Chloros, Chloros (Browser) or Chloros CLI the license is cached locally and works offline for 30 days.

***

### Q: Can I use the SDK on a server without GUI?

**A:** Yes! Requirements:

* Windows Server 2016 or later
* Chloros installed (one-time)
* License activated on any machine (cached license copied to server)

***

### Q: What's the difference between Desktop, CLI, and SDK?

| Feature         | Desktop GUI | CLI Command Line | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interface**   | Point-click | Command          | Python API  |
| **Best For**    | Visual work | Scripting        | Integration |
| **Automation**  | Limited     | Good             | Excellent   |
| **Flexibility** | Basic       | Good             | Maximum     |
| **License**     | Chloros+    | Chloros+         | Chloros+    |

***

### Q: Can I distribute apps built with the SDK?

**A:** SDK code can be integrated into your applications, but:

* End users need Chloros installed
* End users need active Chloros+ licenses
* Commercial distribution requires OEM licensing

Contact info@mapir.camera for OEM inquiries.

***

### Q: How do I update the SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Q: Where are processed images saved?

By default, in the Project Path :

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Q: Can I process images from Python scripts running on schedule?

**A:** Yes! Use Windows Task Scheduler with Python scripts:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Schedule via Task Scheduler to run daily.

***

### Q: Does the SDK support async/await?

**A:** Current version is synchronous. For async behavior, use `wait=False` or run in separate thread:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Getting Help

### Documentation

* **API Reference**: This page

### Support Channels

* **Email**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Pricing**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Sample Code

All examples listed here are tested and production-ready. Copy and adapt them for your use case.

***

## License

**Proprietary Software** - Copyright (c) 2025 MAPIR Inc.

SDK requires an active Chloros+ subscription. Unauthorized use, distribution, or modification is prohibited.
