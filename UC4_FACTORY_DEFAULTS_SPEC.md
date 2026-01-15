# UC4 Factory Defaults Specification

**Version:** 1.0  
**Status:** LOCKED  
**Date:** 2026-01-15

---

## Overview

This specification defines the factory defaults architecture for the UC4 Editor's Setup Manager. It replaces the previous `FACTORY_DEFAULTS` object with per-setup factory templates extracted from `factory_default.syx`.

---

## Problem Statement

The original `FACTORY_DEFAULTS` implementation assumed:
- All setups use Channel 1
- All groups use the same CC/Note patterns
- A single parametric function could generate factory values

**Validation against `factory_default.syx` revealed:**
- Each setup (1-16) uses a unique MIDI channel matching its setup number
- Setups 17-18 are Ableton presets with different CC layouts and channels 13-14
- CC/Note assignments are group-specific, not uniform

---

## Validated Factory Patterns

### Channel Assignment

| Setups | MIDI Channel |
|--------|--------------|
| 1-16 | Channel = Setup Number (1-16) |
| 17 | Channel 13 (Ableton) |
| 18 | Channel 14 (Ableton) |

### CC/Note Patterns

#### Generic Template (Setups 1-16)

**Encoders:**
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

**Push Buttons:**
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

**Green Buttons:**
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

**Faders 1-8:**
| Group | CC Numbers |
|-------|------------|
| 1 | 32-39 |
| 2 | 40-47 |
| 3 | 48-55 |
| 4 | 56-63 |
| 5-8 | 104-111 (shared) |

**Fader 9:**
- CC 112 for all groups

**Display Settings:**
- Encoders: display = 1 (Std)
- Push Buttons: display = 0 (OFF)
- Green Buttons: display = 1 (Std)
- Faders: display = 1 (Std)

#### Ableton Template (Setups 17-18)

**Encoders:**
| Group | CC Numbers |
|-------|------------|
| 1 | 0-7 |
| 2 | 8-15 |
| 3 | 16-23 |
| 4 | 24-31 |
| 5 | 56-63 |
| 6 | 48-55 |
| 7 | 32-39 |
| 8 | 56-63 |

**Fader 9:**
- CC 48 for all groups

*(Other controls follow different patterns - extract from SysEx)*

---

## Architecture Decision

### Per-Setup Factory Templates (Option 18A)

**Decision:** Store 18 separate factory templates extracted from `factory_default.syx`.

**Rationale:**
1. Each setup has unique channel assignment
2. Setups 17-18 have completely different CC layouts
3. No parametric formula can capture all variations
4. Exact byte-for-byte restoration on Clear

**Rejected Alternative:** Parametric functions with exceptions tables would be fragile and incomplete.

---

## Data Structure

```javascript
// Global factory templates - populated at app init
let FACTORY_TEMPLATES = null; // Array(18) of setup snapshots

// Each template is a JSON-domain snapshot:
// {
//   groups: [
//     {
//       name: "GrP1",
//       encoders: [{ channel: N, type: 2, cc: X, min: 0, max: 127, acc: 3, display: 1 }, ...],
//       pushButtons: [{ channel: N, typeNibble: 16, cc: X, lower: 0, upper: 127, mode: 0, display: 0 }, ...],
//       greenButtons: [{ channel: N, typeNibble: 16, cc: X, lower: 0, upper: 127, mode: 0, display: 1 }, ...],
//       faders: [{ channel: N, type: 0, cc: X, min: 0, max: 127, mode: 0, display: 1 }, ...],
//       fader9: { channel: N, cc: 112, min: 0, max: 127, mode: 0 }
//     },
//     // ... 8 groups total
//   ]
// }
```

### JSON-Domain Conventions

| Field | Range | Notes |
|-------|-------|-------|
| channel | 1-16 | Display values, not internal 0-15 |
| cc/note | 0-127 | Direct values |
| min/max/lower/upper | 0-127 | Direct values |
| type (encoder) | 0-6 | Internal encoding |
| type (fader) | 0-3 | Internal encoding |
| typeNibble (button) | 0x00/0x10/0x20/0x30/0x40 | Pre-shifted nibble |
| mode | 0-1 | 0=Momentary/Jump, 1=Toggle/Snap |
| display | 0-2 | 0=OFF, 1=Std, 2=bPoL/EXt |
| acc | 0-3 | Encoder acceleration |
| name | string | Group name (4 chars max) |

---

## Implementation

### Initialization

```javascript
async function initFactoryTemplates() {
    // Load factory_default.syx (already loaded as rawBuffer on app init)
    // Or fetch separately if needed
    
    FACTORY_TEMPLATES = [];
    for (let setupIdx = 0; setupIdx < 18; setupIdx++) {
        FACTORY_TEMPLATES[setupIdx] = extractFactorySnapshot(setupIdx);
    }
    
    // Freeze to prevent accidental mutation
    Object.freeze(FACTORY_TEMPLATES);
}

function extractFactorySnapshot(setupIdx) {
    // Use existing captureSetupSnapshot logic against factory buffer
    // Returns JSON-domain snapshot
    return {
        groups: Array.from({ length: 8 }, (_, g) => ({
            name: getGroupNameFromFactory(setupIdx, g),
            encoders: Array.from({ length: 8 }, (_, i) => getEncoderDataFromFactory(setupIdx, g, i)),
            pushButtons: Array.from({ length: 8 }, (_, i) => getPushDataFromFactory(setupIdx, g, i)),
            greenButtons: Array.from({ length: 8 }, (_, i) => getGreenDataFromFactory(setupIdx, g, i)),
            faders: Array.from({ length: 8 }, (_, i) => getFaderDataFromFactory(setupIdx, g, i)),
            fader9: getFader9DataFromFactory(setupIdx, g)
        }))
    };
}
```

### createFactorySetupSnapshot(setupIdx)

```javascript
function createFactorySetupSnapshot(setupIdx) {
    if (!FACTORY_TEMPLATES) {
        throw new Error('Factory templates not initialized');
    }
    return JSON.parse(JSON.stringify(FACTORY_TEMPLATES[setupIdx]));
}
```

### differsFromFactory(setupIdx)

```javascript
function differsFromFactory(setupIdx) {
    if (differsFromFactoryCache.has(setupIdx)) {
        return differsFromFactoryCache.get(setupIdx);
    }
    
    const current = captureSetupSnapshot(setupIdx);
    const factory = FACTORY_TEMPLATES[setupIdx];
    
    // CRITICAL: Full comparison including channel
    // Do NOT normalize or ignore any fields
    const differs = !snapshotsEqual(current, factory);
    
    differsFromFactoryCache.set(setupIdx, differs);
    return differs;
}

function snapshotsEqual(a, b) {
    return JSON.stringify(a) === JSON.stringify(b);
}
```

### Clear Operation

```javascript
function performClear() {
    // ... existing dialog handling ...
    
    const factorySnapshot = createFactorySetupSnapshot(idx);
    restoreSetupSnapshot(idx, factorySnapshot);
    
    // ... undo recording, UI updates ...
}
```

---

## Code Changes Required

### Remove

```javascript
// DELETE this entire block (lines ~7110-7160 in index.html)
const FACTORY_DEFAULTS = {
    encoder: (idx) => ({
        channel: 1,
        type: 2,
        cc: idx + 1,  // WRONG
        // ...
    }),
    // ...
};
```

### Add

```javascript
// Add near top of script, after state declarations
let FACTORY_TEMPLATES = null;

// Add initialization call in loadDefaultSysEx() or init
await initFactoryTemplates();
```

### Modify

```javascript
// createFactorySetupSnapshot - replace entire function
function createFactorySetupSnapshot(setupIdx = 0) {
    return JSON.parse(JSON.stringify(FACTORY_TEMPLATES[setupIdx]));
}

// differsFromFactory - update to use FACTORY_TEMPLATES
function differsFromFactory(setupIdx) {
    if (differsFromFactoryCache.has(setupIdx)) {
        return differsFromFactoryCache.get(setupIdx);
    }
    
    const current = captureSetupSnapshot(setupIdx);
    const factory = FACTORY_TEMPLATES[setupIdx];
    const differs = !snapshotsEqual(current, factory);
    
    differsFromFactoryCache.set(setupIdx, differs);
    return differs;
}
```

---

## Test Cases

### Existing Tests (Update Expected Values)

| ID | Test | Notes |
|----|------|-------|
| CL1 | Clear single setup | Now restores slot-specific template |
| CL2 | Clear setup with label | Label removed, slot-specific data restored |
| CL3 | Clear multiple setups | Each restored to its own template |
| CL4 | Undo clear | Restores previous state |
| CL5 | Clear setup 17 | Restores Ableton preset |
| CL6 | Redo clear | Re-applies clear |

### New Tests

| ID | Test | Expected |
|----|------|----------|
| **DF-CHAN** | Load factory_default.syx, check differsFromFactory(0..17) | All return false |
| **DF-EDIT** | Edit any control, check differsFromFactory | Returns true |
| **DF-UNDO** | Edit then undo, check differsFromFactory | Returns false |
| **CL-SETUPCHAN** | Clear Setup 5, verify all controls | channel = 5 |
| **CL-ABLETON17** | Clear Setup 17, verify encoder[0].cc | cc = 0 (not 8) |
| **CL-ABLETON18** | Clear Setup 18, verify channel | channel = 14 |
| **CL7** | Clear Setup 17 | Restores Ableton preset with ch=13 |
| **CL8** | Clear Setup 18 | Restores Ableton preset with ch=14 |

---

## Edge Cases

### Push Button Display = 0 (OFF)

Factory defaults have push button display = OFF. This is correct hardware behavior. After Clear, push buttons will show no value on the UC4 display. This is intentional.

### Faders G5-G8 Share CC Range

Groups 5-8 all use CC 104-111 for faders. This is not a bug - it's the actual factory configuration. The per-setup template approach handles this automatically.

### Setup 17-18 Channel Mismatch

Setups 17-18 use channels 13-14, not 17-18. This is intentional (Ableton preset configuration). The per-setup template preserves this exactly.

---

## Validation Checklist

Before marking implementation complete:

- [ ] `FACTORY_TEMPLATES` populated for all 18 setups
- [ ] `differsFromFactory(0..17)` all return false after loading factory_default.syx
- [ ] Clear Setup 5 results in all controls having channel = 5
- [ ] Clear Setup 17 results in encoder[0].cc = 0, channel = 13
- [ ] Clear Setup 18 results in channel = 14
- [ ] Undo after Clear restores previous state exactly
- [ ] Modified indicator (â—) disappears after Clear (for unedited setups)

---

## Summary

| Aspect | Before | After |
|--------|--------|-------|
| Factory data | Parametric functions | 18 extracted snapshots |
| Channel handling | Hardcoded to 1 | Per-setup (1-16, 13, 14) |
| CC patterns | Single formula | Group-specific from SysEx |
| Clear operation | Generic template | Slot-specific restore |
| differsFromFactory | Compared against wrong baseline | Exact per-slot comparison |

**Key insight:** Factory defaults are not a formula - they are 18 distinct configurations that must be preserved exactly.
