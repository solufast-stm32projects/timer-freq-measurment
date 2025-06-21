# 🚀 STM32U5xx Quick Reference Card

## 🔧 Essential Commands

### Build Commands
```bash
# Generate Makefile
python3 makefile_gen.py

# Clean and build
make clean && make

# Build with verbose output
make VERBOSE=1

# Check build size
arm-none-eabi-size build/stm32_timer_freqmeas.elf
```

### Debug Commands
```bash
# Test OpenOCD connection
./xpack-openocd-0.12.0-3/bin/openocd -f openocd_stm32u585_new.cfg -c "init" -c "targets" -c "exit"

# Flash using OpenOCD
./xpack-openocd-0.12.0-3/bin/openocd -f openocd_stm32u585_new.cfg -c "program build/stm32_timer_freqmeas.elf verify reset exit"
```

## 📁 Key Files

| File | Purpose |
|------|---------|
| `config.yaml` | Main configuration |
| `makefile_gen.py` | Makefile generator |
| `openocd_stm32u585_new.cfg` | OpenOCD config |
| `src/main.c` | Your main code |
| `include/main.h` | Your headers |

## ⚡ Daily Workflow

1. **Start session:**
   ```bash
   cd ~/stm32dev/stm32_timer_freqmeas
   python3 makefile_gen.py
   make clean && make
   ```

2. **Debug in VS Code:**
   - Set breakpoints
   - Press F5
   - Use Debug Console

3. **Add new files:**
   - Add to `config.yaml` under `paths:source:`
   - Run `python3 makefile_gen.py`
   - Run `make`

## 🚨 Common Fixes

### Wrong main.c being compiled
```bash
rm -f Makefile
python3 makefile_gen.py
make clean && make
```

### OpenOCD connection issues
```bash
# Use new OpenOCD binary
./xpack-openocd-0.12.0-3/bin/openocd -f openocd_stm32u585_new.cfg
```

### Build errors
```bash
# Check if ARM toolchain is in PATH
echo $PATH | grep arm-none-eabi

# If not, add to PATH
export PATH=$PATH:/usr/bin
```

## 📋 Environment Checklist

- [ ] ARM toolchain installed: `arm-none-eabi-gcc --version`
- [ ] Python YAML installed: `python3 -c "import yaml"`
- [ ] OpenOCD 0.12.0 downloaded: `./xpack-openocd-0.12.0-3/bin/openocd --version`
- [ ] VS Code extensions installed (C/C++, Cortex-Debug)
- [ ] Board connected via ST-Link V3

## 🔗 Important Paths

- **ARM Toolchain:** `/usr/bin/arm-none-eabi-gcc`
- **OpenOCD:** `./xpack-openocd-0.12.0-3/bin/openocd`
- **Build Output:** `build/stm32_timer_freqmeas.elf`
- **Config:** `config.yaml`

---

**💡 Tip:** Bookmark this file and the main README.md for quick access! 