# Build Instructions

## Setup

```bash
cd ~/repos/zmk
source .venv/bin/activate
cd ~/repos/zmk/app
```

## Build Commands

### Right Half
```bash
west build -p -d build/right -b cosmos_lemon_wireless -- -DSHIELD=palmctrl_right -DZMK_EXTRA_MODULES=/home/dj/repos/palmctrl-module
```

### Left Half
```bash
west build -p -d build/left -b cosmos_lemon_wireless -- -DSHIELD=palmctrl_left -DZMK_EXTRA_MODULES=/home/dj/repos/palmctrl-module
```

## Output

Firmware files will be in:
- `build/right/zephyr/zmk.uf2`
- `build/left/zephyr/zmk.uf2`

## Copy to Desktop

```bash
cp ~/repos/zmk/app/build/right/zephyr/zmk.uf2 $WINDOWS_HOME/Desktop/right.uf2
cp ~/repos/zmk/app/build/left/zephyr/zmk.uf2 $WINDOWS_HOME/Desktop/left.uf2
```
