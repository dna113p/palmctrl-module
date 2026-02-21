# Gaming Layer Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a toggleable GAMING layer (layer 7) to palmctrl.keymap that remaps the left thumb cluster for comfortable gaming and exits on any right-half keypress.

**Architecture:** Add a `GAMING` layer constant, a ZMK combo on LGUI+td_raise positions to `&tog GAMING`, and a new `gaming_layer` block in the keymap. Left half is mostly `&trans` with three thumb-row keys remapped; right half is all `&tog GAMING`.

**Tech Stack:** ZMK firmware, Devicetree/Kconfig, `.keymap` file (C preprocessor + devicetree syntax)

---

### Task 1: Add GAMING layer constant and combo

**Files:**
- Modify: `boards/shields/palmctrl/palmctrl.keymap`

**Context:**
The keymap already defines layer constants at the top (`#define DEFAULT 0` … `#define RGB_CTRL 6`). ZMK combos are declared in a `/ { combos { … }; };` block. The left thumb row physical positions (0-indexed across the whole matrix) are:

```
Row 4 (thumb row), left half columns 0–5:
  pos 48 = LCTRL
  pos 49 = LGUI
  pos 50 = td_rgb
  pos 51 = LALT
  pos 52 = td_lower
  pos 53 = td_raise
```

(Row 4 starts at position 48 because rows 0–3 each have 12 keys = 48 keys before row 4.)

**Step 1: Add the GAMING layer constant**

In `palmctrl.keymap`, after line 14 (`#define RGB_CTRL   6`), add:

```c
#define GAMING      7
```

**Step 2: Update zmk,meta layers list**

Find this line (currently line 46):
```c
        layers = <DEFAULT LOWER RAISE NUMPAD MOUSE BT_CTRL RGB_CTRL>;
```
Change it to:
```c
        layers = <DEFAULT LOWER RAISE NUMPAD MOUSE BT_CTRL RGB_CTRL GAMING>;
```

**Step 3: Add the combo block**

Inside the `/ { … };` block, after the closing `};` of the `behaviors { … };` block and before `zmk,meta`, add:

```c
    combos {
        compatible = "zmk,combos";
        combo_gaming {
            timeout-ms = <50>;
            key-positions = <49 53>;
            bindings = <&tog GAMING>;
            layers = <DEFAULT GAMING>;
        };
    };
```

This fires `&tog GAMING` when LGUI (pos 49) and td_raise (pos 53) are pressed simultaneously, and only activates from DEFAULT or GAMING layers.

**Step 4: Commit**

```bash
git add boards/shields/palmctrl/palmctrl.keymap
git commit -m "feat: add GAMING layer constant and toggle combo"
```

---

### Task 2: Add the gaming_layer keymap block

**Files:**
- Modify: `boards/shields/palmctrl/palmctrl.keymap`

**Context:**
The keymap has 5 rows × 12 columns = 60 bindings per layer. The split is left cols 0–5, right cols 6–11. The thumb row is row 4.

Left half thumb row positions in the binding list (row 4, cols 0–5 = positions 48–53 in the flat list):
- Col 0 (pos 48): LCTRL → `&trans`
- Col 1 (pos 49): LGUI → `&trans`
- Col 2 (pos 50): td_rgb → `&kp F14`
- Col 3 (pos 51): LALT → `&trans`
- Col 4 (pos 52): td_lower → `&kp SPACE`
- Col 5 (pos 53): td_raise → `&kp F13`

Right half (all 30 keys) → `&tog GAMING`

**Step 1: Add gaming_layer block**

After the closing `};` of `rgb_ctrl_layer` and before the closing `};` of `keymap`, add:

```c
        gaming_layer {
            display-name = "GAMING";
            bindings = <
                // Row 0: number row - pass through
                &trans  &trans  &trans  &trans  &trans  &trans      &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING
                // Row 1: QWERTY top - pass through
                &trans  &trans  &trans  &trans  &trans  &trans      &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING
                // Row 2: home row - pass through
                &trans  &trans  &trans  &trans  &trans  &trans      &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING
                // Row 3: bottom row - pass through
                &trans  &trans  &trans  &trans  &trans  &trans      &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING

                // Row 4: thumb row
                // Left: LCTRL LGUI F14(td_rgb) LALT SPACE(td_lower) F13(td_raise)
                &trans  &trans  &kp F14  &trans  &kp SPACE  &kp F13      &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING  &tog GAMING
            >;
        };
```

**Step 2: Commit**

```bash
git add boards/shields/palmctrl/palmctrl.keymap
git commit -m "feat: add gaming_layer with thumb remap and right-half exit"
```

---

### Task 3: Build and verify

**Context:**
Build instructions are in `AGENTS.md`. Activate the ZMK venv first. Build only the left half first to catch errors faster (gaming layer primarily affects left hand behavior; right half build is a sanity check).

**Step 1: Activate venv and build left half**

```bash
source ~/repos/zmk/.venv/bin/activate
cd ~/repos/zmk/app
west build -p -d build/left -b cosmos_lemon_wireless -- -DSHIELD=palmctrl_left -DZMK_EXTRA_MODULES=/home/dj/repos/palmctrl-module
```

Expected: build completes with no errors.

**Step 2: Build right half**

```bash
west build -p -d build/right -b cosmos_lemon_wireless -- -DSHIELD=palmctrl_right -DZMK_EXTRA_MODULES=/home/dj/repos/palmctrl-module
```

Expected: build completes with no errors.

**Step 3: If build fails**

Common ZMK keymap errors:
- Wrong binding count per layer (must be exactly 60)
- Unknown key code (check `<dt-bindings/zmk/keys.h>` for F13/F14 — they are `F13` and `F14`)
- Combo key-positions out of range (max position = 59 for a 60-key board)

Fix the error in `palmctrl.keymap` and rebuild.

**Step 4: Copy firmware to desktop**

```bash
cp ~/repos/zmk/app/build/right/zephyr/zmk.uf2 $WINDOWS_HOME/Desktop/right.uf2
cp ~/repos/zmk/app/build/left/zephyr/zmk.uf2 $WINDOWS_HOME/Desktop/left.uf2
```

**Step 5: Final commit**

```bash
git add boards/shields/palmctrl/palmctrl.keymap
git commit -m "feat: gaming layer complete and builds successfully"
```

---

## Manual Test Checklist (on hardware)

1. **Enter gaming mode**: Press LGUI + td_raise simultaneously → layer indicator or key behavior should change
2. **SPACE**: Press td_lower position → should send SPACE
3. **F14**: Press td_rgb position → should send F14 (verify with key tester)
4. **F13**: Press td_raise position → should send F13
5. **Left WASD**: Press W/A/S/D → should still send W/A/S/D (trans to DEFAULT)
6. **Exit gaming mode**: Press any right-half key → should exit GAMING layer
7. **Re-enter**: LGUI + td_raise again → should re-enter GAMING
