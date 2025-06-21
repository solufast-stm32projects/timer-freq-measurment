# STM32U5xx Timer Frequency Measurement Project

A custom STM32U5xx project with a modern build system using Python-based Makefile generation and YAML configuration. This project demonstrates how to set up a robust development environment for STM32U5xx microcontrollers with proper debugging support.

## 🎯 Project Overview

This project provides:
- **Custom build system** with Python-based Makefile generation
- **YAML configuration** for easy project management
- **STM32U5xx HAL/LL driver support**
- **OpenOCD debugging** with latest version
- **VS Code integration** with Cortex-Debug extension
- **Template-free build** (excludes CubeMX template files)

## 📋 Prerequisites

### Hardware Requirements
- **STM32U585I-IOT02A** development board (or compatible STM32U5xx board)
- **ST-Link V3** debugger (built into the board)
- **USB Type-C cable** for power and debugging

### Software Requirements
- **Linux** (Ubuntu 20.04+ recommended)
- **Python 3.6+**
- **Git**

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone <your-repo-url>
cd stm32_timer_freqmeas
```

### 2. Install ARM Toolchain
```bash
# Download and install ARM GCC toolchain
wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2
tar -xjf gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2
sudo mv gcc-arm-none-eabi-10.3-2021.10 /opt/
echo 'export PATH="/opt/gcc-arm-none-eabi-10.3-2021.10/bin:$PATH"' >> ~/.profile
source ~/.profile
```

### 3. Download OpenOCD
```bash
# Download the latest OpenOCD (required for debugging)
wget https://github.com/xpack-dev-tools/openocd-xpack/releases/download/v0.12.0-3/xpack-openocd-0.12.0-3-linux-x64.tar.gz
tar -xzf xpack-openocd-0.12.0-3-linux-x64.tar.gz
```

### 4. Install Python Dependencies
```bash
# Install PyYAML for configuration parsing
pip3 install pyyaml
```

### 5. Build the Project
```bash
# Generate Makefile from config.yaml
python3 makefile_gen.py

# Build the project
make

# Check build size
arm-none-eabi-size build/stm32_timer_freqmeas.elf
```

### 6. Flash and Debug
```bash
# Flash using OpenOCD
./xpack-openocd-0.12.0-3/bin/openocd -f openocd_stm32u585_new.cfg -c "program build/stm32_timer_freqmeas.elf verify reset exit"

# Or use VS Code with Cortex-Debug extension
# Configure launch.json to use the new OpenOCD binary and config
```

## 📁 Project Structure

```
stm32_timer_freqmeas/
├── config.yaml              # Main configuration file
├── makefile_gen.py          # Python Makefile generator
├── openocd_stm32u585_new.cfg # OpenOCD configuration
├── README.md                # This file
├── QUICK_REFERENCE.md       # Quick reference card
├── .gitignore              # Git ignore rules
├── src/                    # Your source code
│   ├── main.c             # Main application
│   └── stm32u5xx_it.c     # Interrupt handlers
├── include/                # Header files
│   ├── main.h
│   ├── stm32u5xx_hal_conf.h
│   └── stm32u5xx_it.h
├── cube/                   # STM32CubeMX generated files
│   ├── Core/              # Core files
│   ├── Drivers/           # HAL/LL drivers
│   └── startup_stm32u585xx.s
└── linker/                # Linker scripts
    └── STM32U585xx_FLASH.ld
```

## ⚙️ Configuration

### config.yaml Structure
The `config.yaml` file contains all build configuration:

```yaml
project:
  name: stm32_timer_freqmeas

board:
  mcu: STM32U585xx
  fpu: fpv5-sp-d16
  float-abi: hard

paths:
  source:                    # Source files
    - src                    # Your custom code
    - cube/Core/Src/stm32u5xx_hal_msp.c
    - cube/Drivers/STM32U5xx_HAL_Driver/Src
  includes:                  # Include paths
    - include
    - cube/Core/Inc
    - cube/Drivers/STM32U5xx_HAL_Driver/Inc

flags:
  defines:                   # Preprocessor definitions
    - USE_HAL_DRIVER
    - STM32U585xx
  cflags:                    # Compiler flags
    - -Wall
    - -g
    - -Og
    - -mcmse                # TrustZone support
```

## 🔧 Build System

### Makefile Generation
The build system uses a Python script to generate Makefiles from YAML configuration:

```bash
python3 makefile_gen.py
```

This creates a `Makefile` with:
- Explicit source file rules (no VPATH conflicts)
- Proper dependency tracking
- Template file exclusion
- Optimized compilation flags

### Build Commands
```bash
make                    # Build the project
make clean             # Clean build artifacts
make VERBOSE=1         # Verbose build output
make size              # Show memory usage
```

## 🐛 Debugging Setup

### VS Code Configuration
Create `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "STM32U5 Debug",
            "type": "cortex-debug",
            "request": "launch",
            "servertype": "openocd",
            "cwd": "${workspaceFolder}",
            "executable": "./build/stm32_timer_freqmeas.elf",
            "device": "STM32U585xx",
            "configFiles": [
                "openocd_stm32u585_new.cfg"
            ],
            "openOCDLaunchCommands": [
                "adapter speed 8000"
            ],
            "svdFile": "./cube/Drivers/CMSIS/Device/ST/STM32U5xx/Include/stm32u585xx.svd"
        }
    ]
}
```

### OpenOCD Configuration
The `openocd_stm32u585_new.cfg` file is configured for:
- ST-Link V3 interface
- STM32U5xx target
- 8MHz adapter speed
- Proper TrustZone support

## 🔍 Common Issues and Solutions

### 1. "Cannot identify target" Error
**Problem**: Old OpenOCD version doesn't support STM32U5xx
**Solution**: Use the provided OpenOCD 0.12.0 binary

### 2. Multiple Definition Errors
**Problem**: Duplicate source files (e.g., main.c in both src/ and cube/)
**Solution**: The build system excludes CubeMX templates automatically

### 3. Linker Errors with _exit
**Problem**: Conflict between syscalls.c and libnosys
**Solution**: Removed `-lnosys` from linker flags in config.yaml

### 4. Wrong main.c Being Used
**Problem**: VPATH finding wrong main.c file
**Solution**: Build system uses explicit paths instead of VPATH

### 5. TrustZone Errors
**Problem**: Missing `-mcmse` flag for secure mode
**Solution**: Added `-mcmse` to compiler flags

## 🛠️ Development Workflow

### Daily Development
1. **Edit source files** in `src/` directory
2. **Update configuration** in `config.yaml` if needed
3. **Regenerate Makefile**: `python3 makefile_gen.py`
4. **Build**: `make clean && make`
5. **Debug**: Use VS Code or OpenOCD directly

### Adding New Source Files
1. Add files to `src/` directory
2. They're automatically included in the build
3. No need to modify Makefile manually

### Adding New Include Paths
1. Edit `config.yaml`
2. Add path to `paths:includes:` section
3. Regenerate Makefile: `python3 makefile_gen.py`

### Adding New Compiler Flags
1. Edit `config.yaml`
2. Add flags to `flags:cflags:` section
3. Regenerate Makefile: `python3 makefile_gen.py`

## 📊 Build Information

### Memory Usage
```
   text    data     bss     dec     hex filename
  40384     456    1232   42072    a458 build/stm32_timer_freqmeas.elf
```

### Build Time
- Clean build: ~30 seconds
- Incremental build: ~5 seconds

## 🔄 Maintenance

### Updating STM32CubeMX Files
1. Generate new files with STM32CubeMX
2. Copy to `cube/` directory
3. Update `config.yaml` if needed
4. Regenerate Makefile: `python3 makefile_gen.py`

### Updating OpenOCD
1. Download new version
2. Replace `xpack-openocd-*/` directory
3. Test with: `./xpack-openocd-*/bin/openocd -f openocd_stm32u585_new.cfg -c "init" -c "targets" -c "exit"`

### Updating ARM Toolchain
1. Download new version
2. Update path in `~/.profile`
3. Test with: `arm-none-eabi-gcc --version`

## 📚 Additional Resources

- [STM32U5xx Reference Manual](https://www.st.com/resource/en/reference_manual/rm0456-stm32u5-series-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [STM32U5xx HAL Documentation](https://www.st.com/en/embedded-software/stm32u5xx-hal-drivers.html)
- [OpenOCD Documentation](http://openocd.org/doc/)
- [ARM GCC Documentation](https://gcc.gnu.org/onlinedocs/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Note**: This project uses a custom build system that prioritizes clarity and maintainability over traditional STM32CubeIDE workflows. It's designed for developers who prefer command-line tools and want full control over their build process. 