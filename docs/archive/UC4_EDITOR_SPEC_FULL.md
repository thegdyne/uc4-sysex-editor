# UC4 SysEx Editor - Complete Application Specification

## Purpose
Web-based editor to configure Faderfox UC4 MIDI controller via SysEx dumps. Provides full access to all programmable parameters without requiring the hardware's on-device edit mode.

## Target Device
Faderfox UC4 micromodul - universal MIDI controller with:
- 8 push encoders (without detents, ~36 pulses resolution)
- 8 encoder push buttons (programmable separately from encoder rotation)
- 9 faders (fader 9 has simplified parameters)
- 8 green buttons with LEDs
- 4-digit 7-segment display
- 8 switchable groups per setup (independent for encoders vs faders/buttons)
- 18 user setups (17-18 preconfigured for Ableton Live)

## Data Architecture

### SysEx Encoding (Critical Implementation Detail)
All values are stored as **nibble-packed triplets**, not raw bytes:
```
Storage: 4D [highNibbleCarrier] [lowNibbleCarrier]
Decode:  value = ((high & 0x0F) << 4) | (low & 0x0F)
Encode:  high = 0x20 | ((value >> 4) & 0x0F)
         low  = 0x10 | (value & 0x0F)
```

The UI operates on decoded 0-127 values; import/export handles triplet conversion.

### Memory Model
- **Full dump size**: 100,640 bytes (all 18 setups)
- **v1 targets "Send All Setups" format only** (100,640 bytes)
- The UC4 manual also documents single-setup send/receive (Sndc command), but that message format is not implemented in v1
- Load complete dump into memory, edit one setup at a time, export all 18

### Hierarchy
```
Dump (100,640 bytes)
â””â”€â”€ Setup (Ã—18)
    â”œâ”€â”€ Encoder Groups (Ã—8)
    â”‚   â”œâ”€â”€ Encoders (Ã—8 per group = 64 per setup)
    â”‚   â””â”€â”€ Push Buttons (Ã—8 per group = 64 per setup)
    â”œâ”€â”€ Fader/Button Groups (Ã—8, independent from encoder groups)
    â”‚   â”œâ”€â”€ Faders 1-8 (Ã—8 per group = 64 per setup)
    â”‚   â”œâ”€â”€ Fader 9 (Ã—1 per group = 8 per setup)
    â”‚   â””â”€â”€ Green Buttons (Ã—8 per group = 64 per setup)
    â””â”€â”€ Group Names (Ã—8, 4 characters each)

Total controls per setup: 264 (33 controls Ã— 8 groups)
```

### Group Independence
The UC4 maintains **separate group selections** for:
- Encoder group (Shift + Encoder 1-8 on hardware)
- Fader/Button group (Shift + Green Button 1-8 on hardware)

The editor must reflect this with independent group selectors.

---

## Control Parameters - Complete Reference

### Encoders (8 per group)

| Parameter | Range | Display Values | Description |
|-----------|-------|----------------|-------------|
| **MIDI Channel** | 1-16 | Ch01-Ch16 | Target MIDI channel |
| **CC Number** | 0-127 | n000-n127 | Controller number (0-31 for 14-bit MSB, LSB auto-assigned +32) |
| **Type** | 7 options | See below | Command type sent on rotation |
| **Acceleration** | 4 levels | Acc0-Acc3 | Velocity scaling for faster turns |
| **Display Mode** | 3 options | OFF/Std/bPoL | How value appears on 7-segment |
| **Min Value** | 0-127 | L000-L127 | Lower bound of output range |
| **Max Value** | 0-127 | U000-U127 | Upper bound of output range |

#### Encoder Types
| Type | Code | Description | Value Range |
|------|------|-------------|-------------|
| CCr1 | 0 | Relative mode 1 | Sends 1 / 127 (direction depends on host interpretation) |
| CCr2 | 1 | Relative mode 2 | Sends 63 / 65 (direction depends on host interpretation) |
| CCAb | 2 | Absolute 7-bit | 0-127 (factory default) |
| PrGC | 3 | Program Change | 0-127 |
| CCAh | 4 | Absolute 14-bit high-res | 0-16383 (uses CC# and CC#+32) |
| Pbnd | 5 | Pitch Bend | 0-16383 (14-bit) |
| AFtt | 6 | Aftertouch (Channel Pressure) | 0-127 |

#### Encoder Acceleration Modes
| Mode | Behaviour |
|------|-----------|
| Acc0 | No acceleration - consistent increment per pulse |
| Acc1 | Low acceleration - slight speedup on fast turns |
| Acc2 | Medium acceleration |
| Acc3 | Maximum acceleration - fast sweeps across full range |

Note: Hold Shift while turning encoder on hardware to temporarily disable acceleration for precise control.

#### Encoder Display Modes
| Mode | Behaviour |
|------|-----------|
| OFF | No display feedback |
| Std | Standard 0-127 display |
| bPoL | Bipolar display (-63 to +63) |

---

### Encoder Push Buttons (8 per group)

Each encoder's push function is separately programmable from its rotation.

| Parameter | Range | Display Values | Description |
|-----------|-------|----------------|-------------|
| **MIDI Channel** | 1-16 | Ch01-Ch16 | Target MIDI channel |
| **CC/Note Number** | 0-127 | n000-n127 | Controller or note number |
| **Type** | 5 options | See below | Command type sent on push |
| **Mode** | 2 options | btn/toGL | Momentary or toggle behaviour |
| **Lower Value** | 0-127 | L000-L127 | Sent on release (momentary) or off (toggle) |
| **Upper Value** | 0-127 | U000-U127 | Sent on press (momentary) or on (toggle) |

#### Push Button Types
| Type | Nibble Value | Description | Encoding Status |
|------|--------------|-------------|-----------------|
| OFF | 0x00 | Button disabled - no command sent | Unverified |
| Note | 0x10 | Note On/Off messages | **Confirmed** |
| CC | 0x20 | Control Change messages | **Confirmed** |
| PrGC | 0x30? | Program Change (upper=press, lower=release) | Unverified |
| AFtt | 0x40? | Aftertouch (Channel Pressure) | Unverified |

**Note**: Only Note (0x10) and CC (0x20) nibble values are confirmed via dump analysis. These are stored as-is â€” do NOT shift. Other types exist per the manual but their nibble values need verification.

**Mode+Display byte (Section 0x1E Bank 0x40):**
- Upper nibble: Mode (0x00=Momentary, 0x10=Toggle)
- Lower nibble: Display mode

#### Push Button Modes
| Mode | Display | Behaviour |
|------|---------|-----------|
| Momentary | btn | Upper value on press, lower value on release |
| Toggle | toGL | Alternates between upper and lower on each press |

**Tip**: Set lower=upper for Program Change to send single value on press only.

---

### Green Buttons (8 per group)

| Parameter | Range | Display Values | Description |
|-----------|-------|----------------|-------------|
| **MIDI Channel** | 1-16 | Ch01-Ch16 | Target MIDI channel |
| **CC/Note Number** | 0-127 | n000-n127 | Controller or note number |
| **Type** | 5 options | See below | Command type |
| **Mode** | 2 options | btn/toGL | Momentary or toggle |
| **Display Mode** | 3 options | OFF/Std/EXt | LED feedback mode |
| **Lower Value** | 0-127 | L000-L127 | Off/release value |
| **Upper Value** | 0-127 | U000-U127 | On/press value |

#### Green Button Types
Same as Push Button types: OFF, Note, CC, PrGC, AFtt

**Encoding**: Same nibble values as push buttons (Note=0x10, CC=0x20 confirmed; others unverified). Store nibble as-is, never shift.

**Mode+Display byte (Section 0x1F Bank 0x80):**
- Upper nibble: Mode (0x00=Momentary, 0x10=Toggle)
- Lower nibble: Display mode (0=OFF, 1=Std, 2=EXt)

#### Green Button Display (LED) Modes
| Mode | Behaviour |
|------|-----------|
| OFF | LED never illuminates |
| Std | LED controlled by device and external feedback |
| EXt | LED controlled only by external MIDI feedback |

**Note**: External control uses same CC/Note number to reflect state back to button LED.

---

### Faders 1-8 (8 per group)

| Parameter | Range | Display Values | Description |
|-----------|-------|----------------|-------------|
| **MIDI Channel** | 1-16 | Ch01-Ch16 | Target MIDI channel |
| **CC Number** | 0-127 | n000-n127 | Controller number |
| **Type** | 4 options | See below | Command type |
| **Mode** | 2 options | SnAP/JMP | Pickup behaviour |
| **Display Mode** | 3 options | OFF/Std/bPoL | Display feedback |
| **Min Value** | 0-127 | L000-L127 | Lower output bound |
| **Max Value** | 0-127 | U000-U127 | Upper output bound |

#### Fader Types
| Type | Code | Description |
|------|------|-------------|
| CCAb | 0 | Control Change 7-bit absolute (factory default) |
| PrGC | 1 | Program Change |
| Pbnd | 2 | Pitch Bend (14-bit) |
| AFtt | 3 | Aftertouch (Channel Pressure) |

Note: Fader type codes (0-3) differ from encoder type codes (0-6). Faders do NOT support relative CC modes.

#### Fader Modes
| Mode | Display | Behaviour |
|------|---------|-----------|
| Snap | SnAP | Must "catch" the current value before sending - prevents jumps |
| Jump | JMP | Sends immediately when moved - may cause value jumps |

**Mode+Display byte (Section 0x20 Bank 0xC0):**
- Upper nibble: Mode (0x00=Jump, 0x10=Snap)
- Lower nibble: Display mode (0=OFF, 1=Std, 2=bPoL)

**Tip**: Hold Shift while moving fader on hardware to temporarily swap modes.

---

### Fader 9 (1 per group)

Fader 9 has a simplified parameter set - always sends CC (no type selection).

| Parameter | Range | Display Values | Description |
|-----------|-------|----------------|-------------|
| **MIDI Channel** | 1-16 | Ch01-Ch16 | Target MIDI channel |
| **CC Number** | 0-127 | n000-n127 | Controller number |
| **Min Value** | 0-127 | L000-L127 | Lower output bound |
| **Max Value** | 0-127 | U000-U127 | Upper output bound |
| **Mode** | 2 options | SnAP/JMP | Pickup behaviour |

**Fader 9 Mode encoding (Section 0x17 Bank 0x00, offset [4] per group):**
- 1 (0x01) = Jump
- 17 (0x11) = Snap
- Bit 4 (0x10) = Snap flag

---

### Group Names (8 per setup)

| Parameter | Range | Description |
|-----------|-------|-------------|
| **Name** | 4 characters | Displayed on 7-segment when group selected |

**Storage (Section 0x14 Bank 0x80):**
- 32 values = 4 characters Ã— 8 groups
- Character encoding: 7-segment display codes (character map TBD)

Character set: Limited to 7-segment displayable characters (0-9, A-Z subset, some symbols).
Factory defaults: GrP1, GrP2, GrP3, GrP4, GrP5, GrP6, GrP7, GrP8
Ableton setups: Snd1, Snd2, Snd3, Snd4, trAC, RAC, PAn, GLOb

---

## Setups

### Setup Structure
- 18 independent setups stored in device
- Each setup contains complete configuration for all controls and groups
- Setup selection preserved across power cycles

### Special Setups
| Setup | Purpose |
|-------|---------|
| 1-16 | User-configurable, factory defaults use sequential CC numbers |
| 17 | Ableton Live tracks 1-8 (preconfigured) |
| 18 | Ableton Live tracks 9-16 (preconfigured) |

### Factory Default CC Assignments (Setups 1-16)

#### Encoders
- Channel: Matches setup number (Setup 1 = Ch1, etc.)
- Type: CCAb (absolute)
- Mode: Acc3 (max acceleration)
- Display: Std

| Group | CC Numbers |
|-------|------------|
| 1 | 8-15 |
| 2 | 16-23 |
| 3 | 24-31 |
| 4 | 32-39 |
| 5 | 72-79 |
| 6 | 80-87 |
| 7 | 88-95 |
| 8 | 96-103 |

#### Push Buttons
- Channel: Matches setup number
- Type: Note
- Mode: Momentary

| Group | Note Numbers |
|-------|--------------|
| 1 | 0-7 |
| 2 | 8-15 |
| 3 | 16-23 |
| 4 | 24-31 |
| 5 | 32-39 |
| 6 | 40-47 |
| 7 | 48-55 |
| 8 | 56-63 |

#### Green Buttons
- Channel: Matches setup number
- Type: Note
- Mode: Momentary
- Display: Std

| Group | Note Numbers |
|-------|--------------|
| 1 | 64-71 |
| 2 | 72-79 |
| 3 | 80-87 |
| 4 | 88-95 |
| 5 | 96-103 |
| 6 | 104-111 |
| 7 | 112-119 |
| 8 | 120-127 |

#### Faders 1-8
- Channel: Matches setup number
- Type: CCAb
- Mode: Jump
- Display: Std

| Group | CC Numbers |
|-------|------------|
| 1 | 32-39 |
| 2 | 40-47 |
| 3 | 48-55 |
| 4 | 56-63 |
| 5-8 | 104-111 |

#### Fader 9
- All groups: CC 112
- Channel: Matches setup number
- Mode: Jump

---

## User Interface

### Navigation Structure
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ [Import SysEx] [Export SysEx] [Import JSON] [Export JSON]       â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Setup: [â–¼ 1-18]                                                 â”‚
â”‚                                                                 â”‚
â”‚ Encoder Group: [1][2][3][4][5][6][7][8]  â†’ GrP1                â”‚
â”‚ Fader/Btn Group: [1][2][3][4][5][6][7][8]  â†’ GrP1              â”‚
â”‚                                                                 â”‚
â”‚ Group Names: [GrP1][GrP2][GrP3][GrP4][GrP5][GrP6][GrP7][GrP8]  â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ [Focused View] [Overview]                                       â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Note**: There is one set of 8 group names per setup (displayed on encoder-group selection per the manual; fader/button group uses LED indication). The editor may display that name next to either selector as a convenience label. Group name byte locations are NOT YET MAPPED in the protocol spec.

### Focused View (Primary Editor)

Edit one group at a time with full parameter access.

#### Encoders Section
```
ENCODERS (Group 1)
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Enc 1    â”‚ Enc 2    â”‚ Enc 3    â”‚ Enc 4    â”‚ Enc 5    â”‚ Enc 6    â”‚ Enc 7    â”‚ Enc 8    â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚
â”‚ CC  [8 ] â”‚ CC  [9 ] â”‚ CC  [10] â”‚ CC  [11] â”‚ CC  [12] â”‚ CC  [13] â”‚ CC  [14] â”‚ CC  [15] â”‚
â”‚ Type[CCAbâ–¼]â”‚ Type[CCAbâ–¼]â”‚        â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Acc [Acc3â–¼]â”‚ Acc [Acc3â–¼]â”‚        â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Disp[Stdâ–¼]â”‚ Disp[Stdâ–¼]â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Min [0  ] â”‚ Min [0  ] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Max [127] â”‚ Max [127] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

#### Push Buttons Section
```
PUSH BUTTONS (Group 1)
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Push 1   â”‚ Push 2   â”‚ Push 3   â”‚ Push 4   â”‚ Push 5   â”‚ Push 6   â”‚ Push 7   â”‚ Push 8   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚          â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Note[0 ] â”‚ Note[1 ] â”‚          â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Type[Noteâ–¼]â”‚Type[Noteâ–¼]â”‚        â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Mode[btnâ–¼]â”‚Mode[btnâ–¼]â”‚          â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Lo  [0  ] â”‚ Lo  [0  ] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Hi  [127] â”‚ Hi  [127] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

#### Green Buttons Section
```
GREEN BUTTONS (Group 1)
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Btn 1    â”‚ Btn 2    â”‚ Btn 3    â”‚ Btn 4    â”‚ Btn 5    â”‚ Btn 6    â”‚ Btn 7    â”‚ Btn 8    â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚          â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Note[64] â”‚ Note[65] â”‚          â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Type[Noteâ–¼]â”‚Type[Noteâ–¼]â”‚        â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Mode[btnâ–¼]â”‚Mode[btnâ–¼]â”‚          â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ LED [Stdâ–¼]â”‚LED [Stdâ–¼]â”‚          â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Lo  [0  ] â”‚ Lo  [0  ] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â”‚ Hi  [127] â”‚ Hi  [127] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

#### Faders Section
```
FADERS (Group 1)
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Fad 1    â”‚ Fad 2    â”‚ Fad 3    â”‚ Fad 4    â”‚ Fad 5    â”‚ Fad 6    â”‚ Fad 7    â”‚ Fad 8    â”‚ Fad 9    â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚ Ch  [1â–¼] â”‚
â”‚ CC  [32] â”‚ CC  [33] â”‚ CC  [34] â”‚ CC  [35] â”‚ CC  [36] â”‚ CC  [37] â”‚ CC  [38] â”‚ CC  [39] â”‚ CC  [112]â”‚
â”‚ Type[CCAbâ–¼]â”‚Type[CCAbâ–¼]â”‚        â”‚          â”‚          â”‚          â”‚          â”‚          â”‚ -------- â”‚
â”‚ Mode[JMPâ–¼]â”‚Mode[JMPâ–¼]â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚ Mode[JMPâ–¼]â”‚
â”‚ Disp[Stdâ–¼]â”‚Disp[Stdâ–¼]â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚ -------- â”‚
â”‚ Min [0  ] â”‚ Min [0  ] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚ Min [0  ] â”‚
â”‚ Max [127] â”‚ Max [127] â”‚         â”‚          â”‚          â”‚          â”‚          â”‚          â”‚ Max [127] â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

Note: Fader 9 shows "--------" for Type and Display as it doesn't support these parameters.

### Overview View (All Groups at Once)

Compact view showing all 8 groups for quick comparison and conflict detection.

```
ENCODERS OVERVIEW
        â”‚ Group 1 â”‚ Group 2 â”‚ Group 3 â”‚ Group 4 â”‚ Group 5 â”‚ Group 6 â”‚ Group 7 â”‚ Group 8 â”‚
â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
Enc 1   â”‚ Ch1 CC8 â”‚ Ch1 CC16â”‚ Ch1 CC24â”‚ Ch1 CC32â”‚ Ch1 CC72â”‚ Ch1 CC80â”‚ Ch1 CC88â”‚ Ch1 CC96â”‚
Enc 2   â”‚ Ch1 CC9 â”‚ Ch1 CC17â”‚ Ch1 CC25â”‚ Ch1 CC33â”‚ Ch1 CC73â”‚ Ch1 CC81â”‚ Ch1 CC89â”‚ Ch1 CC97â”‚
...     â”‚         â”‚         â”‚         â”‚         â”‚         â”‚         â”‚         â”‚         â”‚
```

- Click any cell to jump to Focused View for that group
- Highlight conflicts (same CC on same channel)

---

## File Operations

### Import SysEx (.syx)
1. File picker accepts .syx files
2. Validate file size = 100,640 bytes
3. Validate SysEx framing (F0 start, F7 end)
4. Parse into internal data model
5. Default to Setup 1, Encoder Group 1, Fader Group 1

### Export SysEx (.syx)
1. Recalculate checksum for **each bank block** (every section+bank pair has its own checksum)
2. Generate complete 100,640 byte dump
3. Download as `uc4_edited.syx` or user-specified name

**Checksum calculation:**
```javascript
// Sum all decoded triplet values in the bank
let sum = 0;
let pos = dataOffset;
while (buf[pos] === 0x4D) {
  sum += ((buf[pos+1] & 0x0F) << 4) | (buf[pos+2] & 0x0F);
  pos += 3;
}
sum = sum & 0xFFFF;  // 16-bit wrap (explicit for portability)

// Write to 4B/4C markers at checksumOffset
buf[checksumOffset + 1] = 0x20 | ((sum >> 12) & 0x0F);
buf[checksumOffset + 2] = 0x10 | ((sum >> 8) & 0x0F);
buf[checksumOffset + 4] = 0x20 | ((sum >> 4) & 0x0F);
buf[checksumOffset + 5] = 0x10 | (sum & 0x0F);
```

### Data Model Safety (Critical)
- **Raw buffer is source of truth**: Keep original dump in memory
- **Unknown bytes preserved verbatim**: Only write to mapped field locations
- **Never regenerate from scratch**: Layer decoded "views" on top of raw buffer
- This ensures unmapped parameters (acceleration, min/max, button modes, etc.) survive round-trip editing

### Import JSON (optional)
- Load previously saved configuration
- Human-readable format for version control

### Export JSON (optional)
- Save current configuration as JSON
- Structure mirrors internal data model

---

## Validation

### Duplicate Detection
- Key on **(channel, command-type, number)** - treat as separate namespaces:
  - CC messages
  - Note messages
  - Program Change (channel-wide, number is program)
  - Pitch Bend (channel-wide, no number)
  - Aftertouch (channel-wide, no number)
- **High-res CC (CCAh)**: treat as occupying **both CC N and CC N+32** for collision warnings
- Warn when collision detected within a group
- Visual indicator: Yellow highlight or warning icon
- Non-blocking: User can proceed (some setups intentionally use duplicates)

### Range Enforcement
| Parameter | Valid Range | Internal Storage |
|-----------|-------------|------------------|
| MIDI Channel | 1-16 | 0-15 |
| CC/Note Number | 0-127 | 0-127 |
| Min/Max Values | 0-127 | 0-127 |

### 14-bit High-Resolution Encoder Constraints
When encoder Type = CCAh:
- CC Number must be **0-31** (MSB range)
- LSB is automatically assigned to CC Number + 32
- UI should:
  - Enforce 0-31 range for CC field
  - Display read-only "LSB: [N+32]" to show the paired CC

### Inverted Ranges
- Min > Max is valid (inverts control direction)
- Display indicator that range is inverted

---

## Implementation Phases

### Phase 1 (v1) - Core Functionality
All parameters now mapped from Phase 2 analysis.

**Complete Section/Bank Reference (per setup):**

| Control | Data | Section | Bank | Values | Encoding |
|---------|------|---------|------|--------|----------|
| **Encoders** | Channel+Type | 0x1C | 0x00 | 64 | (Type<<4)\|Chan |
| | CC Number | 0x1C | 0x40 | 64 | Direct 0-127 |
| | Min | 0x1C | 0x80 | 64 | Direct 0-127 |
| | Max | 0x1C | 0xC0 | 64 | Direct 0-127 |
| | Acc+Display | 0x1D | 0x00 | 64 | (Acc<<4)\|Disp |
| **Push Buttons** | Channel+Type | 0x1D | 0x40 | 64 | TypeNibble\|Chan |
| | CC/Note | 0x1D | 0x80 | 64 | Direct 0-127 |
| | Lower Value | 0x1D | 0xC0 | 64 | Direct 0-127 |
| | Upper Value | 0x1E | 0x00 | 64 | Direct 0-127 |
| | Mode+Display | 0x1E | 0x40 | 64 | (Mode<<4)\|Disp |
| **Green Buttons** | Channel+Type | 0x1E | 0x80 | 64 | TypeNibble\|Chan |
| | CC/Note | 0x1E | 0xC0 | 64 | Direct 0-127 |
| | Lower Value | 0x1F | 0x00 | 64 | Direct 0-127 |
| | Upper Value | 0x1F | 0x40 | 64 | Direct 0-127 |
| | Mode+Display | 0x1F | 0x80 | 64 | (Mode<<4)\|Disp |
| **Faders 1-8** | Channel+Type | 0x1F | 0xC0 | 64 | (Type<<4)\|Chan |
| | CC Number | 0x20 | 0x00 | 64 | Direct 0-127 |
| | Min | 0x20 | 0x40 | 64 | Direct 0-127 |
| | Max | 0x20 | 0x80 | 64 | Direct 0-127 |
| | Mode+Display | 0x20 | 0xC0 | 64 | (Mode<<4)\|Disp |
| **Fader 9** | All params | 0x17 | 0x00 | 40 | 5 values Ã— 8 groups |
| **Group Names** | Characters | 0x14 | 0x80 | 32 | 4 chars Ã— 8 groups |

**Encoders (fully mapped):**
- âœ“ Channel (lower nibble of Channel+Type byte)
- âœ“ Type (upper nibble of Channel+Type byte) - values 0-6
- âœ“ CC Number (separate bank, direct 0-127)
- âœ“ Min Value (separate bank, direct 0-127)
- âœ“ Max Value (separate bank, direct 0-127)
- âœ“ Acceleration (upper nibble of Acc+Display byte) - values 0-3
- âœ“ Display Mode (lower nibble of Acc+Display byte) - 0=OFF, 1=Std, 2=bPoL

**Encoder Acc+Display byte encoding:**
```javascript
// Section 0x1D Bank 0x00
raw = (acceleration << 4) | displayMode;
// acceleration: 0=Acc0, 1=Acc1, 2=Acc2, 3=Acc3
// displayMode: 0=OFF, 1=Std, 2=bPoL
```

**Push Buttons (fully mapped):**
- âœ“ Channel (lower nibble of Channel+Type byte)
- âœ“ Type (upper nibble as pre-shifted nibble: 0x10=Note, 0x20=CC)
- âœ“ CC/Note Number (separate bank, direct 0-127)
- âœ“ Lower Value (separate bank, direct 0-127)
- âœ“ Upper Value (separate bank, direct 0-127)
- âœ“ Mode (upper nibble of Mode+Display byte) - bit 4: 0=Momentary, 1=Toggle
- âœ“ Display Mode (lower nibble of Mode+Display byte)

**Push Button Mode+Display byte encoding:**
```javascript
// Section 0x1E Bank 0x40
// Mode: 0x00 = Momentary, 0x10 = Toggle (bit 4)
raw = modeNibble | displayMode;
```

**Green Buttons (fully mapped):**
- âœ“ Channel (lower nibble of Channel+Type byte)
- âœ“ Type (upper nibble as pre-shifted nibble: 0x10=Note, 0x20=CC)
- âœ“ CC/Note Number (separate bank, direct 0-127)
- âœ“ Lower Value (separate bank, direct 0-127)
- âœ“ Upper Value (separate bank, direct 0-127)
- âœ“ Mode (upper nibble of Mode+Display byte) - bit 4: 0=Momentary, 1=Toggle
- âœ“ Display Mode (lower nibble of Mode+Display byte) - 0=OFF, 1=Std, 2=EXt

**Green Button Mode+Display byte encoding:**
```javascript
// Section 0x1F Bank 0x80
// Mode: 0x00 = Momentary, 0x10 = Toggle (bit 4)
raw = modeNibble | displayMode;
```

**Faders 1-8 (fully mapped):**
- âœ“ Channel (lower nibble of Channel+Type byte)
- âœ“ Type (upper nibble of Channel+Type byte) - 0=CCAb, 1=PrGC, 2=Pbnd, 3=AFtt
- âœ“ CC Number (separate bank, direct 0-127)
- âœ“ Min Value (separate bank, direct 0-127)
- âœ“ Max Value (separate bank, direct 0-127)
- âœ“ Mode (upper nibble of Mode+Display byte) - bit 4: 0=Jump, 1=Snap
- âœ“ Display Mode (lower nibble of Mode+Display byte) - 0=OFF, 1=Std, 2=bPoL

**CORRECTION from Phase 1:** Section 0x1F Bank 0xC0 contains Type+Channel, not Channel alone.

**Fader Channel+Type byte encoding:**
```javascript
// Section 0x1F Bank 0xC0
raw = (type << 4) | channel;
// type: 0=CCAb, 1=PrGC, 2=Pbnd, 3=AFtt
```

**Fader Mode+Display byte encoding:**
```javascript
// Section 0x20 Bank 0xC0
// Mode: 0x00 = Jump, 0x10 = Snap (bit 4)
raw = modeNibble | displayMode;
```

**Fader 9 (fully mapped):**
- âœ“ Channel (offset [0], direct 0-15)
- âœ“ CC Number (offset [1], direct 0-127)
- âœ“ Min Value (offset [2], direct 0-127)
- âœ“ Max Value (offset [3], direct 0-127)
- âœ“ Mode (offset [4], confirmed: 1=Jump, 17=Snap â†’ bit 4 is Snap flag)
- Layout: 40 values total = 5 values Ã— 8 groups
- Group N data starts at: dataOffset + (N Ã— 5 Ã— 3) bytes (accounting for triplet encoding)

**Fader 9 Mode encoding:**
```javascript
// Section 0x17 Bank 0x00, offset [4] per group
// Mode: 1 = Jump, 17 (0x11) = Snap
// Bit 4 (0x10) = Snap flag
mode = (raw & 0x10) ? 'Snap' : 'Jump';
```

**Group Names (mapped):**
- Section 0x14 Bank 0x80
- 32 values = 4 characters Ã— 8 groups
- Character encoding: 7-segment display codes (requires character map)

**Button Type Handling:**
- Note (0x10) and CC (0x20) confirmed
- OFF/PrGC/AFtt nibble values still unverified - preserve if encountered

**Critical: Encoder vs Button vs Fader Type Encoding**

*Encoders* store type as a 0-6 value shifted into high nibble:
```javascript
// Encoder channel byte (Section 0x1C Bank 0x00)
raw = (type << 4) | channel;  // type is 0..6
// type: 0=CCr1, 1=CCr2, 2=CCAb, 3=PrGC, 4=CCAh, 5=Pbnd, 6=AFtt
```

*Faders 1-8* store type as a 0-3 value shifted into high nibble:
```javascript
// Fader channel byte (Section 0x1F Bank 0xC0)
raw = (type << 4) | channel;  // type is 0..3
// type: 0=CCAb, 1=PrGC, 2=Pbnd, 3=AFtt
```

*Buttons* store type as a pre-shifted high nibble (0x10, 0x20, etc.) â€” **never shift button types**:
```javascript
// Button channel byte â€” NO shifting of type values
raw = typeNibble | channel;   // typeNibble is ALREADY 0x10, 0x20, etc.

// Reading:
typeNibble = raw & 0xF0;      // yields 0x10, 0x20, etc. (NOT 1, 2)
channel = raw & 0x0F;

// Safe edits:
// Change channel only, preserve type:
newRaw = (oldRaw & 0xF0) | newChannel;
// Change type only, preserve channel:
newRaw = newTypeNibble | (oldRaw & 0x0F);  // newTypeNibble must be 0x10, 0x20, etc.
```

**This is the #1 place abstraction bugs occur.** If you normalise button types to 0,1,2... you'll double-shift and corrupt the data. The type nibble is stored as-is.

### Phase 2 - Remaining Items
Most parameters now mapped. Remaining:
- Button type nibbles for OFF/PrGC/AFtt (only Note=0x10, CC=0x20 tested)
- Group name character encoding (7-segment codes need mapping)
- Verify display mode values across all control types

### Phase 3 - Advanced Features
- Overview view with conflict highlighting
- Copy/paste between controls, groups, setups
- Undo/redo
- MIDI learn integration (if WebMIDI available)
- Live preview via SysEx send

---

## Technical Requirements

### Platform
- Single HTML file, no build process required
- Vanilla HTML/CSS/JS (no external frameworks)
- Works offline after initial page load

### Browser Support
- Modern browsers (Chrome, Firefox, Safari, Edge)
- File API for import/export
- Optional: WebMIDI API for direct device communication

### UI/UX
- Dark theme (matches Faderfox aesthetic)
- Responsive layout (desktop primary, tablet acceptable)
- Keyboard navigation for efficient editing
- Visual feedback on parameter changes

### Internal Architecture

#### Import-Time Index Building
On SysEx import, scan the entire dump and build a per-setup index:

```javascript
// Index is per-setup. Each (sectionId, bank) appears 18 times in the full dump.
// Structure:
index[setupIdx][sectionId][bank] = {
  dataOffset,       // byte position where 4D triplets start
  valueCount,       // derived by counting 0x4D triplets (not assumed)
  checksumOffset    // where 0x4B..0x4C markers begin
}

// setupIdx: 0..17
// sectionId: e.g. 0x1C (encoders), 0x1D (push), 0x1E (green), etc.
// bank: e.g. 0x00, 0x40, 0x80, 0xC0
```

**Recommended scanning approach:**
```javascript
// 1. During scan, collect occurrences in encounter order:
const occ = {};  // occ[sectionId][bank] = [ {dataOffset, valueCount, checksumOffset}, ... ]

// 2. After scan, validate and assign to setup indices:
for (const secId of REQUIRED_SECTIONS) {
  for (const bank of REQUIRED_BANKS[secId]) {
    if (occ[secId][bank].length !== 18) {
      throw new Error(`Expected 18 occurrences of section ${secId} bank ${bank}`);
    }
    for (let setupIdx = 0; setupIdx < 18; setupIdx++) {
      index[setupIdx][secId][bank] = occ[secId][bank][setupIdx];
    }
  }
}
```

This approach:
- Doesn't hardcode setup byte boundaries
- Doesn't care if unknown sections exist between known ones
- Validates expected structure on import

**Deriving valueCount (don't hardcode):**
```javascript
// Starting at dataOffset, count triplets:
let pos = dataOffset;
while (buf[pos] === 0x4D) {
  pos += 3;
}
valueCount = (pos - dataOffset) / 3;
// checksumOffset is where buf[pos] === 0x4B && buf[pos+3] === 0x4C
```

This approach:
- Handles all 18 setups correctly
- Stays robust to variable-length banks (e.g., group names)
- Doesn't assume fixed value counts

#### Buffer Architecture
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Raw SysEx Buffer (100,640 bytes)        â”‚ â† Source of truth
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Section Index (built at import)         â”‚ â† Lookup table
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Decoded Views (UI-friendly values)      â”‚ â† Read via decode()
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ UI State (selected setup/group)         â”‚ â† Navigation
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Write path: UI change â†’ encode() â†’ write to raw buffer at indexed offset â†’ mark checksum dirty
Export path: Recalculate dirty checksums â†’ output raw buffer
```

---

## Reference Documents

| Document | Purpose |
|----------|---------|
| UC4_SYSEX_PROTOCOL.md | Byte-level encoding, section maps, checksums |
| UC4_Manual_V03.pdf | Parameter names, ranges, factory defaults |
| factory_default.syx | Reference dump for testing |

---

## Appendix A: 7-Segment Character Set

Characters displayable on UC4's 4-digit display:
```
0-9: Standard numerals
A, b, C, c, d, E, F, G, H, h, I, i, J, L, n, O, o, P, r, S, t, U, u, Y
- (dash), _ (underscore), space
```

Group names should use these characters for accurate hardware display.

---

## Appendix B: MIDI Implementation Notes

### Relative Encoder Modes
| Mode | Values Sent | Notes |
|------|-------------|-------|
| CCr1 | 1 / 127 | Direction interpretation varies by host software |
| CCr2 | 63 / 65 | Direction interpretation varies by host software |

Common host interpretations:
- Ableton Live: CCr1 (1=inc, 127=dec) or CCr2
- Bitwig: CCr1
- Verify on your specific host

### 14-bit High Resolution
- MSB sent on CC number N (0-31)
- LSB automatically sent on CC number N+32
- Provides 16384 steps for sensitive parameters

### Pitch Bend
- Full 14-bit resolution (0-16383)
- Center value: 8192
- Per-channel, no CC number needed

---

## Appendix C: Ableton Live Setup Reference

### Setup 17 (Tracks 1-8) / Setup 18 (Tracks 9-16)

#### Encoder Groups
| Group | Name | Function |
|-------|------|----------|
| 1 | Snd1 | Send A for 8 tracks |
| 2 | Snd2 | Send B for 8 tracks |
| 3 | Snd3 | Send C for 8 tracks |
| 4 | Snd4 | Send D for 8 tracks |
| 5 | trAC | Selected track controls |
| 6 | RAC | Rack macro 1-8 |
| 7 | PAn | Track panorama |
| 8 | GLOb | Global controls (tempo, transport, master) |

#### Fader/Button Groups
- Faders: Track volumes (same all groups)
- Buttons vary: Launch, Stop, Active, Solo, Arm, Monitor, Track control, Select
