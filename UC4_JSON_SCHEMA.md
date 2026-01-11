# UC4 Editor JSON Schema Reference

Quick reference for generating valid UC4 configuration JSON files.

---

## File Structure

```json
{
  "format": "uc4-editor",
  "version": "1.0",
  "exported": "ISO-8601 timestamp",
  "setups": [ /* 18 setups, index 0-17 */ ]
}
```

**Setup** (×18):
```json
{
  "index": 0-17,
  "groups": [ /* 8 groups, index 0-7 */ ]
}
```

**Group** (×8 per setup):
```json
{
  "index": 0-7,
  "name": "GrP1",           // 4 chars, displayed on UC4
  "encoders": [],           // 8 encoders
  "pushButtons": [],        // 8 push buttons (under encoders)
  "greenButtons": [],       // 8 green buttons
  "faders": [],             // 8 faders
  "fader9": {}              // 1 expression fader
}
```

---

## Control Parameters

### Encoders (8 per group)

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| `channel` | int | 1-16 | MIDI channel |
| `type` | int | 0-6 | Message type (see below) |
| `cc` | int | 0-127 | CC number (0-31 for CCAh high-res) |
| `min` | int | 0-127 | Minimum output value |
| `max` | int | 0-127 | Maximum output value |
| `acc` | int | 0-3 | Acceleration sensitivity |
| `display` | int | 0-2 | Display mode |

**Encoder Types:**
| Value | Name | Description |
|-------|------|-------------|
| 0 | CCr1 | Relative mode 1 (sends 1/127) |
| 1 | CCr2 | Relative mode 2 (sends 63/65) |
| 2 | CCAb | **Absolute 7-bit CC (0-127)** — most common |
| 3 | PrGC | Program Change |
| 4 | CCAh | Absolute 14-bit high-res (CC N + CC N+32) |
| 5 | Pbnd | Pitch Bend (14-bit) |
| 6 | AFtt | Aftertouch / Channel Pressure |

**Acceleration (acc):**
| Value | Name | Description |
|-------|------|-------------|
| 0 | Acc0 | No acceleration |
| 1 | Acc1 | Low acceleration |
| 2 | Acc2 | Medium acceleration |
| 3 | Acc3 | **Maximum acceleration** — factory default |

**Display Mode (display):**
| Value | Name | Description |
|-------|------|-------------|
| 0 | OFF | No display feedback |
| 1 | Std | Standard 0-127 display |
| 2 | bPoL | Bipolar display (-63 to +63) |

---

### Faders 1-8 (8 per group)

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| `channel` | int | 1-16 | MIDI channel |
| `type` | int | 0-3 | Message type (see below) |
| `cc` | int | 0-127 | CC number |
| `min` | int | 0-127 | Minimum output value |
| `max` | int | 0-127 | Maximum output value |
| `mode` | int | 0-1 | Pickup mode |
| `display` | int | 0-2 | Display mode (same as encoder) |

**Fader Types:**
| Value | Name | Description |
|-------|------|-------------|
| 0 | CCAb | **Absolute 7-bit CC** — most common |
| 1 | PrGC | Program Change |
| 2 | Pbnd | Pitch Bend (14-bit) |
| 3 | AFtt | Aftertouch / Channel Pressure |

**Fader Mode:**
| Value | Name | Description |
|-------|------|-------------|
| 0 | Jump | Sends value immediately on move |
| 1 | Snap | Waits until fader matches last value (pickup) |

---

### Push Buttons (8 per group)

Under each encoder — triggered by pressing the encoder.

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| `channel` | int | 1-16 | MIDI channel |
| `typeNibble` | int | 0/16/32/48/64 | Message type (see below) |
| `cc` | int | 0-127 | CC or Note number |
| `lower` | int | 0-127 | Value on release (momentary) or off (toggle) |
| `upper` | int | 0-127 | Value on press (momentary) or on (toggle) |
| `mode` | int | 0-1 | Button behavior |
| `display` | int | 0-1 | Display mode |

---

### Green Buttons (8 per group)

Row of 8 illuminated buttons below the faders.

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| `channel` | int | 1-16 | MIDI channel |
| `typeNibble` | int | 0/16/32/48/64 | Message type (see below) |
| `cc` | int | 0-127 | CC or Note number |
| `lower` | int | 0-127 | Value on release/off |
| `upper` | int | 0-127 | Value on press/on |
| `mode` | int | 0-1 | Button behavior |
| `display` | int | 0-2 | LED display mode |

**Button typeNibble (both push and green):**
| Value | Name | Description |
|-------|------|-------------|
| 0 | OFF | Button disabled |
| 16 | Note | **Note On/Off messages** — factory default |
| 32 | CC | Control Change messages |
| 48 | PrGC | Program Change |
| 64 | AFtt | Aftertouch |

**Button Mode:**
| Value | Name | Description |
|-------|------|-------------|
| 0 | Momentary | Upper on press, lower on release |
| 1 | Toggle | Alternates between upper/lower each press |

**Green Button Display (LED):**
| Value | Name | Description |
|-------|------|-------------|
| 0 | OFF | LED never lights |
| 1 | Std | LED reflects button state + external feedback |
| 2 | EXt | LED controlled only by external MIDI feedback |

---

### Fader 9 (1 per group)

Expression pedal input / 9th fader. Simplified parameters.

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| `channel` | int | 1-16 | MIDI channel |
| `cc` | int | 0-127 | CC number |
| `min` | int | 0-127 | Minimum output value |
| `max` | int | 0-127 | Maximum output value |
| `mode` | int | 0-1 | 0=Jump, 1=Snap |

---

## Common Patterns

### Channel-per-group
Each group uses a different MIDI channel, same CCs:
```
Group 1 → Channel 1, CCs 1-8
Group 2 → Channel 2, CCs 1-8
...
```

### CC-per-group  
All groups on same channel, different CCs:
```
Group 1 → Channel 1, CCs 1-8
Group 2 → Channel 1, CCs 9-16
...
```

### Typical Defaults
```json
// Encoder - standard absolute CC
{ "channel": 1, "type": 2, "cc": 1, "min": 0, "max": 127, "acc": 3, "display": 1 }

// Fader - standard absolute CC  
{ "channel": 1, "type": 0, "cc": 1, "min": 0, "max": 127, "mode": 0, "display": 1 }

// Button - CC toggle
{ "channel": 1, "typeNibble": 32, "cc": 1, "lower": 0, "upper": 127, "mode": 1, "display": 1 }

// Button - Note momentary
{ "channel": 1, "typeNibble": 16, "cc": 60, "lower": 0, "upper": 127, "mode": 0, "display": 1 }
```

---

## Totals

| Level | Count |
|-------|-------|
| Setups | 18 |
| Groups per setup | 8 |
| Encoders per group | 8 |
| Push buttons per group | 8 |
| Green buttons per group | 8 |
| Faders per group | 8 |
| Fader 9 per group | 1 |
| **Total controls per setup** | **264** |
| **Total controls in dump** | **4,752** |

---

## Notes

- **Inverted ranges**: `min` > `max` is valid (reverses control direction)
- **14-bit CC (CCAh)**: CC must be 0-31; LSB auto-assigned to CC+32
- **Group names**: 4 characters, set on hardware, shown in editor
- **Setups 17-18** (index 16-17): Factory presets for Ableton Live
