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

Checks structure, ranges, and mapping against your spec.
