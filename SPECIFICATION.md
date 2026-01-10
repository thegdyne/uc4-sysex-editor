# Faderfox UC4 SysEx Format Specification

**Version:** 1.0  
**Date:** January 2026  
**Status:** Reverse-engineered - Work in Progress

## Overview

The UC4 stores configuration data in SysEx format. Each single-setup dump is approximately 5KB. A full 18-setup dump is approximately 90KB.

## Message Structure

```
F0 00 00 00 [header bytes] [data blocks] 4F XX YY F7
```

- `F0` = SysEx Start
- `00 00 00` = Manufacturer ID placeholder
- `4F XX YY` = Footer
- `F7` = SysEx End

## Header Section

```
41 XX YY  - Setup identifier / Group name char 1
42 XX YY  - Group name char 2  
43 XX YY  - Group name char 3
44 XX YY  - Group name char 4
```

## Data Block Structure

### Block Marker (49 XX YY)
Indicates start of a data block:
- `XX` = Block type identifier (21, 22, etc.)
- `YY` = Block sub-type / group info

### Section Marker (4A XX 10)
Indicates data type within block:
- `4A 20 10` = Encoder Type/Mode/Display data
- `4A 24 10` = Fader data
- `4A 28 10` = Green Button data  
- `4A 2C 10` = Push Button data

### Data Entry (4D XX YY)
Each control's parameters encoded as 3-byte triplet.

## Encoding Details

### Encoder Type (4D XX YY in 4A 20 section)

**XX byte - Low Nibble = Type:**
| Value | Type | Description |
|-------|------|-------------|
| 0 | CCr1 | CC Relative Mode 1 (values 1/127) |
| 1 | CCr2 | CC Relative Mode 2 (values 63/65) |
| 2 | CCAb | CC Absolute 7-bit (0-127) [DEFAULT] |
| 3 | PrGC | Program Change |
| 4 | CCAh | CC 14-bit High Resolution |
| 5 | Pbnd | Pitch Bend (14-bit) |
| 6 | AFtt | Aftertouch (Channel Pressure) |

**XX byte - High Nibble = Flags (TBD)**

**YY byte - High Nibble = MIDI Channel:**
- Value 0-15 = Channel 1-16

**YY byte - Low Nibble = Display Scale:**
| Value | Mode | Description |
|-------|------|-------------|
| 0 | OFF | No display |
| 1 | Std | Standard (0-127) [DEFAULT] |
| 2 | bPoL | Bipolar (-63 to +63) |

### Encoder Acceleration (in Mode section)

**XX byte - Low Nibble = Acceleration:**
| Value | Mode | Description |
|-------|------|-------------|
| 0 | Acc0 | No acceleration |
| 1 | Acc1 | Low acceleration |
| 2 | Acc2 | Medium acceleration |
| 3 | Acc3 | Maximum acceleration [DEFAULT] |

### Button Type (4D XX YY in 4A 28 section)

**XX byte - Low Nibble = Type:**
| Value | Type | Description |
|-------|------|-------------|
| 0 | OFF | Button disabled |
| 1 | notE | Note [DEFAULT] |
| 2 | CC | Control Change |
| 3 | PrGC | Program Change |
| 4 | AFtt | Aftertouch |

### Button Mode

**XX byte encoding:**
| Value | Mode | Description |
|-------|------|-------------|
| 0 | btn | Momentary (press/release) [DEFAULT] |
| 1 | toGL | Toggle (on/off) |

### Fader Type (4D XX YY in 4A 2C section)

**XX byte - Low Nibble = Type:**
| Value | Type | Description |
|-------|------|-------------|
| 0 | CCAb | CC Absolute [DEFAULT] |
| 1 | PrGC | Program Change (assumed) |
| 2 | Pbnd | Pitch Bend ✓ |
| 3 | AFtt | Aftertouch ✓ |

**Note:** Fader type/mode data is stored in the 4A 2C section, same location as push button data. Entry position determines which fader (1-8).

### Fader Mode
| Value | Mode | Description |
|-------|------|-------------|
| ? | JMP | Jump (immediate) [DEFAULT per manual] |
| ? | SnAP | Snap (catch value first) |

*Note: Fader mode encoding could not be confirmed through testing. The mode may be encoded in the high nibble of the type byte or in a different section.*

### Value Encoding (Lower/Upper Values)

Values 0-127 encoded in 4D XX YY format:
- XX low nibble = Value >> 4 (high nibble)
- YY low nibble = Value & 0x0F (low nibble)

**Examples:**
| Value | Decimal | Encoding |
|-------|---------|----------|
| 0x00 | 0 | 4D 20 10 |
| 0x40 | 64 | 4D 24 10 |
| 0x7F | 127 | 4D 27 1F |

### CC Number Encoding

CC numbers 0-127 use same encoding as values:
- XX low nibble = CC >> 4
- YY low nibble = CC & 0x0F

Combined with channel in YY high nibble.

### MIDI Channel Encoding

Channel 1-16 encoded as 0-15 in YY high nibble:
- Channel 1 = 0x0_
- Channel 5 = 0x4_
- Channel 9 = 0x8_
- Channel 16 = 0xF_

## Block End Markers

- `4B XX YY` = Fader 9 special entry
- `4C XX YY` = End of section marker

## Group Structure

Each setup contains 8 encoder groups and 8 fader/button groups.

Block identifiers:
- `49 21 14` = Special header block
- `49 21 17` = ?
- `49 21 1C` = Group 1 data
- `49 21 1D` = Group 2 data  
- `49 21 1E` = Group 3 data
- `49 21 1F` = Group 4 data
- `49 22 10` = Groups 5-8 data

## Test Data Summary

### Phase 1 Tests (Types, Modes, Values)

| Test | Parameter Changed | Position | From | To |
|------|-------------------|----------|------|-----|
| 05 | Enc Type → AFtt | 395 | 22 | 26 |
| 06 | Enc Type → CCAh | 395 | 22 | 24 |
| 07 | Enc Mode → Acc0 | 1331 | 23 | 20 |
| 08 | Enc Mode → Acc1 | 1331 | 23 | 21 |
| 09 | Enc Mode → Acc2 | 1331 | 23 | 22 |
| 11 | Btn Type → CC | 2735 | 21 | 22 |
| 12 | Btn Type → PrGC | 2735 | 21 | 23 |
| 13 | Btn Type → AFtt | 2735 | 21 | 24 |
| 14 | Btn Mode → toGL | 3671 | 20 | 21 |
| 15 | Enc Lower → 64 | 3203 | 20 | 24 |
| 16 | Enc Upper → 64 | 3437 | 27 | 24 |
| 17 | Enc Disp → bPoL | 1332 | 11 | 12 |
| 18 | Enc Disp → OFF | 1332 | 11 | 10 |

### Phase 2 Tests (Push Buttons, Faders, Names)

| Test | Parameter Changed | Position | From | To | Notes |
|------|-------------------|----------|------|-----|-------|
| 19 | Fad1 → SnAP | - | - | - | No change - SNAP is default |
| 20 | Pbt1 Type → CC | 1565 | 21 | 22 | Push btn type confirmed |
| 21 | Pbt1 Type → notE | - | - | - | No change - Note is default |
| 22 | Pbt1 Mode → toGL | 2501 | 20 | 21 | Push btn mode confirmed |
| 23 | Group Name | 23,24,27,30,33 | various | various | Custom 7-seg encoding |
| 24 | Fad1 Type → Pbnd | 3905 | 20 | 22 | Test error - wrong control |
| 25 | Btn1 Lower → 64 | 3203 | 20 | 24 | Value encoding confirmed |

### Phase 3 Tests (Fader Types - Corrected)

| Test | Parameter Changed | Position | From | To | Notes |
|------|-------------------|----------|------|-----|-------|
| 26 | Fad1 Type → Pbnd | 3905 | 20 | 22 | In Group 4, 4A 2C section |
| 27 | Fad1 Mode → JMP | - | - | - | No change - JMP is default |
| 28 | Fad1 Type → AFtt | 3905 | 20 | 23 | Confirmed type encoding |

**Key Discovery:** Fader type data is stored in the 4A 2C section. The default fader group after factory reset appears to be Group 4, not Group 1.

## Data Block Structure

Each group has 4 separate data blocks with same group ID but different subsections:

```
49 21 1C 4A 20 ... (Group 1 Encoder data)
49 21 1C 4A 24 ... (Group 1 Fader data)
49 21 1C 4A 28 ... (Group 1 Green Button data)
49 21 1C 4A 2C ... (Group 1 Push Button data)
```

Group ID encoding (3rd byte of 49 marker):
- 1C = Group 1
- 1D = Group 2
- 1E = Group 3
- 1F = Group 4
- 22 10 = Groups 5-8

## Confirmed Defaults

- Fader Mode: JMP (per manual, but see note below)
- Fader Type: CCAb (CC Absolute)
- Push Button Type: Note
- Push Button Mode: Momentary (btn)
- Encoder Type: CCAb (CC Absolute)
- Encoder Acceleration: Acc3 (max)
- Display Scale: Std

**Note:** Manual states JMP is default for setups 1-16, SNAP for setups 17-18. Testing showed no change when switching modes, suggesting either the mode is stored elsewhere or the encoding is combined with type.

## Known Limitations

1. Fader mode encoding not confirmed (SNAP vs JMP)
2. Group name uses custom 7-segment character encoding (not ASCII)
3. 14-bit high-res CC LSB handling unclear
4. Fader 9 special encoding (4B marker) needs more testing
5. Default fader group after factory reset is Group 4, not Group 1

## References

- UC4 User Manual V03
- Test dumps from systematic parameter changes
- Previous reverse engineering session data
