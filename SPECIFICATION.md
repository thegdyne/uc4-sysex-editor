# UC4 SysEx Editor — Technical Specification

Version: 2.1.0  
Date: 2026-01-15  
Status: Implemented

---

## Overview

The UC4 SysEx Editor is a single-page web application for editing Faderfox UC4 MIDI controller configurations. It operates entirely client-side with no server requirements.

### Scope

| Capability | Description |
|------------|-------------|
| **Setups** | All 18 setups fully editable |
| **Groups** | All 8 groups per setup |
| **Controls** | Encoders, Push Buttons, Green Buttons, Faders, Fader 9 |
| **File formats** | SysEx (.syx), JSON (full and single-setup) |
| **Persistence** | Browser localStorage |

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    UC4 SysEx Editor                     │
├─────────────────────────────────────────────────────────┤
│  UI Layer                                               │
│  ├── Header (file ops, undo/redo, guide link)          │
│  ├── Navigation (setup, groups, view toggle)           │
│  ├── Setup Manager (modal dialog)                      │
│  ├── Main Content (Focused or Overview)                │
│  │   ├── Quick Paste Toolbar (Overview only)           │
│  │   └── Tooltip System (Focused only)                 │
│  └── Status Bar (modified indicator, file info)        │
├─────────────────────────────────────────────────────────┤
│  State Management                                       │
│  ├── rawBuffer (Uint8Array, 100,640 bytes)             │
│  ├── sectionIndex (byte offset lookup)                 │
│  ├── undoStack / redoStack                             │
│  ├── clipboard / selection                             │
│  ├── quickPaste (mode, scope, source, multipliers)     │
│  ├── setupLabels (Map: index → label)                  │
│  ├── factoryTemplates (baseline for reset)             │
│  ├── conflicts (concurrent / mutuallyExclusive)        │
│  └── session (localStorage persistence)                │
├─────────────────────────────────────────────────────────┤
│  Data Layer                                             │
│  ├── getValue / setValue (SysEx byte access)           │
│  ├── getEncoderData / getFaderData / etc.              │
│  ├── captureSetupSnapshot / restoreSetupSnapshot       │
│  ├── exportJSON / importJSON                           │
│  └── detectConflicts                                   │
└─────────────────────────────────────────────────────────┘
```

---

## Data Model

### SysEx Structure

| Field | Value |
|-------|-------|
| Total size | 100,640 bytes |
| Setups | 18 |
| Groups per setup | 8 |
| Controls per group | 33 (8 enc + 8 push + 8 green + 8 fader + 1 fader9) |

### Control Types

| Type | Count | Parameters |
|------|-------|------------|
| **Encoder** | 8 per group | channel, cc, type, acc, display, min, max |
| **Push Button** | 8 per group | channel, typeNibble, note/cc, lower, upper, mode |
| **Green Button** | 8 per group | channel, typeNibble, note/cc, lower, upper, mode |
| **Fader** | 8 per group | channel, cc, type, min, max |
| **Fader 9** | 1 per group | channel, cc |

### Indexing Convention

| Concept | Internal | Display |
|---------|----------|---------|
| Setup | 0-17 | 1-18 |
| Group | 0-7 | 1-8 |
| Control | 0-7 | 1-8 |
| Channel | 0-15 (stored) | 1-16 (displayed) |

---

## Features

### 1. Dual View System

#### Focused View
Full parameter editing for selected groups with contextual tooltips.

**Section Order:**
1. Faders (fader group)
2. Green Buttons (fader group)
3. Encoders (encoder group)
4. Push Buttons (encoder group)
5. Fader 9 (fader group)

**Section Header Format:**
```
┌─ SECTION_NAME ─────────────────────── GrPN ─┐
```
Where `GrPN` is the 4-character group name in accent color.

**Tooltips:**
- Hover over parameter labels for explanations
- Hover over dropdown options for detailed descriptions
- 500ms delay before showing, 10s timeout

#### Overview Mode
8×8 grid display with tabs and Quick Paste toolbar.

**Tabs:**
- **All** (default) — All control types stacked vertically
- **Encoders** — 8×8 encoder grid
- **Push Buttons** — 8×8 push grid
- **Green Buttons** — 8×8 green grid
- **Faders** — 8×8 fader grid + fader9 row

**Cell Format:**
```
Ch:Type CC#
```
Examples: `1:CC 64`, `2:Nt 60`, `1:PB --`

**Interactions:**
| Action | Result |
|--------|--------|
| Single-click | Select cell (green outline) |
| Double-click | Jump to Focused view |
| Right-click | Context menu + select |
| Arrow keys | Move selection |
| Enter | Jump to Focused view |
| Q | Toggle Quick Paste |

---

### 2. Setup Manager

Modal dialog for managing entire setups.

#### Features

| Operation | Description |
|-----------|-------------|
| **Label** | Assign human-readable name to setup |
| **Clear** | Zero out all parameters in setup |
| **Copy** | Duplicate setup to other slot(s) |
| **Swap** | Exchange two setups |
| **Reset** | Restore setup to factory defaults |
| **Export** | Save single setup as JSON |
| **Import** | Load single setup JSON to slot |

#### Setup Grid
```
┌────────────────────────────────────────────┐
│  1: Label    2: Label    3            4    │
│  5           6           7            8    │
│  9           10          11           12   │
│  13          14          15           16   │
│  17: Ableton 18: Ableton                   │
└────────────────────────────────────────────┘
```

**Visual Indicators:**
- Modified setups: Accent color border
- Factory-matching setups: "(factory)" suffix
- Selected setups: Highlighted background

#### Selection
- Click to select single setup
- Ctrl+click for multi-select
- Click empty area to deselect

#### Label Storage
Labels stored in localStorage, not in SysEx. Key: `uc4-editor-setup-labels`.

#### Single Setup JSON Format
```json
{
  "format": "uc4-editor-setup",
  "version": "1.0",
  "exported": "2026-01-15T12:00:00.000Z",
  "sourceSetup": 0,
  "label": "Synth",
  "groups": [...]
}
```

---

### 3. Quick Copy/Paste System

Rapid copy/paste workflow for Overview mode.

#### State
```javascript
let quickPaste = {
    mode: 'off' | 'copy' | 'paste',
    scope: 'cell' | 'column' | 'row',
    source: null | { controlType, group, index, data, lockedScope, count },
    chMultiplier: 1,   // -8 to +8
    ccMultiplier: 0,   // -8 to +8
    pasteCount: 0
};
```

#### Toolbar
```
┌──────────────────────────────────────────────────────────────┐
│ Mode [Off][Copy][Paste]  Scope [Cell][Column][Row]           │
│ Source: Enc G1.1         Ch [+1▾]  CC [0▾]  [Clear Source]   │
│ Status: Click to paste • 3 pasted                            │
└──────────────────────────────────────────────────────────────┘
```

#### Offset Calculation
```javascript
offset = (targetGroup - sourceGroup) * multiplier
```

Example: Source G1, Target G3, Ch multiplier +1
- Group diff = 2
- Channel offset = 2 × 1 = +2

#### Visual Feedback
- Source cells: Green highlight
- Hover preview: Cyan outline
- Toast notifications on paste

#### Keyboard Shortcuts
| Key | Action |
|-----|--------|
| Q | Toggle Quick Paste mode |
| 1 | Cell scope |
| 2 | Column scope |
| 3 | Row scope |

---

### 4. Contextual Tooltips

Help text for all parameters in Focused view.

#### Implementation
```javascript
const TOOLTIPS = {
    'encoder.chan': 'MIDI channel (1-16). Controls which channel receives encoder messages.',
    'encoderType.CCr1': 'Relative mode 1. Sends 1 for clockwise, 127 for counter-clockwise.',
    // ... (see UC4_FOCUSED_TOOLTIPS_SPEC_v6.md for complete list)
};
```

#### Behavior
- Show delay: 500ms after hover
- Hide timeout: 10s
- Position: Above element when possible
- Keyboard accessible via `:focus`

#### Coverage
- All parameter labels (Chan, CC, Type, Acc, Disp, Min, Max, etc.)
- All dropdown options (CCr1, CCr2, CCAb, PrGC, CCAh, Pbnd, AFtt, etc.)

---

### 5. Conflict Detection

#### Classification

| Type | Key | Severity |
|------|-----|----------|
| **Concurrent** | Same message, active simultaneously | High — fix required |
| **Mutually-Exclusive** | Same message, different groups | Low — usually intentional |

#### Concurrent Conflicts
Controls in the SAME active domain that send identical messages:
- Encoder + Encoder (same encoder group)
- Push + Push (same encoder group)
- Fader + Fader (same fader group)
- Green + Green (same fader group)
- Encoder + Push (same encoder group)
- Fader + Green (same fader group)
- Any control + Fader9 (same fader group)

#### Mutually-Exclusive Conflicts
Same message type across different groups (only one group active at a time).

#### Conflict Key Format
```
Ch{channel} {type} {number}
```
Examples:
- `Ch1 CC 64`
- `Ch2 Note 60`
- `Ch1 PB` (pitch bend, no number)
- `Ch3 PC 0-127` (program change range)

#### UI Representation

**Filter Chips:**
```
[✓ Concurrent (3)] [□ Mutually-Exclusive (12)]
```

**Cell Highlighting:**
- Concurrent: Bright amber background
- Mutually-Exclusive: Dim amber background (when filter enabled)
- Both: Warning icon (⚠️) prefix

---

### 6. Copy/Paste Operations (Context Menu)

#### Clipboard Structure
```javascript
{
    type: 'control' | 'row' | 'column',
    controlType: 'encoder' | 'push' | 'green' | 'fader' | 'fader9',
    sourceSetup: number,
    sourceGroup: number,
    sourceIndex: number,
    data: object | object[]
}
```

#### Copy Scopes

| Scope | Description |
|-------|-------------|
| **Control** | Single cell, all parameters |
| **Row** | One control index across all 8 groups |
| **Column** | All controls in one group |

#### Paste Operations

**Simple Paste:** Direct copy of parameters to target.

**Paste Special Transforms:**
| Transform | Range | Description |
|-----------|-------|-------------|
| Channel offset | -15 to +15 | Shift MIDI channel |
| CC/Number offset | -127 to +127 | Shift CC or note number |
| Auto-increment | Any integer | Sequential values per target |
| Wrap mode | clamp / wrap | Out-of-range handling |

---

### 7. Undo/Redo System

#### Stack Structure
```javascript
{
    type: 'single' | 'batch' | 'setup-copy' | 'setup-swap' | 'setup-clear' | 'setup-reset' | 'setup-import',
    timestamp: number,
    description: string,
    // Type-specific fields...
}
```

#### Supported Operations
- Individual parameter changes
- Batch paste operations
- Setup copy/swap/clear/reset
- Setup import
- Label changes

#### Coalescing
Rapid edits to the same parameter (within 1 second) are merged into one undo step.

#### Limits
- Maximum stack size: 100 entries
- Oldest entries dropped when limit exceeded

#### Shortcuts
| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z | Cmd+Shift+Z |
| Redo (alt) | Ctrl+Y | — |

---

### 8. Session Persistence

#### Storage Key
```
uc4_editor_session
```

#### Stored Data
```javascript
{
    rawBuffer: base64-encoded Uint8Array,
    timestamp: ISO date string,
    currentSetup: number,
    encoderGroup: number,
    faderGroup: number,
    dirtyBanks: number[]  // Setups with unsaved changes
}
```

#### Auto-Save Trigger
- Debounced: 2 seconds after last edit
- On any data modification

#### Restore Flow
1. Page loads
2. Check localStorage for session
3. If exists and has dirty banks:
   - Show restore dialog
   - User chooses Restore or Discard
4. If restored: Load session state
5. If discarded or no session: Load factory defaults

#### Session Clear Triggers
- Export SysEx
- Export JSON
- Import new file
- User clicks Discard

---

### 9. Link Groups

#### Purpose
Synchronize encoder and fader group selectors for setups where groups map to channels.

#### State
```javascript
let linkGroups = false;  // Checkbox state
```

#### Behavior
When `linkGroups === true`:
- Clicking encoder group N → sets fader group to N
- Clicking fader group N → sets encoder group to N
- On enable: Syncs fader group to match encoder group

---

### 10. Keyboard Navigation

#### Overview Mode Shortcuts

| Key | Action |
|-----|--------|
| ↑↓←→ | Move selection |
| Enter | Jump to Focused view |
| Tab | Move right |
| Shift+Tab | Move left |
| Escape | Clear selection |
| Ctrl+C | Copy selected control |
| Ctrl+V | Paste to selected |
| Q | Toggle Quick Paste |

#### Selection State
```javascript
let selection = {
    mode: 'none' | 'single',
    controlType: string | null,
    group: number | null,
    index: number | null
};
```

---

## File Formats

### SysEx (.syx)

Binary format matching UC4 hardware dump.

| Offset | Size | Content |
|--------|------|---------|
| 0 | 100,640 | Raw SysEx data |

Validation: File size must equal 100,640 bytes.

### JSON (Full Export)

```json
{
  "version": 1,
  "exportDate": "2026-01-15T20:00:00.000Z",
  "setups": [
    {
      "index": 0,
      "groups": [
        {
          "index": 0,
          "name": "GrP1",
          "encoders": [...],
          "pushButtons": [...],
          "greenButtons": [...],
          "faders": [...],
          "fader9": { "channel": 1, "cc": 9 }
        }
      ]
    }
  ]
}
```

### JSON (Single Setup Export)

```json
{
  "format": "uc4-editor-setup",
  "version": "1.0",
  "exported": "2026-01-15T12:00:00.000Z",
  "sourceSetup": 0,
  "label": "Synth",
  "groups": [...]
}
```

---

## UI Components

### Header Bar
- Logo/title
- File operations: Import SysEx, Export SysEx, Import JSON, Export JSON
- Undo/Redo buttons
- Setup Manager button
- Guide link

### Navigation Bar
- Setup selector (dropdown, 1-18, with labels)
- Link groups checkbox
- Encoder group tabs (1-8) + group name display
- Fader group tabs (1-8) + group name display
- View toggle (Focused / Overview)

### Quick Paste Toolbar (Overview only)
- Mode buttons: Off / Copy / Paste
- Scope buttons: Cell / Column / Row
- Source display
- Channel multiplier dropdown
- CC multiplier dropdown
- Clear Source button
- Status text

### Status Bar
- Modified indicator (green/amber dot)
- File info (byte count)

### Toast Notifications
- Position: Top-right
- Auto-dismiss: 3 seconds
- Types: info, success, warning, error

### Context Menu
- Position: At cursor
- Dismiss: Click outside, Escape key
- Items: Copy Control, Copy Row, Copy Column, separator, Paste, Paste Special

### Setup Manager Dialog
- Modal overlay
- 18-setup grid with labels
- Selection highlight
- Action buttons: Label, Clear, Copy, Swap, Reset, Export, Import
- Close button

### Tooltips
- Position: Above target element
- Delay: 500ms
- Timeout: 10s
- Style: Dark background, light text, accent border

---

## CSS Variables

```css
:root {
    --bg-dark: #0a0a0c;
    --bg-panel: #111114;
    --bg-control: #18181c;
    --bg-input: #1e1e24;
    --border: #2a2a32;
    --border-light: #3a3a44;
    --text: #e0e0e8;
    --text-dim: #888898;
    --text-muted: #555560;
    --accent: #00d4aa;
    --accent-dim: #00a888;
    --warning: #ffaa00;
    --error: #ff4466;
    --encoder: #00aaff;
    --push: #aa66ff;
    --green-btn: #44dd66;
    --fader: #ff8844;
}
```

---

## Browser Requirements

| Feature | Requirement |
|---------|-------------|
| JavaScript | ES6+ |
| localStorage | Required for session persistence |
| File API | Required for import/export |
| Fetch API | Required for factory default loading |
| CSS Grid | Required for layouts |
| CSS Custom Properties | Required for theming |

**Tested Browsers:**
- Chrome 90+
- Firefox 90+
- Safari 15+
- Edge 90+

---

## Performance Considerations

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Conflict detection | O(n²) | Runs on setup change, cached |
| Overview render | O(n) | 64 cells per tab, 256 for All |
| Undo/redo | O(1) | Stack operations |
| JSON export | O(n) | Full traversal |
| Session save | O(1) | Debounced, async |
| Factory diff | O(n) | Cached per setup |

---

## Security

- No network requests except factory_default.syx fetch
- No cookies
- localStorage scoped to origin
- No eval() or dynamic code execution
- User files processed client-side only

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | — | Initial release |
| 2.0.0 | 2026-01-11 | Undo/redo, session persistence, overview mode, conflict detection, copy/paste, keyboard navigation, link groups, All tab |
| 2.1.0 | 2026-01-15 | Setup Manager (copy, swap, clear, reset, labels), Quick Copy/Paste system, contextual tooltips, single-setup export/import, factory reset |
