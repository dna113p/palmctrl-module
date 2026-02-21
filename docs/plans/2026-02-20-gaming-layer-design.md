# Gaming Layer Design

**Date:** 2026-02-20

## Problem

In gaming mode, the thumb cluster and modifier row are awkward. Tap-dance and layer-momentary behaviors interfere with gameplay. The user needs:
- The td_lower position to send `SPACE`
- The td_rgb position to send `F14` (in-game remappable)
- The td_raise position to send `F13` (in-game remappable)
- All other left-half keys to pass through to DEFAULT unchanged
- Any right-half keypress to exit gaming mode

## Design

### New Layer

`GAMING = 7` — added after `RGB_CTRL = 6`.

### Toggle Mechanism

A ZMK **combo** defined on the simultaneous press of:
- Left thumb `LGUI` key (position 49)
- Left thumb `td_raise` key (position 52)

The combo fires `&tog GAMING`, which toggles the layer on/off persistently.

### Left Half Bindings (GAMING layer)

| Row | Behavior |
|-----|----------|
| Rows 0–3 (letters/numbers) | `&trans` — passes through to DEFAULT |
| Thumb row modifiers (LCTRL, LGUI) | `&trans` |
| td_rgb position | `&kp F14` |
| LALT position | `&trans` |
| td_lower position | `&kp SPACE` |
| td_raise position | `&kp F13` |

### Right Half Bindings (GAMING layer)

All 30 right-half keys → `&tog GAMING`

Pressing any right-half key exits gaming mode. This acts as a natural "I'm done gaming, back to keyboard" trigger.

## Key Position Reference (left thumb row = row 4)

```
Row 4 (thumb): LCTRL  LGUI  td_rgb  LALT  td_lower  td_raise
Positions:       48     49    50      51      52        53
```

Combo keys: positions 49 (LGUI) + 53 (td_raise) pressed simultaneously.

## Trade-offs Considered

- **Combo vs tap-dance toggle**: Combo is more reliable for simultaneous key detection; tap-dance requires precise timing and could misfire during normal typing near the gaming toggle.
- **Right-half `&tog` vs single exit key**: Any-key exit is the most natural — you reach for the keyboard and the mode switches automatically.
- **F13/F14 as placeholder keys**: These are not bound to system functions on modern OSes and are ideal for in-game rebinding.
