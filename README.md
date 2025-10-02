# libpointing Raw HID Mouse Demo

A Python demonstration application that uses the [libpointing](http://libpointing.org/) library to capture raw HID (Human Interface Device) events directly from pointing devices, bypassing the operating system's transfer functions.

## Quick Start

```bash
# Clone and setup
git clone https://github.com/yourusername/libpointing_example.git
cd libpointing_example
git clone https://github.com/INRIA/libpointing.git

# Install dependencies
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Fix libpointing bindings
cp __init__.py libpointing/bindings/Python/cython/libpointing/__init__.py
cp build_homebrew.py libpointing/bindings/Python/cython/build_homebrew.py

# Build Python bindings
cd libpointing/bindings/Python/cython
python build_homebrew.py build_ext --inplace
cd ../../../..

# Run the demo
python libpointing_demo.py
```

## Overview

This demo showcases how to:
- Access raw HID events directly from pointing devices
- Retrieve actual device parameters (resolution, update frequency)
- Display raw counts from the device (not pixels)
- Bypass OS pointer acceleration and transfer functions
- Work with libpointing's Python bindings via Cython

## Features

- **Raw HID Access**: Captures mouse events directly from hardware
- **Real Device Metrics**: Shows actual resolution (counts/inch) and update frequency (Hz)
- **Visual Feedback**: Interactive pygame window with crosshair pointer
- **Device Information**: Displays vendor, product, VID:PID, and URI information
- **Configurable**: JSON configuration for device selection and display settings

## Prerequisites

### macOS

1. **libpointing** (via Homebrew):
   ```bash
   brew install libpointing
   ```

2. **Python 3.8+** with pip

3. **Xcode Command Line Tools** (for building Python bindings):
   ```bash
   xcode-select --install
   ```

## Installation

### Step 1: Clone the Repository

```bash
git clone https://github.com/yourusername/libpointing_example.git
cd libpointing_example
```

### Step 2: Clone the libpointing Library

The libpointing library needs to be cloned as a submodule or directly:

```bash
# Clone the official libpointing library
git clone https://github.com/INRIA/libpointing.git
```

### Step 3: Set Up Python Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### Step 4: Install Python Dependencies

```bash
pip install -r requirements.txt
```

### Step 5: Fix the libpointing Python Bindings for macOS

The original libpointing bindings need two modifications to work properly on macOS:

#### 5a. Fix the `__init__.py` file

The original `__init__.py` tries to import Windows-specific modules on all platforms. Replace it with the fixed version:

```bash
cp __init__.py libpointing/bindings/Python/cython/libpointing/__init__.py
```

Or manually edit `libpointing/bindings/Python/cython/libpointing/__init__.py` to wrap the Windows import in a platform check:

```python
import platform

from .libpointing import (
    PointingDevice,
    DisplayDevice,
    TransferFunction,
    PointingDeviceManager,
    PointingDeviceDescriptor,
    )

# Windows-specific acceleration function
if platform.system() == 'Windows':
    from .libpointing import winSystemPointerAcceleration
```

#### 5b. Add the Homebrew build script

Copy the `build_homebrew.py` script into the bindings directory:

```bash
cp build_homebrew.py libpointing/bindings/Python/cython/build_homebrew.py
```

This script is configured to use the Homebrew-installed libpointing library.

### Step 6: Build the libpointing Python Bindings

```bash
cd libpointing/bindings/Python/cython
python build_homebrew.py build_ext --inplace
cd ../../../..
```

**Note**: If your terminal has a Python alias override, use the explicit path:

```bash
cd libpointing/bindings/Python/cython
/Users/yourusername/projects/libpointing_example/venv/bin/python build_homebrew.py build_ext --inplace
cd ../../../..
```

### Step 7: Verify the Installation

```bash
python -c "import sys; sys.path.insert(0, 'libpointing/bindings/Python/cython'); from libpointing.libpointing import PointingDevice; print('✓ Bindings loaded successfully!')"
```

Or with explicit Python path:

```bash
./venv/bin/python -c "import sys; sys.path.insert(0, 'libpointing/bindings/Python/cython'); from libpointing.libpointing import PointingDevice; print('✓ Bindings loaded successfully!')"
```

## Configuration

Edit `config.json` to configure your device and display settings:

```json
{
  "device": {
    "uri": "any:?vendor=0x46d&product=0xb034&cpi=2000&hz=125&debugLevel=2",
    "description": "MX Master 3S Bluetooth"
  },
  "display": {
    "width": 1200,
    "height": 800,
    "target_size": 25,
    "background_color": [20, 20, 25],
    "target_color": [100, 200, 255],
    "text_color": [220, 220, 220]
  }
}
```

### Device URI Options

The device URI supports several formats:

- `any:` - Use any available pointing device
- `usb:` - Use any USB pointing device
- `usb:046d:c52b` - Specific device by VID:PID
- `any:?vendor=0x46d&product=0xb034&cpi=2000&hz=125` - Device with parameters

Parameters:
- `vendor` - Vendor ID (hex)
- `product` - Product ID (hex)
- `cpi` - Counts per inch (resolution)
- `hz` - Update frequency
- `debugLevel` - Debug verbosity (0-3)

## Usage

1. **Activate the virtual environment** (if not already active):
   ```bash
   source venv/bin/activate
   ```

2. **Run the demo**:
   ```bash
   python libpointing_demo.py
   ```
   
   If your terminal has a Python alias override, use the explicit path:
   ```bash
   ./venv/bin/python libpointing_demo.py
   ```

3. **Controls**:
   - Move your mouse to see raw HID data
   - `R` - Reset pointer position to center
   - `ESC` or `Q` - Quit application

## What You'll See

The demo window displays:

### Device Information
- Vendor and Product names
- Device URI
- Vendor ID and Product ID (VID:PID)
- Resolution (counts per inch)
- Update Frequency (Hz)

### Raw HID Data
- Current position in screen coordinates
- Last delta (raw counts from device)
- Total counts accumulated (X and Y)
- Number of events received
- Last event timestamp (microseconds)

## Project Structure

```
libpointing_example/
├── README.md                  # This file
├── requirements.txt           # Python dependencies
├── config.json               # Configuration file
├── .gitignore                # Git ignore rules
├── libpointing_demo.py       # Main demo application
├── __init__.py               # Fixed __init__.py for libpointing (to copy)
├── build_homebrew.py         # Build script for Homebrew (to copy)
├── libpointing/              # libpointing library (cloned from GitHub)
│   └── bindings/
│       └── Python/
│           └── cython/       # Python bindings
│               ├── build_homebrew.py  # (copied here during setup)
│               └── libpointing/       # Built Python module
│                   └── __init__.py    # (fixed during setup)
└── venv/                     # Python virtual environment
```

### Helper Files

The repository includes two helper files in the root directory that need to be copied into the libpointing library during setup:

- **`__init__.py`**: Fixed version that properly handles platform-specific imports (prevents Windows-only imports on macOS/Linux)
- **`build_homebrew.py`**: Build script configured for Homebrew-installed libpointing on macOS

## Troubleshooting

### Setup Issues

**Error: Platform-specific import errors on non-Windows systems**
- Make sure you copied the fixed `__init__.py` file:
  ```bash
  cp __init__.py libpointing/bindings/Python/cython/libpointing/__init__.py
  ```

**Can't find build_homebrew.py in libpointing/bindings/Python/cython/**
- Copy the build script from the root directory:
  ```bash
  cp build_homebrew.py libpointing/bindings/Python/cython/build_homebrew.py
  ```

### Build Issues

**Error: `library 'pointing' not found`**
- Ensure libpointing is installed: `brew list libpointing`
- Verify Homebrew path: `brew --prefix` (should be `/opt/homebrew` on Apple Silicon)

**Error: `ModuleNotFoundError: No module named 'Cython'`**
- Install Cython: `pip install Cython`

**Python version mismatch or alias issues**
- Use the explicit venv Python path:
  ```bash
  ./venv/bin/python build_homebrew.py build_ext --inplace
  ```

### Runtime Issues

**Error: Failed to initialize libpointing device**
- Check device permissions (you may need to grant input monitoring access in System Preferences)
- Try different URI formats: `any:`, `usb:`, or specific VID:PID
- Verify your device is connected and recognized by the system

**Import Error: No module named 'libpointing'**
- Ensure you built the Python bindings (see Installation step 4)
- Check that you're running from the correct directory
- Verify the virtual environment is activated

### Device Detection

To find your device's VID:PID:
```bash
system_profiler SPUSBDataType
```

Or check the demo output - it lists all detected HID devices on startup.

## Understanding Raw HID Data

### Counts vs Pixels
- **Raw Counts**: Hardware sensor readings from the mouse
- **Pixels**: Screen coordinates after OS transfer function
- This demo shows **raw counts** directly from the device

### Resolution (CPI/DPI)
- Counts Per Inch (CPI) or Dots Per Inch (DPI)
- Higher resolution = more counts per physical inch of movement
- Example: 2000 CPI means 2000 counts per inch

### Update Frequency
- How often the device reports position updates
- Measured in Hertz (Hz)
- Common values: 125 Hz, 500 Hz, 1000 Hz

## Technical Details

### libpointing Library
- Provides cross-platform access to pointing devices
- Supports Windows, macOS, and Linux
- Offers raw HID access and transfer function research
- More info: http://libpointing.org/

### Python Bindings
- Built with Cython for performance
- Wraps C++ libpointing API
- Provides Pythonic interface to device events

### Architecture
- **Callback-based**: Device events trigger Python callbacks
- **Thread-safe**: Uses locks for data synchronization
- **Non-blocking**: Uses pygame event loop with libpointing idle()

## Credits

- **libpointing**: Géry Casiez, Nicolas Roussel, and contributors (INRIA)
- **Demo Application**: Built as an example of raw HID access

## License

This demo application is provided as an example. Please refer to the libpointing library for its license terms.

## Further Reading

- [libpointing Official Website](http://libpointing.org/)
- [libpointing GitHub Repository](https://github.com/INRIA/libpointing)
- [Transfer Functions Research](http://ns.inria.fr/mjolnir/TransferFunctions/)

---

**Note**: This demo requires appropriate permissions to access HID devices. On macOS, you may need to grant "Input Monitoring" permissions in System Preferences → Security & Privacy → Privacy.
