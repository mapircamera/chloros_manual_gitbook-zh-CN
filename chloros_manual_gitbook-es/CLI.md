# CLI : Command Line

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

The **Chloros CLI** provides powerful command-line access to the Chloros image processing engine, enabling automation, scripting, and headless operation for your imaging workflows.

### Key Features

* 🚀 **Automation** - Script batch processing of multiple datasets
* 🔗 **Integration** - Embed in existing workflows and pipelines
* 💻 **Headless Operation** - Run without GUI
* 🌍 **Multi-Language** - Support for 38 languages
* ⚡ **Parallel Processing** - Dynamically scales to your CPU (up to 16 parallel workers)

### Requirements

| Requirement          | Details                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operating System** | Windows 10/11 (64-bit)                                              |
| **License**          | Chloros+ ([paid plan required](https://cloud.mapir.camera/pricing)) |
| **Memory**           | 8GB RAM minimum (16GB recommended)                                  |
| **Internet**         | Required for license activation                                     |
| **Disk Space**       | Varies by project size                                              |

{% hint style="warning" %}
**License Requirement**: The CLI requires a paid Chloros+ subscription. Standard (free) plans do not have CLI access. Visit [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) to upgrade.
{% endhint %}

## Quick Start

### Installation

The CLI is automatically included with the Chloros installer:

1. Download and run **Chloros Installer.exe**
2. Complete the installation wizard
3. CLI installed to: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
The installer automatically adds `chloros-cli` to your system PATH. Restart your terminal after installation.
{% endhint %}

### First-Time Setup

Before using the CLI, activate your Chloros+ license:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Basic Usage

Process a folder with default settings:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Command Reference

### General Syntax

```
chloros-cli [global-options] <command> [command-options]
```

***

## Commands

### `process` - Process Images

Process images in a folder with calibration.

**Syntax:**

```bash
chloros-cli process <input-folder> [options]
```

**Example:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Process Command Options

| Option                | Type    | Default        | Description                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Path    | _Required_     | Folder containing RAW/JPG multispectral images                                         |
| `-o, --output`        | Path    | Same as input  | Output folder for processed images                                                     |
| `-n, --project-name`  | String  | Auto-generated | Custom project name                                                                    |
| `--vignette`          | Flag    | Enabled        | Enable vignette correction                                                             |
| `--no-vignette`       | Flag    | -              | Disable vignette correction                                                            |
| `--reflectance`       | Flag    | Enabled        | Enable reflectance calibration                                                         |
| `--no-reflectance`    | Flag    | -              | Disable reflectance calibration                                                        |
| `--ppk`               | Flag    | Disabled       | Apply PPK corrections from .daq light sensor data                                      |
| `--format`            | Choice  | TIFF (16-bit)  | Output format: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Integer | Auto           | Minimum target size in pixels for calibration panel detection                          |
| `--target-clustering` | Integer | Auto           | Target clustering threshold (0-100)                                                    |
| `--exposure-pin-1`    | String  | None           | Lock exposure for camera model (Pin 1)                                                 |
| `--exposure-pin-2`    | String  | None           | Lock exposure for camera model (Pin 2)                                                 |
| `--recal-interval`    | Integer | Auto           | Recalibration interval in seconds                                                      |
| `--timezone-offset`   | Integer | 0              | Timezone offset in hours                                                               |

***

### `login` - Authenticate Account

Login with your Chloros+ credentials to enable CLI processing.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Example:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Special Characters**: Use single quotes around passwords containing characters like `$`, `!`, or spaces.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

***

### `logout` - Clear Credentials

Clear stored credentials and logout from your account.

**Syntax:**

```bash
chloros-cli logout
```

**Example:**

```powershell
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Check License Status

Display current license and authentication status.

**Syntax:**

```bash
chloros-cli status
```

**Example:**

```powershell
chloros-cli status
```

**Output:**

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

### `export-status` - Check Export Progress

Monitor Thread 4 export progress during or after processing.

**Syntax:**

```bash
chloros-cli export-status
```

**Example:**

```powershell
chloros-cli export-status
```

**Use Case:** Call this command while processing is running to check export progress.

***

### `language` - Manage Interface Language

View or change the CLI interface language.

**Syntax:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Examples:**

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

#### Supported Languages (38 Total)

| Code    | Language              | Native Name      |
| ------- | --------------------- | ---------------- |
| `en`    | English               | English          |
| `es`    | Spanish               | Español          |
| `pt`    | Portuguese            | Português        |
| `fr`    | French                | Français         |
| `de`    | German                | Deutsch          |
| `it`    | Italian               | Italiano         |
| `ja`    | Japanese              | 日本語              |
| `ko`    | Korean                | 한국어              |
| `zh`    | Chinese (Simplified)  | 简体中文             |
| `zh-TW` | Chinese (Traditional) | 繁體中文             |
| `ru`    | Russian               | Русский          |
| `nl`    | Dutch                 | Nederlands       |
| `ar`    | Arabic                | العربية          |
| `pl`    | Polish                | Polski           |
| `tr`    | Turkish               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesian            | Bahasa Indonesia |
| `vi`    | Vietnamese            | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
| `sv`    | Swedish               | Svenska          |
| `da`    | Danish                | Dansk            |
| `no`    | Norwegian             | Norsk            |
| `fi`    | Finnish               | Suomi            |
| `el`    | Greek                 | Ελληνικά         |
| `cs`    | Czech                 | Čeština          |
| `hu`    | Hungarian             | Magyar           |
| `ro`    | Romanian              | Română           |
| `uk`    | Ukrainian             | Українська       |
| `pt-BR` | Brazilian Portuguese  | Português Brasileiro |
| `zh-HK` | Cantonese             | 粵語             |
| `ms`    | Malay                 | Bahasa Melayu    |
| `sk`    | Slovak                | Slovenčina       |
| `bg`    | Bulgarian             | Български        |
| `hr`    | Croatian              | Hrvatski         |
| `lt`    | Lithuanian            | Lietuvių         |
| `lv`    | Latvian               | Latviešu         |
| `et`    | Estonian              | Eesti            |
| `sl`    | Slovenian             | Slovenščina      |

{% hint style="success" %}
**Automatic Persistence**: Your language preference is saved to `~/.chloros/cli_language.json` and persists across all sessions.
{% endhint %}

***

### `set-project-folder` - Set Default Project Folder

Change the default project folder location (shared with GUI).

**Syntax:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Example:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Show Project Folder

Display the current default project folder location.

**Syntax:**

```bash
chloros-cli get-project-folder
```

**Example:**

```powershell
chloros-cli get-project-folder
```

**Output:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Reset to Default

Reset the project folder to the default location.

**Syntax:**

```bash
chloros-cli reset-project-folder
```

***

## Global Options

These options apply to all commands:

| Option          | Type    | Default       | Description                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Path    | Auto-detected | Path to backend executable                       |
| `--port`        | Integer | 5000          | Backend API port number                          |
| `--restart`     | Flag    | -             | Force restart backend (kills existing processes) |
| `--version`     | Flag    | -             | Show version information and exit                |
| `--help`        | Flag    | -             | Show help information and exit                   |

**Example with Global Options:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Processing Settings Guide

### Parallel Processing

Chloros+ CLI **automatically scales** parallel processing to match your computer's capabilities:

**How It Works:**

* Detects your CPU cores and RAM
* Allocates workers: **2× CPU cores** (uses hyperthreading)
* **Maximum: 16 parallel workers** (for stability)

**System Tiers:**

| System Type   | CPU        | RAM      | Workers  | Performance     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ cores  | 32+ GB   | Up to 16 | Maximum speed   |
| **Mid-Range** | 8-15 cores | 16-31 GB | 8-16     | Excellent speed |
| **Low-End**   | 4-7 cores  | 8-15 GB  | 4-8      | Good speed      |

{% hint style="success" %}
**Automatic Optimization**: The CLI automatically detects your system specs and configures optimal parallel processing. No manual configuration needed!
{% endhint %}

### Debayer Methods

The CLI uses **High Quality (Faster)** as the default and recommended debayer algorithm:

| Method                      | Quality | Speed | Description                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **High Quality (Faster)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Edge-aware algorithm (default, recommended) |

### Vignette Correction

**What it does:** Corrects light falloff at image edges (darker corners common in camera imagery).

* **Enabled by default** - Most users should keep this enabled
* Use `--no-vignette` to disable

{% hint style="success" %}
**Recommendation**: Always enable vignette correction to ensure uniform brightness across the frame.
{% endhint %}

### Reflectance Calibration

Converts raw sensor values to standardized reflectance percentages using calibration panels.

* **Enabled by default** - Essential for vegetation analysis
* Requires calibration target panels in images
* Use `--no-reflectance` to disable

{% hint style="info" %}
**Requirements**: Ensure calibration panels are properly exposed and visible in your images for accurate reflectance conversion.
{% endhint %}

### PPK Corrections

**What it does:** Applies Post-Processed Kinematic corrections using DAQ-A-SD log data for improved GPS accuracy.

* **Disabled by default**
* Use `--ppk` to enable
* Requires .daq files in project folder from MAPIR DAQ-A-SD light sensor.

### Output Formats

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bit Depth</th><th width="116.5999755859375">File Size</th><th>Best For</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bit integer</td><td>Large</td><td>GIS analysis, photogrammetry (recommended)</td></tr><tr><td><strong>TIFF (32-bit, Percent)</strong></td><td>32-bit float</td><td>Very Large</td><td>Scientific analysis, research</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit integer</td><td>Medium</td><td>Visual inspection, web sharing</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bit integer</td><td>Small</td><td>Quick preview, compressed output</td></tr></tbody></table>

***

## Automation & Scripting

### PowerShell Batch Processing

Process multiple dataset folders automatically:

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

### Windows Batch Script

Simple loop for batch processing:

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

### Python Automation Script

Advanced automation with error handling:

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

## Processing Workflow

### Standard Workflow

1. **Input**: Folder containing RAW/JPG image pairs
2. **Discovery**: CLI auto-scans for supported image files
3. **Processing**: Parallel mode scales to your CPU cores (Chloros+)
4. **Output**: Creates camera-model subfolders with processed images

### Example Output Structure

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

### Processing Time Estimates

Typical processing times for 100 images (12MP each):

| Mode              | Time      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Parallel Mode** | 5-10 min  | i7/Ryzen 7, 16GB RAM, SSD (up to 16 workers) |
| **Parallel Mode** | 10-15 min | i5/Ryzen 5, 8GB RAM, HDD (up to 8 workers)   |

{% hint style="info" %}
**Performance Tip**: Processing time varies based on image count, resolution, and computer specs.
{% endhint %}

***

## Troubleshooting

### CLI Not Found

**Error:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Solutions:**

1. Verify installation location:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Use full path if not in PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Add to PATH manually:
   * Open System Properties → Environment Variables
   * Edit PATH variable
   * Add: `C:\Program Files\Chloros\resources\cli`
   * Restart terminal

***

### Backend Failed to Start

**Error:**

```
Backend failed to start within 30 seconds
```

**Solutions:**

1. Check if backend already running (close it first)
2. Check Windows Firewall is not blocking
3. Try different port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Force restart backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### License / Authentication Issues

**Error:**

```
Chloros+ license required for CLI access
```

**Solutions:**

1. Verify you have an active Chloros+ subscription
2. Login with your credentials:

```powershell
chloros-cli login user@example.com 'password'
```

3. Check license status:

```powershell
chloros-cli status
```

4. Contact support: info@mapir.camera

***

### No Images Found

**Error:**

```
No images found in the specified folder
```

**Solutions:**

1. Verify folder contains supported formats (.RAW, .TIF, .JPG)
2. Check folder path is correct (use quotes for paths with spaces)
3. Ensure you have read permissions for the folder
4. Check file extensions are correct

***

### Processing Stalls or Hangs

**Solutions:**

1. Check available disk space (ensure enough for output)
2. Close other applications to free memory
3. Reduce image count (process in batches)

***

### Port Already in Use

**Error:**

```
Port 5000 is already in use
```

**Solution:**

Specify a different port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## FAQ

### Q: Do I need a license for the CLI?

**A:** Yes! The CLI requires a paid **Chloros+ license**.

* ❌ Standard (free) plan: CLI disabled
* ✅ Chloros+ (paid) plans: CLI fully enabled

Subscribe at: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q: Can I use the CLI on a server without GUI?

**A:** Yes! The CLI runs completely headless. Requirements:

* Windows Server 2016 or later
* Visual C++ Redistributable installed
* Sufficient RAM (8GB minimum, 16GB recommended)
* One-time GUI license activation on any machine

***

### Q: Where are processed images saved?

**A:** By default, processed images are saved in the **same folder as input** in camera-model subfolders (e.g., `Survey3N_RGN/`).

Use `-o` option to specify different output folder:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Q: Can I process multiple folders at once?

**A:** Not directly in one command, but you can use scripting to process folders sequentially. See [Automation & Scripting](CLI.md#automation--scripting) section.

***

### Q: How do I save CLI output to a log file?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Q: What happens if I press Ctrl+C during processing?

**A:** The CLI will:

1. Stop processing gracefully
2. Shut down the backend
3. Exit with code 130

Partially processed images may remain in the output folder.

***

### Q: Can I automate CLI processing?

**A:** Absolutely! The CLI is designed for automation. See [Automation & Scripting](CLI.md#automation--scripting) for PowerShell, Batch, and Python examples.

***

### Q: How do I check the CLI version?

**A:**

```powershell
chloros-cli --version
```

**Output:**

```
Chloros CLI 1.0.2
```

***

## Getting Help

### Command-Line Help

View help information directly in the CLI:

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Support Channels

* **Email**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Pricing**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Complete Examples

### Example 1: Basic Processing

Process with default settings (vignette, reflectance):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Example 2: High-Quality Scientific Output

32-bit float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Example 3: Fast Preview Processing

8-bit PNG without calibration for quick review:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Example 4: PPK-Corrected Processing

Apply PPK corrections with reflectance:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Example 5: Custom Output Location

Process to different drive with specific format:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Example 6: Authentication Workflow

Complete authentication flow:

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

### Example 7: Multi-Language Usage

Change interface language:

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
