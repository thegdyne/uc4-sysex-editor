# UC4 JSON Quick Start

## Files

| File | What it is |
|------|------------|
| `UC4_JSON_SCHEMA.md` | All valid parameters and their meanings |
| `uc4_config_template.json` | Clean working example to start from |

---

## Creating a Config with Claude

Just ask naturally:

> "Using the UC4 JSON schema and template, create a config where..."

### Examples

**Simple:**
> "Make all encoders CC 1-8 on channel 1"

**Channel-per-group:**
> "Set up groups 1-8 on channels 1-8, faders control volume (CC 7)"

**Mixed types:**
> "Encoders as pitch bend, green buttons as CC toggles for mute"

**Specific setup:**
> "In Setup 3, make push buttons send program changes 0-7"

**With custom names:**
> "Name the groups: Osc1, Osc2, FILt, EnU1, EnU2, LFO1, LFO2, Out"

---

## Group Names

The UC4's 7-segment display limits which characters can be shown.

### Valid Characters (32 total)

```
0 1 2 3 4 5 6 7 8 9
A b C d E F G H I J L n O t P S r U Y
- _ (space)
```

### Naming Rules

| Rule | Example |
|------|---------|
| Exactly 4 characters | `"FILt"` ✓ |
| Short names get space-padded | `"LFO"` → `"LFO "` |
| Some letters are lowercase only | B→b, D→d, N→n, T→t, R→r |
| Invalid chars are rejected | `"DRUM"` ✗ (M not available) |

### Quick Reference

**Always lowercase:** b, d, n, t, r  
**Not available:** K, M, Q, V, W, X, Z

### Good Name Examples

| Purpose | Name | Notes |
|---------|------|-------|
| Oscillator 1 | `"OSC1"` | All valid |
| Filter | `"FILt"` | t is lowercase |
| Envelope | `"EnU1"` | n lowercase |
| LFO | `"LFO "` | Padded |
| Bass | `"bASS"` | B→b |
| Drums | `"drU1"` | Can't spell "DRUM" |
| Synth | `"SYnt"` | n,t lowercase |
| Send 1 | `"Snd1"` | n,d lowercase |
| Global | `"GLOb"` | b lowercase |

---

## Quick Reference

### Control Types

**Encoders** (type 0-6):
- `2` = CCAb (standard CC) ← most common
- `5` = Pitch bend
- `0`/`1` = Relative modes

**Faders** (type 0-3):
- `0` = CCAb (standard CC) ← most common
- `2` = Pitch bend

**Buttons** (typeNibble):
- `16` = Note ← factory default
- `32` = CC ← use for mute/solo etc.

### Modes

**Buttons:**
- `0` = Momentary (send on press, release)
- `1` = Toggle (alternate each press)

**Faders:**
- `0` = Jump (immediate)
- `1` = Snap (pickup)

---

## Workflow

1. Ask Claude to generate JSON
2. Download the `.json` file
3. Import into UC4 Editor
4. Export as `.syx`
5. Send to UC4 via MIDI

---

## Validation

Ask Claude to validate:
> "Can you validate this UC4 JSON?" (attach file)

Checks structure, ranges, group name charset, and mapping against your spec.

---

## Common Naming Schemes

### By Function
```json
"name": "OSC1"  // Oscillator 1
"name": "OSC2"  // Oscillator 2  
"name": "FILt"  // Filter
"name": "EnU1"  // Envelope 1
"name": "LFO1"  // LFO 1
"name": "Out "  // Output
```

### By Channel
```json
"name": "Ch01"
"name": "Ch02"
// ...
"name": "Ch08"
```

### By Instrument
```json
"name": "bASS"  // Bass
"name": "LEAd"  // Lead
"name": "PAd "  // Pad
"name": "drU1"  // Drums (can't spell DRUM)
```

### Generic
```json
"name": "GrP1"  // Factory default style
"name": "Grp1"  // Alternative
"name": "Set1"  // Set-based
"name": "A   "  // Minimal
```
