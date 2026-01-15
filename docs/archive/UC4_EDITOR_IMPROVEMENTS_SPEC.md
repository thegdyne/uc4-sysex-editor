# UC4 Editor Improvements Specification

## Overview

This document specifies four interconnected features to enhance the UC4 SysEx Editor:

---

## Current Implementation Context

### Auto-Load Factory Defaults

The editor currently auto-loads `factory_default.syx` from the repository on startup:

```javascript
async function loadDefaultSysEx() {
    try {
        updateStatus('', 'Loading factory_default.syx...');
        const response = await fetch('factory_default.syx');
        if (!response.ok) {
            console.log('No factory_default.syx found, waiting for manual import');
            updateStatus('', 'No data loaded');
            return;
        }
        // ... validates and initializes rawBuffer, builds sectionIndex ...
        
        // Reset baseline state (critical for consistent "dirty" semantics)
        State.isModified = false;
        State.dirtyBanks.clear();
    }
}
```

**Benefits:**
- Users get a working editor immediately without dumping from hardware
- No embedded base64 blob - file lives in repo alongside the editor
- Graceful fallback to manual import if file missing

**Current data flow:**
1. Page loads â†’ auto-fetches `factory_default.syx` â†’ ready to edit
2. User makes changes (in-memory only)
3. User exports `.syx` to save their work
4. Page refresh â†’ **back to factory defaults** (edits lost if not exported)

This context is critical for the Session Persistence feature specified below.

---

## Features Overview

1. **Overview Mode** - Visual comparison of all 8 groups simultaneously
2. **Duplicate/Conflict Detection** - Identify MIDI assignment collisions
3. **Copy/Paste Operations** - Efficient bulk editing
4. **Undo/Redo System** - Safe experimentation with changes
5. **Session Persistence** - Protect unsaved work across page refreshes

These features work together: Overview mode surfaces conflicts visually, conflict detection provides the underlying logic, copy/paste enables rapid fixes, undo/redo allows safe experimentation, and session persistence prevents accidental data loss.

### Indexing Convention

**Internal indices are 0-based; UI labels are 1-based.**

| Concept | Internal Range | UI Display |
|---------|----------------|------------|
| Setup | setupIdx: 0â€“17 | Setup 1â€“18 |
| Group | group: 0â€“7 | Group 1â€“8 |
| Control index | index: 0â€“7 | Encoder 1â€“8, Fader 1â€“8, etc. |
| MIDI Channel | 0â€“15 (stored in buffer) | Ch 1â€“16 (displayed, conflict keys) |

All code examples use 0-based indices unless otherwise noted.

**Channel conversion helpers** (defined in Overview Mode section):
- `chDisp(ch0)` â€” Convert stored 0-15 to display 1-16
- `chKey(ch0)` â€” Convert stored 0-15 to conflict key 1-16
- `applyChannelOffset(ch0, delta, wrapMode)` â€” Apply offset in 0-15 range

---

## 1. Overview Mode

### Purpose
Display all 8 groups simultaneously in a compact format for:
- Quick visual comparison across groups
- Spotting conflicts and duplicates at a glance
- Understanding the complete setup configuration
- Navigating to specific controls for detailed editing

### Design Approach

#### Tab-Based Control Type Views
Rather than cramming everything onto one screen, provide tabs for each control type:

```
[Encoders] [Push Buttons] [Green Buttons] [Faders]
```

Each tab shows all 8 groups for that control type in a dense table format.

#### Encoder Overview Layout

```
ENCODERS - Setup 1                                    [Focused View]
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
        â”‚ Group 1  â”‚ Group 2  â”‚ Group 3  â”‚ Group 4  â”‚ Group 5  â”‚ Group 6  â”‚ Group 7  â”‚ Group 8  â”‚
        â”‚  GrP1    â”‚  GrP2    â”‚  GrP3    â”‚  GrP4    â”‚  GrP5    â”‚  GrP6    â”‚  GrP7    â”‚  GrP8    â”‚
â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
Enc 1   â”‚ 1:CC 8   â”‚ 1:CC 16  â”‚ 1:CC 24  â”‚ 1:CC 32  â”‚ 1:CC 72  â”‚ 1:CC 80  â”‚ 1:CC 88  â”‚ 1:CC 96  â”‚
Enc 2   â”‚ 1:CC 9   â”‚ 1:CC 17  â”‚ 1:CC 25  â”‚ 1:CC 33  â”‚ 1:CC 73  â”‚ 1:CC 81  â”‚ 1:CC 89  â”‚ 1:CC 97  â”‚
Enc 3   â”‚ 1:CC 10  â”‚ 1:CC 18  â”‚ 1:CC 26  â”‚ 1:CC 34  â”‚ 1:CC 74  â”‚ 1:CC 82  â”‚ 1:CC 90  â”‚ 1:CC 98  â”‚
Enc 4   â”‚ 1:CC 11  â”‚ 1:CC 19  â”‚ 1:CC 27  â”‚ 1:CC 35  â”‚ 1:CC 75  â”‚ 1:CC 83  â”‚ 1:CC 91  â”‚ 1:CC 99  â”‚
Enc 5   â”‚ 1:CC 12  â”‚ 1:CC 20  â”‚ 1:CC 28  â”‚ 1:CC 36  â”‚ 1:CC 76  â”‚ 1:CC 84  â”‚ 1:CC 92  â”‚ 1:CC 100 â”‚
Enc 6   â”‚ 1:CC 13  â”‚ 1:CC 21  â”‚ 1:CC 29  â”‚ 1:CC 37  â”‚ 1:CC 77  â”‚ 1:CC 85  â”‚ 1:CC 93  â”‚ 1:CC 101 â”‚
Enc 7   â”‚ 1:CC 14  â”‚ 1:CC 22  â”‚ 1:CC 30  â”‚ 1:CC 38  â”‚ 1:CC 78  â”‚ 1:CC 86  â”‚ 1:CC 94  â”‚ 1:CC 102 â”‚
Enc 8   â”‚ 1:CC 15  â”‚ 1:CC 23  â”‚ 1:CC 31  â”‚ 1:CC 39  â”‚ 1:CC 79  â”‚ 1:CC 87  â”‚ 1:CC 95  â”‚ 1:CC 103 â”‚
â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

#### Cell Display Format
Compact representation: `{channel}:{type} {number}`

| Type | Display | Example |
|------|---------|---------|
| CC Absolute | `CC` | `1:CC 64` |
| CC Relative 1 | `r1` | `1:r1 64` |
| CC Relative 2 | `r2` | `1:r2 64` |
| CC 14-bit | `Hi` | `1:Hi 0` (shows MSB only) |
| Note | `N` | `1:N 60` |
| Program Change | `PC` | `1:PC` (no number for buttons) |
| Pitch Bend | `PB` | `1:PB` (channel-wide) |
| Aftertouch | `AT` | `1:AT` (channel-wide) |
| OFF | `--` | `--` |

#### Visual Indicators
- **Normal cell**: Default styling
- **Conflict cell**: Yellow/amber background with warning icon
- **Selected cell**: Highlighted border (for copy/paste operations)
- **Hover**: Show tooltip with full parameters

#### Cell Tooltip (on hover)
```
Encoder 3, Group 2
Channel: 1
Type: CC Absolute
CC#: 18
Min: 0, Max: 127
Acc: Acc3, Display: Std
```

#### Interaction

**Default Mode (Navigation):**
- **Click cell**: Jump to Focused View for that group, scrolled to that control
- **Right-click cell**: Context menu (Copy, Paste, View Conflicts)
- **Click column header (group name)**: Select entire group for bulk operations
- **Click row header (control name)**: Select that control across all groups

**Quick Edit Mode (Optional Toggle):**

A toolbar toggle enables inline editing of "headline fields" without leaving Overview:

```
[Quick Edit â˜]  â† Toggle button in Overview toolbar
```

When enabled:
- **Click cell**: Opens a compact popover editor for **headline fields only**:
  - Channel
  - Type  
  - CC/Note Number
- **Enter**: Commits changes
- **Escape**: Cancels
- All other parameters (min/max/acc/mode/display) remain Focused View only

**Why gated behind a toggle:**
- Overview is already dense with scanning, conflicts, and selection
- Inline editing increases accidental edits
- Explicit mode makes intent clear

### Fader Overview
Similar layout but includes Fader 9 as a separate row:

```
FADERS - Setup 1                                      [Focused View]
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
        â”‚ Group 1  â”‚ Group 2  â”‚ ...
â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼
Fad 1   â”‚ 1:CC 32  â”‚ 1:CC 40  â”‚
Fad 2   â”‚ 1:CC 33  â”‚ 1:CC 41  â”‚
...     â”‚          â”‚          â”‚
Fad 8   â”‚ 1:CC 39  â”‚ 1:CC 47  â”‚
â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼
Fad 9   â”‚ 1:CC 112 â”‚ 1:CC 112 â”‚  â† Note: same CC across groups = potential conflict
â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´
```

### Button Overview (Push and Green)
Show type and mode together:

```
Cell format: {channel}:{type} {number} [{mode}]
Example: 1:N 64 [M]  (Note 64, Momentary)
         1:CC 20 [T] (CC 20, Toggle)
```

Mode indicators: `[M]` = Momentary, `[T]` = Toggle

**Note on unmapped modes:** If button mode decoding is incomplete in the current build (e.g., only Note/CC confirmed, not OFF/PrGC/AFtt), display mode as `[-]` and omit from conflict keying until verified.

### Implementation Notes

```javascript
// Overview state
let overviewTab = 'encoders'; // 'encoders' | 'push' | 'green' | 'faders'
let selectedCells = new Set(); // For multi-select: "enc-2-3" = encoder, group 2, index 3

// Render overview grid
function renderOverview() {
    const main = document.getElementById('mainContent');
    main.innerHTML = '';
    
    // Tab bar
    const tabs = createOverviewTabs();
    main.appendChild(tabs);
    
    // Grid based on active tab
    switch (overviewTab) {
        case 'encoders': main.appendChild(createEncoderOverview()); break;
        case 'push': main.appendChild(createPushOverview()); break;
        case 'green': main.appendChild(createGreenOverview()); break;
        case 'faders': main.appendChild(createFaderOverview()); break;
    }
}

// MIDI Channel Helpers
// Storage: 0-15 (internal), Display/Keys: 1-16 (human-readable)
function chDisp(ch0) { return (ch0 | 0) + 1; }    // 0->1 ... 15->16
function chKey(ch0)  { return (ch0 | 0) + 1; }    // keep conflict keys human-readable

function applyChannelOffset(ch0, delta, wrapMode) {
    return applyOffset(ch0, delta, 0, 15, wrapMode);  // NOTE: 0-15 range for stored values
}

// Format control name for display (uses 1-based indices)
function formatControlName(controlType, group, index) {
    const g = group + 1;  // 0-7 -> 1-8
    const i = index + 1;  // 0-7 -> 1-8
    switch (controlType) {
        case 'encoder': return `Encoder G${g}.${i}`;
        case 'push':    return `Push G${g}.${i}`;
        case 'green':   return `Green G${g}.${i}`;
        case 'fader':   return `Fader G${g}.${i}`;
        case 'fader9':  return `Fader9 G${g}`;
        default:        return `${controlType} G${g}.${i}`;
    }
}

// Generate compact cell content
function formatEncoderCell(data) {
    const typeMap = {
        0: 'r1', 1: 'r2', 2: 'CC', 3: 'PC', 4: 'Hi', 5: 'PB', 6: 'AT'
    };
    const type = typeMap[data.type] || '??';
    const ch = chDisp(data.channel);  // Convert 0-15 to 1-16 for display
    
    // Program Change: show range instead of CC
    if (data.type === 3) {
        const lo = Math.max(0, Math.min(127, data.min | 0));
        const hi = Math.max(0, Math.min(127, data.max | 0));
        return `${ch}:PC ${Math.min(lo, hi)}-${Math.max(lo, hi)}`;
    }
    
    // Pitch bend and aftertouch have no CC number
    if (data.type === 5 || data.type === 6) {
        return `${ch}:${type}`;
    }
    
    return `${ch}:${type} ${data.cc}`;
}

// Generate compact cell content for faders
function formatFaderCell(data) {
    const typeMap = { 0: 'CC', 1: 'PC', 2: 'PB', 3: 'AT' };
    const type = typeMap[data.type] || '??';
    const ch = chDisp(data.channel);  // Convert 0-15 to 1-16 for display
    
    // Program Change: show range
    if (data.type === 1) {
        const lo = Math.max(0, Math.min(127, data.min | 0));
        const hi = Math.max(0, Math.min(127, data.max | 0));
        return `${ch}:PC ${Math.min(lo, hi)}-${Math.max(lo, hi)}`;
    }
    
    // Pitch bend and aftertouch have no CC number
    if (data.type === 2 || data.type === 3) {
        return `${ch}:${type}`;
    }
    
    return `${ch}:${type} ${data.cc}`;
}
```

---

## 2. Duplicate/Conflict Detection

### Purpose
Identify when multiple controls are assigned to the same MIDI message, which could cause:
- Unintended parameter coupling
- Confusion during live performance
- Wasted controller assignments

### Conflict Definition

A **conflict** exists when two or more controls within the same setup would send identical MIDI messages.

#### Conflict Key Structure
```javascript
// Unique identifier for a MIDI assignment
conflictKey = `${channel}-${messageType}-${number}`

// Message types (separate namespaces):
// - 'cc': Control Change (number = CC# 0-127)
// - 'note': Note On/Off (number = note# 0-127)
// - 'pc': Program Change (number = program# 0-127)
// - 'pb': Pitch Bend (number = null, channel-wide)
// - 'at': Aftertouch (number = null, channel-wide)
```

#### Special Cases

**14-bit High-Res Encoders (CCAh)**
- Occupy TWO CC numbers: N and N+32
- Must check conflicts for both

```javascript
// For encoder with type=CCAh and cc=5:
keys = ['1-cc-5', '1-cc-37']  // Both MSB and LSB
```

**Channel-Wide Messages**
- Pitch Bend: One per channel (no number)
- Aftertouch: One per channel (no number)

```javascript
// Pitch bend on channel 3
key = '3-pb-null'

// Aftertouch on channel 1
key = '1-at-null'
```

**Program Change Keys**
- Each program number (0-127) is a separate key
- Buttons: generate keys for upper and lower values
- Encoders/Faders: expand across min-max range

```javascript
// Button with upper=42, lower=0: generates two keys
keys = ['1-pc-42', '1-pc-0']

// Encoder PrGC with min=20, max=40: generates 21 keys
keys = ['1-pc-20', '1-pc-21', ..., '1-pc-40']
```

**Button Types**
- Note and CC buttons have numbers
- PrGC buttons: generate keys for upper/lower program numbers
- AFtt buttons conflict on channel (aftertouch is channel-wide)

### Conflict Scope

#### Within-Setup Conflicts (Primary)
Controls in the SAME setup that share assignments. This is the most important case since these controls could be active simultaneously.

**Sub-scopes:**
1. **Same group**: Definite conflict - both controls active at once
2. **Different groups**: Potential conflict - depends on which groups are selected

The UC4 has independent group selection for encoders vs faders/buttons, so:
- Encoder in Group 1 + Fader in Group 3 = **can conflict** (different group selectors)
- Encoder in Group 1 + Encoder in Group 3 = **cannot conflict** (same group selector, mutually exclusive)
- Fader in Group 1 + Button in Group 3 = **cannot conflict** (same group selector)

#### Cross-Setup Conflicts (Secondary/Optional)
Less critical since only one setup is active at a time, but useful for users who switch setups frequently.

### Conflict Classification (3-Way Model)

The UC4's independent group selectors create nuanced conflict scenarios. Using precise terminology avoids confusion:

| Category | Condition | Can Fire Simultaneously? | Default Visibility |
|----------|-----------|--------------------------|-------------------|
| **Concurrent** | Different selector domains (encoder-domain vs fader-domain), OR same domain + same group | YES | Shown |
| **Mutually-Exclusive Duplicates** | Same selector domain, different groups | NO (groups are exclusive) | Hidden (counted) |
| **Cross-Setup** | Same assignment in different setups | NO (setups are exclusive) | Hidden |

**Selector Domains:**
- **Encoder-domain**: Encoders + Push Buttons (share encoder group selector)
- **Fader-domain**: Faders 1-8 + Fader 9 + Green Buttons (share fader/button group selector)

**Examples:**
- Encoder G1.3 (CC 64) + Green Button G3.5 (CC 64) = **Concurrent** (different domains, can be active together)
- Encoder G1.3 (CC 64) + Encoder G5.3 (CC 64) = **Mutually-Exclusive** (same domain, different groups)
- Encoder G1.3 (CC 64) + Push Button G1.7 (CC 64) = **Concurrent** (same domain, same group)

### Conflict Data Structure

```javascript
// Built when setup changes or data is modified (key-centric, not pairwise)
const conflicts = {
    concurrent: new Map(),        // key -> { key, refs: ControlRef[] }
    mutuallyExclusive: new Map(), // key -> { key, refs: ControlRef[] }
    crossSetup: new Map()         // key -> { key, refs: ControlRef[] } (optional)
};

// ControlRef structure:
// { type: 'encoder'|'push'|'green'|'fader'|'fader9', group: 0-7, index: 0-7,
//   groupSelector: 'encoder'|'fader', subtype: 'CC'|'r1'|'r2'|'Hi'|'N'|'PC'|'PB'|'AT' }

// Optional: derive "active right now" based on current group selections
function getActiveNowConflicts(concurrent, encoderGroup, faderGroup) {
    const active = [];
    for (const [key, entry] of concurrent) {
        // Check if any refs are in currently selected groups
        const activeRefs = entry.refs.filter(ref => {
            return (ref.groupSelector === 'encoder' && ref.group === encoderGroup) ||
                   (ref.groupSelector === 'fader' && ref.group === faderGroup);
        });
        if (activeRefs.length >= 2) {
            active.push({ key, refs: activeRefs });
        }
    }
    return active;
}
```

### Conflict Detection Algorithm

```javascript
function buildConflictMap(setupIdx) {
    const assignments = new Map(); // conflictKey -> [controlRef, ...]
    
    // Collect all assignments
    for (let group = 0; group < 8; group++) {
        // Encoders
        for (let i = 0; i < 8; i++) {
            const data = getEncoderData(setupIdx, group, i);
            const keys = getEncoderConflictKeys(data);
            for (const key of keys) {
                if (!assignments.has(key)) assignments.set(key, []);
                assignments.get(key).push({
                    type: 'encoder', group, index: i,
                    groupSelector: 'encoder', // Which group selector controls this
                    subtype: ['r1', 'r2', 'CC', 'PC', 'Hi', 'PB', 'AT'][data.type] || 'CC'
                });
            }
        }
        
        // Push buttons (same group selector as encoders)
        for (let i = 0; i < 8; i++) {
            const data = getPushData(setupIdx, group, i);
            const keys = getButtonConflictKeys(data);
            for (const key of keys) {
                if (!assignments.has(key)) assignments.set(key, []);
                assignments.get(key).push({
                    type: 'push', group, index: i,
                    groupSelector: 'encoder',
                    subtype: { 0x10: 'N', 0x20: 'CC', 0x30: 'PC', 0x40: 'AT' }[data.typeNibble] || '?'
                });
            }
        }
        
        // Green buttons (fader/button group selector)
        for (let i = 0; i < 8; i++) {
            const data = getGreenData(setupIdx, group, i);
            const keys = getButtonConflictKeys(data);
            for (const key of keys) {
                if (!assignments.has(key)) assignments.set(key, []);
                assignments.get(key).push({
                    type: 'green', group, index: i,
                    groupSelector: 'fader',
                    subtype: { 0x10: 'N', 0x20: 'CC', 0x30: 'PC', 0x40: 'AT' }[data.typeNibble] || '?'
                });
            }
        }
        
        // Faders 1-8
        for (let i = 0; i < 8; i++) {
            const data = getFaderData(setupIdx, group, i);
            const keys = getFaderConflictKeys(data);
            for (const key of keys) {
                if (!assignments.has(key)) assignments.set(key, []);
                assignments.get(key).push({
                    type: 'fader', group, index: i,
                    groupSelector: 'fader',
                    subtype: ['CC', 'PC', 'PB', 'AT'][data.type] || 'CC'
                });
            }
        }
        
        // Fader 9
        const f9data = getFader9Data(setupIdx, group);
        const f9key = `${chKey(f9data.channel)}-cc-${f9data.cc}`;
        if (!assignments.has(f9key)) assignments.set(f9key, []);
        assignments.get(f9key).push({
            type: 'fader9', group, index: 0,
            groupSelector: 'fader',
            subtype: 'CC'
        });
    }
    
    // Build key-centric conflict structure (avoids O(nÂ²) pairwise explosion)
    const conflicts = {
        concurrent: new Map(),        // key -> { key, refs: ControlRef[] }
        mutuallyExclusive: new Map()  // key -> { key, refs: ControlRef[] }
    };
    
    for (const [key, refs] of assignments) {
        if (refs.length < 2) continue;
        
        // Check if ANY pair in this key is concurrent
        let hasConcurrent = false;
        for (let i = 0; i < refs.length && !hasConcurrent; i++) {
            for (let j = i + 1; j < refs.length && !hasConcurrent; j++) {
                if (isConcurrentConflict(refs[i], refs[j])) {
                    hasConcurrent = true;
                }
            }
        }
        
        // Whole key goes into one bucket (UI shows all refs together)
        if (hasConcurrent) {
            conflicts.concurrent.set(key, { key, refs });
        } else {
            conflicts.mutuallyExclusive.set(key, { key, refs });
        }
    }
    
    return conflicts;
}

function isConcurrentConflict(a, b) {
    // Different selector domains = can be active simultaneously = concurrent
    if (a.groupSelector !== b.groupSelector) {
        return true;
    }
    // Same selector domain + same group = concurrent (both active when that group selected)
    // Same selector domain + different groups = mutually exclusive
    return a.group === b.group;
}

function getEncoderConflictKeys(data) {
    const ch = chKey(data.channel);  // Convert 0-15 to 1-16 for human-readable keys
    switch (data.type) {
        case 0: // CCr1
        case 1: // CCr2
        case 2: // CCAb
            return [`${ch}-cc-${data.cc}`];
        case 3: { // PrGC - encoder sweeps a program range: expand all keys
            const lo = Math.max(0, Math.min(127, data.min | 0));
            const hi = Math.max(0, Math.min(127, data.max | 0));
            const a = Math.min(lo, hi), b = Math.max(lo, hi);
            const keys = [];
            for (let p = a; p <= b; p++) keys.push(`${ch}-pc-${p}`);
            return keys;
        }
        case 4: // CCAh (14-bit) - occupies CC and CC+32
            // Range guard: CCAh only valid for CC 0-31
            if (data.cc < 0 || data.cc > 31) return [];
            return [`${ch}-cc-${data.cc}`, `${ch}-cc-${data.cc + 32}`];
        case 5: // Pbnd
            return [`${ch}-pb-null`];
        case 6: // AFtt
            return [`${ch}-at-null`];
        default:
            return [];
    }
}

function getButtonConflictKeys(data) {
    if (data.typeNibble === 0x00) return []; // OFF
    const ch = chKey(data.channel);  // Convert 0-15 to 1-16 for human-readable keys
    switch (data.typeNibble) {
        case 0x10: // Note
            return [`${ch}-note-${data.cc}`];
        case 0x20: // CC
            return [`${ch}-cc-${data.cc}`];
        case 0x30: // PrGC - button sends upper on press, lower on release
            // Generate keys for both possible program numbers
            const keys = [`${ch}-pc-${data.upper}`];
            if (data.lower !== data.upper) {
                keys.push(`${ch}-pc-${data.lower}`);
            }
            return keys;
        case 0x40: // AFtt
            return [`${ch}-at-null`];
        default:
            return [];
    }
}

function getFaderConflictKeys(data) {
    const ch = chKey(data.channel);  // Convert 0-15 to 1-16 for human-readable keys
    switch (data.type) {
        case 0: // CCAb
            return [`${ch}-cc-${data.cc}`];
        case 1: { // PrGC - fader sweeps a program range: expand all keys
            const lo = Math.max(0, Math.min(127, data.min | 0));
            const hi = Math.max(0, Math.min(127, data.max | 0));
            const a = Math.min(lo, hi), b = Math.max(lo, hi);
            const keys = [];
            for (let p = a; p <= b; p++) keys.push(`${ch}-pc-${p}`);
            return keys;
        }
        case 2: // Pbnd
            return [`${ch}-pb-null`];
        case 3: // AFtt
            return [`${ch}-at-null`];
        default:
            return [];
    }
}
```

**Conflict Key Notes:**
- **Program Change**: Uses actual program numbers (0-127). Buttons generate keys for upper/lower values. Encoders/faders expand across the full swept range (exact collision detection).
- **14-bit CCAh**: Range-guarded to CC 0-31 only. Invalid values (CC > 31) generate no keys.
- **Conflict display**: Always show format `ChX <msg> <num> (<subtype>)` where subtype is: `CC`, `r1`, `r2`, `Hi`, `N`, `PC`, `PB`, `AT`

### UI Integration

#### Conflict Filter Chips
Toggle visibility of different conflict categories:

```
[Concurrent âœ“] [Mutually-Exclusive (12)] [Cross-Setup]
```

**Defaults:**
- `Concurrent`: **ON** (these can actually fire together)
- `Mutually-Exclusive`: **OFF** but **count always shown** (one-click reveal)
- `Cross-Setup`: **OFF**

When Mutually-Exclusive is enabled, those conflicts render in **dim/low-contrast** styling vs Concurrent.

#### Conflict Panel
A collapsible panel showing current conflicts (key-centric, not pairwise):

```
â”Œâ”€ Conflicts â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ [Concurrent âœ“] [Mutually-Exclusive (12)] [Cross-Setup]          â”‚
â”‚                                                                  â”‚
â”‚ â–¼ Concurrent Conflicts (3 keys)                                  â”‚
â”‚   âš  Ch1 CC 64: Encoder G1.4 (CC), Green G3.2 (CC), Fader G5.1   â”‚
â”‚   âš  Ch2 PB: Encoder G2.1 (PB), Fader G2.8 (PB)                  â”‚
â”‚   âš  Ch1 PC 42: Encoder G3.5 (PC), Green G1.7 (PC)               â”‚
â”‚                                                                  â”‚
â”‚ â–¶ Mutually-Exclusive Duplicates (12 keys) - click to expand     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

- Click a conflict to highlight all affected controls
- **Concurrent** = can fire simultaneously (bright/amber highlight)
- **Mutually-Exclusive** = same domain, different groups (dim highlight when enabled)
- Key-centric display avoids O(nÂ²) pairwise explosion (3+ collisions show all refs together)

**Domain Annotations:** When a key has mixed concurrent/mutually-exclusive refs (e.g., an encoder from Group 1 and another from Group 3 share a key with a fader), annotate each ref with its domain and group so mutually-exclusive refs don't appear "dangerous":

```
âš  Ch1 CC 64: Enc G1.4 [EncDom], Enc G3.2 [EncDom], Fader G5.1 [FadDom]
             ^^^^^^^^ concurrent with Fader (cross-domain)
                               ^^^^^^^^ ME with G1.4 (same domain, different group)
```

#### Conflict Display Detail
Show encoder type in parentheses for clarity, especially for relative modes that share CC numbers with absolute:

```
âš  Ch1 CC 64 (r1): Encoder G1.4 â†” Ch1 CC 64 (CC): Fader G5.1
```

This helps users understand *why* a conflict is flagged when an encoder in relative mode shares a CC with an absolute control.

#### Visual Indicators

**In Focused View:**
- Control cards with conflicts get a warning badge
- Tooltip shows what conflicts with what

**In Overview:**
- Cells with conflicts highlighted in amber/yellow
- Severity: bright for concurrent, dim for mutually-exclusive

#### Conflict Filter Defaults
```
[Concurrent âœ“]        - ON by default
[Mutually-Exclusive]  - OFF but count shown  
[Cross-Setup]         - OFF
[Highlight in views âœ“] - ON by default
```

---

## 3. Copy/Paste Operations

### Purpose
Enable efficient bulk editing by copying control configurations and pasting them elsewhere.

### Copy Scopes

Matching the hardware UC4's copy/paste capabilities, plus extensions:

| Scope | What's Copied | Paste Targets |
|-------|---------------|---------------|
| Single Control | All parameters for one encoder/button/fader | Same control type |
| Control Row | One control across all 8 groups | Same control type |
| Group Column | All controls of one type in a group | Same control type, any group |
| All of Type | All controls of one type in setup | Same control type, any setup |
| Group Name | 4-character name | Any group |
| Entire Setup | Everything | Any setup |
| **Across-Setups Row** | Same control index across multiple setups | Same control in other setups |
| **Swap** | Exchange two groups/controls | Same control types |

### Selector Domain Constraints

Paste targets are constrained by **selector domain** to prevent confusion:

| Domain | Controls | Group Selector |
|--------|----------|----------------|
| **Encoder-domain** | Encoders + Push Buttons | Encoder group (Shift + Encoder 1-8) |
| **Fader-domain** | Faders 1-8 + Fader 9 + Green Buttons | Fader/Button group (Shift + Green 1-8) |

When pasting to "All of Type" or domain-wide, only compatible controls within the same domain are affected.

### Swap Operations

Swap is a common workflow when reorganizing layouts:

| Swap Scope | Description |
|------------|-------------|
| Swap Groups | Exchange all controls between Group A â†” Group B (within same control type or whole domain) |
| Swap Controls | Exchange two individual controls of same type |
| Swap Setups | Exchange all data between Setup A â†” Setup B |

```javascript
// Swap groups example
function swapGroups(setupIdx, controlType, groupA, groupB) {
    for (let i = 0; i < 8; i++) {
        const dataA = getControlData(controlType, setupIdx, groupA, i);
        const dataB = getControlData(controlType, setupIdx, groupB, i);
        setControlData(controlType, setupIdx, groupA, i, dataB);
        setControlData(controlType, setupIdx, groupB, i, dataA);
    }
    // Record as single undo action
    recordUndo({ type: 'swap-groups', ... });
}
```

### Clipboard Structure

```javascript
const clipboard = {
    type: 'control'           // Single control
        | 'row'               // One control across all 8 groups
        | 'column'            // All controls of one type in a group
        | 'type-all'          // All controls of one type in setup
        | 'group-name'        // 4-character group name
        | 'setup'             // Entire setup
        | 'across-setups-row', // Same control index across multiple setups
    controlType: 'encoder' | 'push' | 'green' | 'fader' | 'fader9' | null,
    sourceSetup: number | null,    // Single setup index (null for across-setups-row)
    sourceSetups?: number[],       // Only for across-setups-row
    sourceGroup: number | null,    // null for multi-group operations
    sourceIndex: number | null,    // null for multi-control operations
    data: ControlData | ControlData[] | AcrossSetupsData
};

// Data type for across-setups-row
interface AcrossSetupsData {
    [setupIdx: number]: {
        [groupIdx: number]: ControlData
    }
}

// NOTE: "Swap" is an OPERATION, not a clipboard type.
// Swap doesn't use the clipboard - it exchanges data in-place.
// See swapGroups() in Swap Operations section.

// Examples:

// Single encoder copied
clipboard = {
    type: 'control',
    controlType: 'encoder',
    sourceSetup: 0,
    sourceGroup: 2,
    sourceIndex: 5,
    data: {
        channel: 0, type: 2, cc: 45, min: 0, max: 127, acc: 3, display: 1  // channel 0 = Ch1 displayed
    }
};

// Encoder column (all 8 encoders in group 2)
clipboard = {
    type: 'column',
    controlType: 'encoder',
    sourceSetup: 0,
    sourceGroup: 2,
    sourceIndex: null,
    data: [
        { channel: 0, type: 2, cc: 40, ... },  // channel 0 = Ch1 displayed
        { channel: 0, type: 2, cc: 41, ... },
        // ... 8 total
    ]
};

// Encoder row (encoder 3 across all groups)
clipboard = {
    type: 'row',
    controlType: 'encoder',
    sourceSetup: 0,
    sourceGroup: null,
    sourceIndex: 2,
    data: [
        { channel: 0, type: 2, cc: 24, ... },  // Group 0, channel 0 = Ch1 displayed
        { channel: 0, type: 2, cc: 32, ... },  // Group 1
        // ... 8 total
    ]
};

// Across-setups row (encoder 3 from setups 0-15)
clipboard = {
    type: 'across-setups-row',
    controlType: 'encoder',
    sourceSetup: null,           // Not used for this type
    sourceSetups: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15],
    sourceGroup: null,           // all groups
    sourceIndex: 2,              // encoder 3
    data: {
        0: { 0: {...}, 1: {...}, ... },  // Setup 0, all groups
        1: { 0: {...}, 1: {...}, ... },  // Setup 1, all groups
        // ... etc
    }
};
```

### Copy Operations

#### Context Menu (Right-Click)
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Copy Control                â”‚
â”‚ Copy Row (Enc 3 Ã— 8 groups) â”‚
â”‚ Copy Column (Group 2 Encs)  â”‚
â”‚ â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ â”‚
â”‚ Paste                       â”‚
â”‚ Paste Special...            â”‚
â”‚ â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ â”‚
â”‚ Reset to Factory Default    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

#### Keyboard Shortcuts
- `Ctrl+C` / `Cmd+C`: Copy selected
- `Ctrl+V` / `Cmd+V`: Paste
- `Ctrl+Shift+C`: Copy column
- `Ctrl+Alt+C`: Copy row

#### Selection Model
```javascript
let selection = {
    mode: 'none' | 'single' | 'row' | 'column' | 'multi',
    controlType: 'encoder' | 'push' | 'green' | 'fader',
    items: [
        { group: 0, index: 3 },
        { group: 0, index: 4 },
        // ...
    ]
};

// Multi-select: Ctrl+Click or Shift+Click in Overview
// Row select: Click row header in Overview
// Column select: Click column header in Overview
```

### Paste Operations

#### Basic Paste
Paste clipboard contents to:
- Current selection (if compatible)
- Current focused control (in Focused View)
- Clicked cell (in Overview)

#### Paste Special Dialog
For advanced paste options:

```
â”Œâ”€ Paste Special â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                         â”‚
â”‚ Source: Encoder G2.5 (Ch1 CC45)                        â”‚
â”‚                                                         â”‚
â”‚ Paste to:                                               â”‚
â”‚   â—‹ Current selection                                   â”‚
â”‚   â—‹ All encoders in current group                       â”‚
â”‚   â—‹ Same encoder in all groups                          â”‚
â”‚   â—‹ All encoders in setup                               â”‚
â”‚                                                         â”‚
â”‚ Options:                                                â”‚
â”‚   â˜‘ Channel                                             â”‚
â”‚   â˜‘ Type                                                â”‚
â”‚   â˜‘ CC/Note Number    [  ] Auto-increment by: [1]      â”‚
â”‚   â˜‘ Min/Max Values                                      â”‚
â”‚   â˜‘ Acceleration/Mode                                   â”‚
â”‚   â˜‘ Display Mode                                        â”‚
â”‚                                                         â”‚
â”‚ Transforms:                                             â”‚
â”‚   Channel offset:  [+0 â–¼] (-15 to +15)                 â”‚
â”‚   Number offset:   [+0 â–¼] (-127 to +127)               â”‚
â”‚   Number wrap:     â—‹ Clamp 0-127  â— Wrap around        â”‚
â”‚                                                         â”‚
â”‚              [Cancel]  [Paste]                          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Auto-increment**: When pasting to multiple destinations, increment CC/note numbers sequentially. Useful for setting up sequential CCs across a group.

**Transforms**:
- **Channel offset**: Shift all channels by N (e.g., copy from Ch1, paste with +3 = Ch4)
- **Number offset**: Shift CC/Note numbers by N
- **Wrap mode**: When offset pushes value past limits, either clamp or wrap around
  - Applies to **both** channel (1-16) and number (0-127) offsets
  - Clamp: values stop at boundaries (e.g., Ch14 + 5 â†’ Ch16)
  - Wrap: values wrap around (e.g., Ch14 + 5 â†’ Ch3)

**14-bit CCAh Constraint**: When pasting to an encoder with type CCAh, the CC number is automatically constrained to 0-31. UI shows informational note: "Will occupy CC N and CC N+32"

### Implementation

```javascript
function copyControl(controlType, setupIdx, group, index) {
    const data = getControlData(controlType, setupIdx, group, index);
    clipboard = {
        type: 'control',
        controlType,
        sourceSetup: setupIdx,
        sourceGroup: group,
        sourceIndex: index,
        data: { ...data }
    };
    showToast('Copied ' + formatControlName(controlType, group, index));
}

function copyColumn(controlType, setupIdx, group) {
    const count = controlType === 'fader9' ? 1 : 8;
    const data = [];
    for (let i = 0; i < count; i++) {
        data.push(getControlData(controlType, setupIdx, group, i));
    }
    clipboard = {
        type: 'column',
        controlType,
        sourceSetup: setupIdx,
        sourceGroup: group,
        sourceIndex: null,
        data
    };
    showToast(`Copied ${controlType} column (Group ${group + 1})`);
}

function pasteToControl(clip, controlType, setupIdx, group, index, options = {}) {
    if (!clip || !canPaste(clip, controlType)) {
        showToast('Nothing to paste', 'error');
        return;
    }
    
    // Record for undo
    const before = getControlData(controlType, setupIdx, group, index);
    
    // Merge clipboard data with options
    let newData = { ...clip.data };
    
    // Apply selective paste if options provided
    if (options.selective) {
        const current = getControlData(controlType, setupIdx, group, index);
        for (const key of Object.keys(current)) {
            if (!options.include[key]) {
                newData[key] = current[key]; // Keep current value
            }
        }
    }
    
    // Apply channel offset (storage is 0-15)
    if (options.channelOffset) {
        newData.channel = applyChannelOffset(newData.channel, options.channelOffset, options.wrapMode);
    }
    
    // Apply number offset (for CC/Note)
    if (options.numberOffset) {
        newData.cc = applyOffset(newData.cc, options.numberOffset, 0, 127, options.wrapMode);
    }
    
    // Apply auto-increment (sequential paste)
    if (options.autoIncrement && options.incrementBy) {
        const delta = options.incrementBy * (options.incrementIndex ?? 0);
        newData.cc = applyOffset(newData.cc, delta, 0, 127, options.wrapMode);
    }
    
    setControlData(controlType, setupIdx, group, index, newData);
    
    // Record undo
    recordUndo({
        type: 'control',  // Changed from 'paste' for consistency
        description: `Paste to ${formatControlName(controlType, group, index)}`,
        controlType,
        setupIdx,
        group,
        index,
        before,
        after: newData
    });
    
    // Single-control paste must trigger autosave + modified UI
    markModified();
    scheduleConflictRebuild();
}

// Helper for offset with wrap/clamp
function applyOffset(value, delta, min, max, wrapMode) {
    const v = value + delta;
    if (wrapMode === 'wrap') {
        const range = max - min + 1;
        return min + (((v - min) % range) + range) % range;
    } else {
        return Math.max(min, Math.min(max, v)); // clamp
    }
}

// Main paste dispatcher - routes to appropriate paste function based on clipboard type
function paste(clip, target, options = {}) {
    if (!clip) {
        showToast('Nothing to paste', 'error');
        return;
    }
    
    switch (clip.type) {
        case 'control':
            pasteToControl(clip, target.controlType, target.setupIdx, target.group, target.index, options);
            break;
            
        case 'row':
            pasteRowToGroups(clip, target.controlType, target.setupIdx, target.index, options);
            break;
            
        case 'column':
            pasteColumnToGroup(clip, target.controlType, target.setupIdx, target.group, options);
            break;
            
        case 'type-all':
            pasteTypeAllToSetup(clip, target.controlType, target.setupIdx, options);
            break;
            
        case 'setup':
            pasteSetupToSetup(clip, target.setupIdx, options);
            break;
            
        case 'group-name':
            pasteGroupName(clip, target.setupIdx, target.group, options);
            break;
            
        case 'across-setups-row':
            pasteAcrossSetupsRow(clip, target.setupIdxList, target.index, options);
            break;
            
        default:
            showToast('Unknown clipboard type', 'error');
    }
}

// Batch undo helpers
function beginBatch(description) {
    return { type: 'batch', description, actions: [] };
}

function batchApplyControl(batch, controlType, setupIdx, group, index, after) {
    const before = getControlData(controlType, setupIdx, group, index);
    // Use silentModified + silentEvents: batch paste handles these once at the end
    setControlData(controlType, setupIdx, group, index, after, { silentModified: true, silentEvents: true });
    batch.actions.push({ controlType, setupIdx, group, index, before, after });
}

// Transform data with paste options
function transformControlData(data, options, incrementIndex = 0) {
    let newData = { ...data };
    if (options.channelOffset) {
        newData.channel = applyChannelOffset(newData.channel, options.channelOffset, options.wrapMode);
    }
    if (options.numberOffset) {
        newData.cc = applyOffset(newData.cc, options.numberOffset, 0, 127, options.wrapMode);
    }
    if (options.autoIncrement && options.incrementBy) {
        newData.cc = applyOffset(newData.cc, options.incrementBy * incrementIndex, 0, 127, options.wrapMode);
    }
    return newData;
}

// Multi-paste implementations with proper batch undo
function pasteRowToGroups(clip, controlType, setupIdx, index, options) {
    const batch = beginBatch(`Paste row to ${controlType} index ${index + 1} (all groups)`);
    for (let g = 0; g < 8; g++) {
        const data = clip.data[g];
        if (data) {
            const after = transformControlData(data, options, g);
            batchApplyControl(batch, controlType, setupIdx, g, index, after);
        }
    }
    if (batch.actions.length > 0) {
        recordUndo(batch);
        markModified();                    // Single autosave debounce
        scheduleConflictRebuild();         // Single conflict rebuild
        emit('dataChanged', { kind: 'batch', description: batch.description });
    }
}

function pasteColumnToGroup(clip, controlType, setupIdx, group, options) {
    const batch = beginBatch(`Paste column to ${controlType} Group ${group + 1}`);
    for (let i = 0; i < clip.data.length; i++) {
        const after = transformControlData(clip.data[i], options, i);
        batchApplyControl(batch, controlType, setupIdx, group, i, after);
    }
    if (batch.actions.length > 0) {
        recordUndo(batch);
        markModified();                    // Single autosave debounce
        scheduleConflictRebuild();         // Single conflict rebuild
        emit('dataChanged', { kind: 'batch', description: batch.description });
    }
}

function pasteTypeAllToSetup(clip, controlType, setupIdx, options) { 
    // TODO: batch paste all 8 groups Ã— 8 controls
}
function pasteSetupToSetup(clip, setupIdx, options) { 
    // TODO: batch paste entire setup
}
function pasteGroupName(clip, setupIdx, group, options) { 
    // TODO: single value paste
}
function pasteAcrossSetupsRow(clip, setupIdxList, index, options) { 
    // TODO: batch paste across multiple setups
}

// Check if clipboard can be pasted to target
// targetKind: 'encoder'|'push'|'green'|'fader'|'fader9'|'setup'|'group-name'
function canPaste(clip, targetKind) {
    if (!clip) return false;

    // Setup paste is special: only into setup targets
    if (clip.type === 'setup') return targetKind === 'setup';

    // Group name paste only into group name target
    if (clip.type === 'group-name') return targetKind === 'group-name';

    // Across-setups-row: only into same control type
    if (clip.type === 'across-setups-row') return clip.controlType === targetKind;

    // Normal clips: must match control type
    if (clip.type === 'control' || clip.type === 'row' || clip.type === 'column' || clip.type === 'type-all') {
        return clip.controlType === targetKind;
    }

    return false;
}
```

---

## 4. Undo/Redo System

### Purpose
Allow users to safely experiment with changes and recover from mistakes.

### Design Approach

**Command Pattern** with state deltas rather than full buffer snapshots (memory efficient).

### Undo Stack Structure

```javascript
const undoStack = [];  // Array of UndoAction
const redoStack = [];  // Cleared on new action
const MAX_UNDO = 100;  // Limit stack size

// Base action structure
interface UndoAction {
    type: string;
    timestamp: number;
    description: string;  // Human-readable for UI
    // ... type-specific data
}
```

### Action Types

#### Single Value Change
```javascript
{
    type: 'value',
    timestamp: Date.now(),
    description: 'Change Encoder G1.3 CC to 45',
    controlType: 'encoder',
    setupIdx: 0,
    group: 0,
    index: 2,
    param: 'cc',
    before: 64,
    after: 45
}
```

#### Control Change (Multiple Parameters)
```javascript
{
    type: 'control',
    timestamp: Date.now(),
    description: 'Paste to Encoder G1.3',
    controlType: 'encoder',
    setupIdx: 0,
    group: 0,
    index: 2,
    before: { channel: 0, type: 2, cc: 64, min: 0, max: 127, acc: 3, display: 1 },  // channel 0 = Ch1
    after: { channel: 1, type: 2, cc: 45, min: 10, max: 100, acc: 2, display: 1 }   // channel 1 = Ch2
}
```

#### Batch Change (Multiple Controls)
```javascript
{
    type: 'batch',
    timestamp: Date.now(),
    description: 'Paste to all encoders in Group 2',
    actions: [
        { controlType: 'encoder', setupIdx: 0, group: 1, index: 0, before: {...}, after: {...} },
        { controlType: 'encoder', setupIdx: 0, group: 1, index: 1, before: {...}, after: {...} },
        // ...
    ]
}
```

### Undo/Redo Operations

```javascript
function recordUndo(action) {
    // Add description if not provided
    if (!action.description) {
        action.description = generateDescription(action);
    }
    action.timestamp = Date.now();
    
    undoStack.push(action);
    
    // Clear redo stack (new action invalidates redo history)
    redoStack.length = 0;
    
    // Trim if over limit
    while (undoStack.length > MAX_UNDO) {
        undoStack.shift();
    }
    
    updateUndoRedoUI();
}

function undo() {
    if (undoStack.length === 0) return;
    
    const action = undoStack.pop();
    
    // Apply reverse
    applyAction(action, true); // true = reverse
    
    redoStack.push(action);
    updateUndoRedoUI();
    renderCurrentView();
}

function redo() {
    if (redoStack.length === 0) return;
    
    const action = redoStack.pop();
    
    // Apply forward
    applyAction(action, false);
    
    undoStack.push(action);
    updateUndoRedoUI();
    renderCurrentView();
}

function applyAction(action, reverse) {
    switch (action.type) {
        case 'value': {
            const value = reverse ? action.before : action.after;
            applyValueChange(action, value);
            break;
        }
            
        case 'control': {
            const data = reverse ? action.before : action.after;
            setControlData(action.controlType, action.setupIdx, action.group, action.index, data, { silentModified: true, silentEvents: true });
            break;
        }
            
        case 'batch': {
            for (const subAction of action.actions) {
                const subData = reverse ? subAction.before : subAction.after;
                setControlData(subAction.controlType, subAction.setupIdx, subAction.group, subAction.index, subData, { silentModified: true, silentEvents: true });
            }
            break;
        }
    }
    
    // After any change (single calls for entire action):
    markModified();
    emit('dataChanged', { kind: action.type, description: action.description });
    
    // Only rebuild conflicts if action affects routing params
    if (actionAffectsConflicts(action)) {
        scheduleConflictRebuild();
    }
}

// Apply a single parameter value change
function applyValueChange(action, value) {
    // Get current control data
    const data = getControlData(action.controlType, action.setupIdx, action.group, action.index);
    
    // Update the specific parameter
    data[action.param] = value;
    
    // Write back through setControlData (silentModified + silentEvents since applyAction handles these)
    setControlData(action.controlType, action.setupIdx, action.group, action.index, data, { silentModified: true, silentEvents: true });
}
```

### Conflict Rebuild Gating

Only rebuild conflict map when changes affect routing parameters:

```javascript
// Parameters that affect conflict detection
const CONFLICT_PARAMS = new Set([
    'channel', 'type', 'cc', 'note', 'typeNibble', 
    'upper', 'lower', 'mode', 'min', 'max'  // upper/lower/min/max affect PrGC ranges
]);

function actionAffectsConflicts(action) {
    if (action.type === 'value') {
        return CONFLICT_PARAMS.has(action.param);
    }
    // control and batch actions are conservative - always rebuild
    // (could optimize by checking which params changed, but not worth complexity)
    return true;
}
```

### Conflict Rebuild Debouncing

Conflict detection should be more responsive than autosave. Use separate debounce:

```javascript
const CONFLICT_REBUILD_DELAY = 200; // 200ms - feels responsive
let conflictRebuildTimer = null;

function scheduleConflictRebuild() {
    clearTimeout(conflictRebuildTimer);
    conflictRebuildTimer = setTimeout(() => {
        rebuildConflictMap();
        updateConflictUI();
    }, CONFLICT_REBUILD_DELAY);
}
```

### Coalescing Rapid Changes

When a user drags a number input or rapidly changes values, we don't want 50 undo entries. Coalesce changes to the same parameter within a time window:

```javascript
const COALESCE_WINDOW = 1000; // 1 second

function recordUndoCoalesced(action) {
    const last = undoStack[undoStack.length - 1];
    
    // Check if we should coalesce with previous action
    if (last && 
        last.type === 'value' &&
        action.type === 'value' &&
        last.controlType === action.controlType &&
        last.setupIdx === action.setupIdx &&
        last.group === action.group &&
        last.index === action.index &&
        last.param === action.param &&
        (Date.now() - last.timestamp) < COALESCE_WINDOW) {
        
        // Update the existing action's 'after' value
        last.after = action.after;
        last.timestamp = Date.now();
        // Keep original 'before' value
        return;
    }
    
    // Otherwise record as new action
    recordUndo(action);
}
```

### UI Integration

#### Toolbar Buttons
```
[â†¶ Undo] [â†· Redo]
```

With tooltips showing the action description:
- "Undo: Change Encoder G1.3 CC to 45"
- "Redo: Paste to all encoders in Group 2"

#### Keyboard Shortcuts
- `Ctrl+Z` / `Cmd+Z`: Undo
- `Ctrl+Shift+Z` / `Cmd+Shift+Z`: Redo
- `Ctrl+Y` / `Cmd+Y`: Redo (alternative)

#### History Panel (Optional Enhancement)
Dropdown showing recent actions:

```
â”Œâ”€ History â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ â— Change Encoder G1.3 CC    â”‚ â† Current state
â”‚ â—‹ Paste to Encoder G1.3     â”‚
â”‚ â—‹ Change Fader G2.1 Channel â”‚
â”‚ â—‹ Copy Encoder G1.5         â”‚ (no state change)
â”‚ â—‹ Import JSON               â”‚
â”‚ ...                         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

Click any item to jump to that state (undoing/redoing multiple actions at once).

### Integration with Existing Code

Modify update functions to record undo:

```javascript
// Before (current implementation)
function updateEncoder(idx, param, value) {
    const data = getEncoderData(currentSetup, encoderGroup, idx);
    data[param] = parseInt(value);
    setEncoderData(State.currentSetup, State.encoderGroup, idx, data);
}

// After (with undo support + autosave + conflict rebuild)
function updateEncoder(idx, param, value) {
    const data = getEncoderData(State.currentSetup, State.encoderGroup, idx);
    const before = data[param];
    const after = parseInt(value, 10);
    
    if (before === after) return; // No change
    
    data[param] = after;
    setControlData('encoder', State.currentSetup, State.encoderGroup, idx, data);
    
    recordUndoCoalesced({
        type: 'value',
        controlType: 'encoder',
        setupIdx: State.currentSetup,
        group: State.encoderGroup,
        index: idx,
        param,
        before,
        after
    });
    
    markModified();  // REQUIRED: triggers autosave debounce + status update
    
    // Only rebuild conflicts if this param affects MIDI routing
    if (CONFLICT_PARAMS.has(param)) {
        scheduleConflictRebuild();
    }
}
```

---

## 5. Session Persistence

### Purpose
Protect unsaved work from accidental page refresh or browser closure, building on the existing auto-load factory defaults behavior.

### Context: Current Auto-Load Flow

The editor already auto-loads `factory_default.syx` on startup (see Current Implementation Context above). The gap is that **edits are lost on page refresh** unless explicitly exported.

### Session Persistence Design

**Note:** All session code accesses state via the `State` object exported from `state.js` (e.g., `State.rawBuffer`, `State.currentSetup`, `State.dirtyBanks`). This keeps the single source of truth pattern consistent.

#### What to Persist (localStorage)
- **Raw SysEx buffer** (the 100,640 bytes)
- **UI state**: currentSetup, encoderGroup, faderGroup, currentView
- **Dirty checksums set** (which banks have been modified)
- **Timestamp** of last edit

#### What NOT to Persist
- **Undo/redo stacks** - Too large, brittle across spec changes, creates confusing "time travel" after re-import

#### dirtyBanks Format
The `dirtyBanks` set contains strings identifying modified banks:
```
Format: "${setupIdx}-${sectionId}-${bank}"
Example: "0-28-64" (Setup 0, Section 0x1C, Bank 0x40)
```
This format is stable across reload and easy to debug.

#### dirtyBanks Semantics: "Touched" vs "Currently Dirty"

**Current implementation: "Touched since baseline" (add-only)**

The `dirtyBanks` set grows when banks are modified but is never reduced (except on baseline reset). This means:
- Edit something â†’ adds bank key
- Undo back to original values â†’ bank key remains
- Refresh â†’ restore prompt appears (even though net state is clean)

**This is intentional.** The "touched" semantics are simple, robust, and require no hashing. The tradeoff is occasional false-positive restore prompts after undo-to-clean.

**Future enhancement (optional): "Currently dirty" semantics**

To make restore prompts truly reflect "actual unsaved edits":
```javascript
// On baseline set (import / loadDefault / export):
baselineCrcByBank[key] = crc32(bytesOfBank);

// In setControlData(), after encoding:
const currentCrc = crc32(bytesOfBank);
if (currentCrc === baselineCrcByBank[key]) {
    State.dirtyBanks.delete(key);
} else {
    State.dirtyBanks.add(key);
}
```

This upgrade is backwards-compatible and can be added later without spec changes.

#### Storage Key
```javascript
const STORAGE_KEY = 'uc4-editor-session';
const SESSION_VERSION = 1; // Increment if format changes
```

#### Save Session (with error handling)
```javascript
function saveSession() {
    try {
        const sessionData = {
            version: SESSION_VERSION,
            timestamp: Date.now(),
            buffer: Array.from(State.rawBuffer),  // Convert Uint8Array for JSON
            uiState: {
                currentSetup: State.currentSetup,
                encoderGroup: State.encoderGroup,
                faderGroup: State.faderGroup,
                currentView: State.currentView
            },
            dirtyBanks: Array.from(State.dirtyBanks)
        };
        
        localStorage.setItem(STORAGE_KEY, JSON.stringify(sessionData));
    } catch (e) {
        console.warn('Session save failed:', e);
        // Optional: notify user if quota exceeded
        if (e.name === 'QuotaExceededError') {
            showToast('Autosave failed (storage full)', 'error');
        }
    }
}
```

**Storage Size Note:** `Array.from(Uint8Array)` serialized to JSON produces ~350-500KB (each byte becomes up to 3 chars + commas). This is well within localStorage limits (typically 5-10MB). Future optimization: store buffer as base64 for smaller/faster storage.

#### Session Validation on Load
```javascript
function loadSession() {
    try {
        const json = localStorage.getItem(STORAGE_KEY);
        if (!json) return null;
        
        const session = JSON.parse(json);
        
        // Version check
        if (session.version !== SESSION_VERSION) {
            console.warn('Session version mismatch, discarding');
            clearSession();  // MUST clear or we'll reprompt forever
            return null;
        }
        
        // Buffer integrity checks
        if (!session.buffer || session.buffer.length !== 100640) {
            console.warn('Invalid buffer length, discarding');
            clearSession();
            return null;
        }
        
        // SysEx framing check (conditional - only if buffer appears to be framed)
        // If buffer starts with F0, it's a .syx blob and must end with F7
        if (session.buffer[0] === 0xF0) {
            if (session.buffer[session.buffer.length - 1] !== 0xF7) {
                console.warn('Invalid SysEx framing, discarding');
                clearSession();
                return null;
            }
        }
        // If buffer doesn't start with F0, rely on length check only
        // (could add UC4-specific signature check here if known)
        
        return session;
    } catch (e) {
        console.error('Failed to parse session:', e);
        clearSession();
        return null;
    }
}
```

### Auto-Save Behavior

```javascript
const AUTO_SAVE_DELAY = 2000; // 2 seconds after last edit
let autoSaveTimer = null;

function scheduleAutoSave() {
    clearTimeout(autoSaveTimer);
    autoSaveTimer = setTimeout(() => {
        saveSession();
    }, AUTO_SAVE_DELAY);
}

// Call scheduleAutoSave() in markModified()
function markModified() {
    State.isModified = true;
    updateStatus('modified', 'Modified - remember to export');
    scheduleAutoSave();  // NEW
}
```

### Restore Flow on Load

**Priority order:**
1. If valid session with unsaved changes exists â†’ prompt restore
2. Else if `factory_default.syx` available â†’ load it
3. Else â†’ await manual import

```javascript
async function initializeEditor() {
    // 0. Check if user previously said "don't ask again"
    if (sessionStorage.getItem('uc4-skip-restore') === 'true') {
        clearSession();  // Discard any lingering session
        State.dirtyBanks.clear();  // Reset baseline
        await loadDefaultSysEx();
        return;
    }
    
    // 1. Check for saved session with actual unsaved changes
    const savedSession = loadSession();
    
    // Only prompt if there are dirty banks (actual unsaved edits)
    if (savedSession && savedSession.dirtyBanks && savedSession.dirtyBanks.length > 0) {
        const timeSince = formatTimeSince(savedSession.timestamp);
        
        // Show restore prompt
        const result = await showRestoreDialog(timeSince, savedSession);
        
        if (result === 'restore') {
            restoreSession(savedSession);
            return;
        } else if (result === 'discard-permanently') {
            clearSession();
            State.dirtyBanks.clear();  // Reset baseline on discard
            // Don't prompt again this browser session
            sessionStorage.setItem('uc4-skip-restore', 'true');
        } else {
            clearSession();
            State.dirtyBanks.clear();  // Reset baseline on discard
        }
    } else if (savedSession) {
        // Session exists but no dirty banks = clean baseline, just clear it
        clearSession();
    }
    
    // 2. Load factory defaults (also clears dirtyBanks internally)
    await loadDefaultSysEx();
    
    // 3. If that fails, await manual import (handled by loadDefaultSysEx fallback)
}
```

### Restore Dialog

```
â”Œâ”€ Restore Previous Session? â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                         â”‚
â”‚ Found unsaved changes from 15 minutes ago.              â”‚
â”‚                                                         â”‚
â”‚ Setup 3, Encoder Group 2                                â”‚
â”‚ 12 modified banks                                       â”‚
â”‚                                                         â”‚
â”‚ â˜ Don't ask again this session                         â”‚
â”‚                                                         â”‚
â”‚        [Discard & Load Factory]  [Restore Session]      â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**"Don't ask again" behavior:** Uses `sessionStorage` (not `localStorage`) so it only persists until the tab/browser closes. Prevents annoying repeated prompts if user intentionally discards.

### Clear Session on Export

After successful `.syx` export, clear the saved session (user explicitly saved their work):

```javascript
function exportSysEx() {
    // ... existing export logic ...
    
    clearSession();              // User saved their work
    State.isModified = false;
    State.dirtyBanks.clear();    // Reset baseline - no banks are "dirty" after export
    updateStatus('loaded', 'Exported successfully');
}
```

### Clear Session on Import

When user imports an external `.syx` file, that becomes the new baseline. Clear any previous session (do NOT immediately save - session will be created on first edit):

```javascript
// In sysexInput change handler, after successful validation:
clearSession();              // Clear any previous session
State.isModified = false;    // Fresh import = no unsaved changes
State.dirtyBanks.clear();    // Reset baseline - new import is clean slate
// Session will be created on first edit via markModified() â†’ saveSession()
```

**Why not save immediately?** Saving a "clean" session would cause restore prompts for unmodified data. Only sessions with `dirtyBanks.length > 0` should prompt for restore.

---

## Implementation Phases

### Phase 1: Undo/Redo Foundation
**Priority: High** (Safety net for all subsequent features)

1. Implement undo/redo stack
2. Add recording to all existing update functions
3. Add coalescing for rapid changes
4. Add toolbar buttons and keyboard shortcuts
5. Test thoroughly with various edit patterns

### Phase 2: Session Persistence
**Priority: High** (Protects work immediately)

1. Implement localStorage save/load
2. Add auto-save on edit with debounce
3. Add restore dialog on page load
4. Clear session on successful export
5. Handle storage errors gracefully

### Phase 3: Conflict Detection
**Priority: High** (Informs Overview display)

1. Implement conflict key generation for all control types
2. Build conflict map on setup load/change
3. Categorize concurrent vs mutually-exclusive conflicts
4. Add conflict filter chips
5. Add conflict panel UI
6. Add visual indicators in Focused View

### Phase 4: Overview Mode
**Priority: Medium** (Depends on conflict detection)

1. Create tab-based control type views
2. Implement compact grid rendering
3. Add cell tooltips
4. Integrate conflict highlighting
5. Add click-to-navigate functionality
6. Add selection model (single, row, column)
7. Add Quick Edit mode toggle

### Phase 5: Copy/Paste
**Priority: Medium** (Benefits from Overview selection)

1. Implement clipboard structure
2. Add copy operations (control, row, column)
3. Add basic paste operation
4. Add context menu
5. Add keyboard shortcuts
6. Implement Paste Special dialog with transforms
7. Add auto-increment feature
8. Add swap operations
9. Add across-setups copy/paste

### Phase 6: Polish & Integration
1. Undo/redo history panel (optional)
2. Cross-setup conflict detection (optional)
3. Performance optimization for large operations
4. Comprehensive keyboard navigation
5. Mobile/touch support for Overview

---

## Resolved Design Decisions

These questions from the initial spec have been resolved:

| Question | Decision | Rationale |
|----------|----------|-----------|
| **Conflict visibility default** | Concurrent ON, Mutually-Exclusive OFF (but counted) | Potential conflicts flood UI on factory dumps; one-click reveal when needed |
| **Undo persistence** | NO - persist buffer/session only | Undo stacks are large, brittle across versions, create confusing time-travel |
| **Copy/paste across setups** | YES | Common workflow (consistent layouts); guard with setup checklist in Paste Special |
| **Overview edit-in-place** | Click-through default, Quick Edit toggle for headline fields | Prevents accidental edits; explicit mode makes intent clear |
| **Multi-select in Focused View** | NO (initially) | Keep Focused View simple; multi-select is Overview's strength |

---

## Appendix A: Keyboard Shortcuts Summary

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z or Ctrl+Y | Cmd+Shift+Z |
| Copy | Ctrl+C | Cmd+C |
| Paste | Ctrl+V | Cmd+V |
| Paste Special | Ctrl+Shift+V | Cmd+Shift+V |
| Copy Column | Ctrl+Shift+C | Cmd+Shift+C |
| Copy Row | Ctrl+Alt+C | Cmd+Option+C |
| Swap Selected | Ctrl+Shift+X | Cmd+Shift+X |
| Select All in Group | Ctrl+A | Cmd+A |
| Deselect | Escape | Escape |
| Navigate | Arrow keys | Arrow keys |
| Switch Overview Tab | Ctrl+1/2/3/4 | Cmd+1/2/3/4 |
| Toggle Quick Edit | E | E |
| Jump to Focused View | Enter | Enter |

---

## Appendix B: Conflict Key Reference

| Message Type | Key Format | Example | Notes |
|--------------|------------|---------|-------|
| Control Change | `{ch}-cc-{num}` | `1-cc-64` | CC number 0-127 |
| Note | `{ch}-note-{num}` | `1-note-60` | Note number 0-127 |
| Program Change | `{ch}-pc-{num}` | `3-pc-42` | Program number 0-127 |
| Pitch Bend | `{ch}-pb-null` | `1-pb-null` | Channel-wide |
| Aftertouch | `{ch}-at-null` | `2-at-null` | Channel-wide |
| 14-bit CC (CCAh) | `{ch}-cc-{N}` + `{ch}-cc-{N+32}` | `1-cc-5`, `1-cc-37` | N must be 0-31 |

**Program Change Details:**
- Buttons: Generate keys for both upper (press) and lower (release) program numbers
- Encoders/Faders: Expand keys across the full swept range (min to max inclusive) for exact collision detection. Example: PrGC with min=20, max=40 generates 21 keys: `{ch}-pc-20` through `{ch}-pc-40`

**14-bit CCAh Guards:**
- CC number must be 0-31 (invalid values generate no conflict keys)
- Automatically occupies both N and N+32

**Display Subtypes for Conflict Panel:**
- `CC` = 7-bit absolute
- `r1` = relative mode 1
- `r2` = relative mode 2
- `Hi` = 14-bit high-res
- `N` = Note
- `PC` = Program Change
- `PB` = Pitch Bend
- `AT` = Aftertouch

---

## Appendix C: Selector Domains

Understanding selector domains is critical for conflict classification:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                        ENCODER DOMAIN                           â”‚
â”‚  Controlled by: Shift + Encoder 1-8                             â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                               â”‚
â”‚  â”‚  Encoders   â”‚  â”‚Push Buttons â”‚                               â”‚
â”‚  â”‚  (8 Ã— 8)    â”‚  â”‚   (8 Ã— 8)   â”‚                               â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                               â”‚
â”‚  Groups 1-8 are MUTUALLY EXCLUSIVE within this domain           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                        FADER DOMAIN                             â”‚
â”‚  Controlled by: Shift + Green Button 1-8                        â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”             â”‚
â”‚  â”‚  Faders 1-8 â”‚  â”‚  Fader 9    â”‚  â”‚Green Buttonsâ”‚             â”‚
â”‚  â”‚  (8 Ã— 8)    â”‚  â”‚   (1 Ã— 8)   â”‚  â”‚   (8 Ã— 8)   â”‚             â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜             â”‚
â”‚  Groups 1-8 are MUTUALLY EXCLUSIVE within this domain           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

CROSS-DOMAIN: Encoder G1 + Fader G3 = CAN BE CONCURRENT
              (different group selectors, both can be active)

SAME-DOMAIN:  Encoder G1 + Encoder G3 = MUTUALLY EXCLUSIVE
              (same group selector, only one group active at a time)
```

---

## Appendix D: Implementation Checklist

### Module Boundaries (Recommended)

To avoid a monolithic codebase, organize into logical sections/files:

| Module | Responsibilities |
|--------|------------------|
| `state.js` | `rawBuffer`, `currentSetup`, `encoderGroup`, `faderGroup`, `currentView`, `dirtyBanks`, event emitter (`on`, `emit`) |
| `encode.js` | `getControlData`, `setControlData` (single source of truth for buffer mutations) |
| `undo.js` | `undoStack`, `redoStack`, `recordUndo`, `recordUndoCoalesced`, `applyAction`, `undo`, `redo` |
| `conflicts.js` | `buildConflictMap`, `scheduleConflictRebuild`, key generators (`chKey`, `chDisp`), `isConcurrentConflict` |
| `clipboard.js` | `clipboard`, `copy*`, `paste()`, batch helpers, transforms |
| `overview.js` | `renderOverview`, tab/selection model, quick-edit toggle + popover edits |
| `session.js` | `saveSession`, `loadSession`, `clearSession`, `initializeEditor`, restore dialog |
| `ui.js` | `markModified`, `showToast`, `updateStatus`, keyboard bindings |
| `app.js` | Event wiring (`on('dataChanged', ...)`, `on('conflictsChanged', ...)`) |

### Single Source of Truth Setter

All control data mutations must go through one function:

```javascript
// encode.js
export function setControlData(controlType, setupIdx, group, index, data, options = {}) {
    // 1. Encode into State.rawBuffer at correct section/bank offsets
    // 2. Add to State.dirtyBanks: `${setupIdx}-${sectionId}-${bank}`
    // 3. Unless options.silentModified: set State.isModified = true
    // 4. Unless options.silentEvents: emit('dataChanged', { controlType, setupIdx, group, index })
    // NOTE: setControlData() NEVER calls markModified() - that's the caller's responsibility
}
```

**Options:**
- `silentModified: true` â€” Skip `isModified` flag (used by batch/undo apply)
- `silentEvents: true` â€” Skip event emission (used when caller handles refresh)

Call graph:
- UI edits â†’ `updateX()` â†’ `recordUndoCoalesced()` â†’ `setControlData()`
- Paste â†’ `setControlData({ silentModified: true, silentEvents: true })` Ã— N â†’ `recordUndo(batch)` â†’ `markModified()` â†’ `emit('dataChanged')`
- Undo/Redo â†’ `applyAction()` â†’ `setControlData({ silentModified: true, silentEvents: true })` Ã— N â†’ `markModified()` â†’ `emit('dataChanged')`

### Debounce Rules

Keep autosave and conflict timers separate:
- `AUTO_SAVE_DELAY = 2000ms`
- `CONFLICT_REBUILD_DELAY = 200ms`

Undo/redo must trigger both:
- `markModified()` (autosave debounce)
- `scheduleConflictRebuild()` gated by `actionAffectsConflicts()`

### Batch Undo Correctness

Two rules to enforce:
1. Multi-paste must **not** call `recordUndo()` per cell
2. Multi-paste must **not** call `markModified()` or `emit('dataChanged')` per cell

Use `{ silentModified: true, silentEvents: true }` inside loops; then call `markModified()`, `scheduleConflictRebuild()`, and emit one `dataChanged` event after the loop.

**Critical rule:** `setControlData()` may set `State.isModified`, but must **never** call `markModified()`. Autosave debouncing happens via explicit `markModified()` calls in mutation entrypoints (UI edits, paste completion, undo/redo completion).

**Invariant:** If an operation results in an edit the user would expect to survive refresh, it must call `markModified()` exactly once per user action (not per cell).

### `markModified()` Entrypoint Checklist

| Entry Point | Calls `markModified()`? | Notes |
|-------------|------------------------|-------|
| `updateX()` (UI edits) | âœ… must be explicit | After `setControlData()` + `recordUndoCoalesced()` |
| `recordUndoCoalesced()` | âŒ | Undo stack only, no autosave |
| `pasteToControl()` | âœ… explicit | Single-control paste |
| `pasteRowToGroups()` | âœ… explicit (once after loop) | Batch paste |
| `pasteColumnToGroup()` | âœ… explicit (once after loop) | Batch paste |
| `applyAction()` (undo/redo) | âœ… explicit | After all `setControlData()` calls |
| `setControlData()` | âŒ NEVER | Only sets `State.isModified` flag |
| Import handler | âŒ | Fresh baseline, no changes yet |
| `loadDefaultSysEx()` | âŒ | Fresh baseline |
| `exportSysEx()` | âŒ | Resets to clean baseline |

### Map Consistency

Conflicts use `new Map()` not arrays. All consumers must iterate:
```javascript
for (const [key, entry] of conflicts.concurrent) { ... }
// NOT: for (const entry of conflicts.concurrent)
```

---

## Appendix E: Test Matrix

### Conflict Keys

| Test Case | Expected Keys |
|-----------|---------------|
| CCAh CC=0 | `ch-cc-0`, `ch-cc-32` |
| CCAh CC=31 | `ch-cc-31`, `ch-cc-63` |
| CCAh CC=32 | `[]` (invalid, no keys) |
| PrGC min=40, max=20 | `ch-pc-20` through `ch-pc-40` (21 keys) |
| PC button upper=lower=42 | `ch-pc-42` (1 key) |
| PB/AT | `ch-pb-null`, `ch-at-null` |

### Classification

| Test Case | Expected |
|-----------|----------|
| Same domain + same group | Concurrent |
| Same domain + different groups | Mutually Exclusive |
| Cross-domain (encoder vs fader) | Concurrent |

### Undo/Redo

| Test Case | Expected |
|-----------|----------|
| Drag value input (rapid changes) | 1 undo entry (coalesced) |
| Paste column (8 controls) | 1 undo entry (batch) |
| Undo â†’ new action | Redo stack cleared |

### Session Persistence

| Test Case | Expected |
|-----------|----------|
| Session exists, `dirtyBanks=[]` | No prompt, session cleared |
| Quota exceeded | No crash, toast shown, operation continues |
| Page refresh with unsaved changes | Prompt appears |

### Autosave / Restore Gating

| Test Case | Expected |
|-----------|----------|
| UI edit + wait 2s | localStorage session written |
| Paste column + wait 2s | Single session write (debounced), not 8 |
| Undo/redo triggers autosave | Session timestamp updates |
| Edit then undo back ("touched" semantics) | Restore prompt still appears (documented behavior) |
| Export clears session | `dirtyBanks` empty, `isModified=false`, no restore prompt on refresh |
| Import new file | Previous session cleared, `dirtyBanks` empty |
