# UC4 SysEx Editor — Technical Specification

Version: 2.0.0  
Date: 2026-01-11  
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
| **File formats** | SysEx (.syx), JSON |
| **Persistence** | Browser localStorage |

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    UC4 SysEx Editor                     │
├─────────────────────────────────────────────────────────┤
│  UI Layer                                               │
│  ├── Header (file ops, undo/redo, guide link)          │
│  ├── Navigation (setup, groups, view toggle)           │
│  ├── Main Content (Focused or Overview)                │
│  └── Status Bar (modified indicator, file info)        │
├─────────────────────────────────────────────────────────┤
│  State Management                                       │
│  ├── rawBuffer (Uint8Array, 100,640 bytes)             │
│  ├── sectionIndex (byte offset lookup)                 │
│  ├── undoStack / redoStack                             │
│  ├── clipboard / selection                             │
│  ├── conflicts (concurrent / mutuallyExclusive)        │
│  └── session (localStorage persistence)                │
├─────────────────────────────────────────────────────────┤
│  Data Layer                                             │
│  ├── getValue / setValue (SysEx byte access)           │
│  ├── getEncoderData / getFaderData / etc.              │
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
Full parameter editing for selected groups.

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

#### Overview Mode
8×8 grid display with tabs.

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

---

### 2. Conflict Detection

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

**Conflict Panel:**
```
⚠ Conflicts:
⚠ Ch1 CC 64: Enc G1.1 (CC), Fad G1.1 (CC)
⚠ Ch2 Note 60: Push G1.3 (Note), Green G1.5 (Note)
```

---

### 3. Copy/Paste Operations

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

**Transform Logic:**
```javascript
// Channel (1-based in UI, stored as 0-15)
newChannel = applyOffset(channel, offset, 1, 16, wrapMode);

// CC/Note (0-127)
newCC = applyOffset(cc, offset + (autoInc * index), 0, 127, wrapMode);
```

#### Context Menu

```
┌─────────────────────────────────┐
│ Copy Control                    │
│ Copy Row (3 × 8 groups)         │
│ Copy Column (Group 2)           │
├─────────────────────────────────┤
│ Paste                           │  ← Disabled if incompatible
│ Paste Special...                │
└─────────────────────────────────┘
```

---

### 4. Undo/Redo System

#### Stack Structure
```javascript
{
    type: 'single' | 'batch',
    timestamp: number,
    // For single:
    param: string,
    controlType: string,
    setup: number,
    group: number,
    index: number,
    oldValue: any,
    newValue: any,
    // For batch:
    operations: array
}
```

#### Coalescing
Rapid edits to the same parameter (within 1 second) are merged into one undo step.

#### Batch Operations
Paste-to-row and paste-to-column create single batch undo entries.

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

### 5. Session Persistence

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
    faderGroup: number
}
```

#### Auto-Save Trigger
- Debounced: 2 seconds after last edit
- On any data modification

#### Restore Flow
1. Page loads
2. Check localStorage for session
3. If exists and rawBuffer valid:
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

### 6. Link Groups

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

### 7. Keyboard Navigation

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

#### Selection State
```javascript
let selection = {
    mode: 'none' | 'single',
    controlType: string | null,
    group: number | null,
    index: number | null
};
```

#### Navigation in "All" Tab
Arrow up/down at section boundaries moves to adjacent section:
- Down from Encoder row 8 → Push row 1
- Up from Push row 1 → Encoder row 8
- etc.

---

## File Formats

### SysEx (.syx)

Binary format matching UC4 hardware dump.

| Offset | Size | Content |
|--------|------|---------|
| 0 | 100,640 | Raw SysEx data |

Validation: File size must equal 100,640 bytes.

### JSON

```json
{
  "version": 1,
  "exportDate": "2026-01-11T20:00:00.000Z",
  "setups": [
    {
      "index": 0,
      "groups": [
        {
          "index": 0,
          "name": "GrP1",
          "encoders": [
            {
              "channel": 1,
              "type": 2,
              "cc": 1,
              "min": 0,
              "max": 127,
              "acc": 1,
              "display": 1
            }
          ],
          "pushButtons": [
            {
              "channel": 1,
              "typeNibble": 144,
              "note": 36,
              "lower": 0,
              "upper": 127,
              "mode": 0,
              "display": 1
            }
          ],
          "greenButtons": [...],
          "faders": [
            {
              "channel": 1,
              "type": 2,
              "cc": 1,
              "min": 0,
              "max": 127
            }
          ],
          "fader9": {
            "channel": 1,
            "cc": 9
          }
        }
      ]
    }
  ]
}
```

---

## UI Components

### Header Bar
- Logo/title
- File operations: Import SysEx, Export SysEx, Import JSON, Export JSON
- Undo/Redo buttons
- Guide link

### Navigation Bar
- Setup selector (dropdown, 1-18)
- Link groups checkbox
- Encoder group tabs (1-8) + group name display
- Fader group tabs (1-8) + group name display
- View toggle (Focused / Overview)

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

### Paste Special Dialog
- Modal overlay
- Sections: Source info, Paste target, Transforms
- Buttons: Cancel, Paste

---

## CSS Variables

```css
:root {
    --bg-dark: #0a0a0f;
    --bg-panel: #12121a;
    --bg-card: #1a1a24;
    --bg-input: #0d0d12;
    --bg-control: #15151f;
    --border: #2a2a3a;
    --text: #e8e8e8;
    --text-dim: #888;
    --text-muted: #666;
    --accent: #00ffaa;
    --encoder: #00d4ff;
    --push: #ff6b9d;
    --green-btn: #7fff00;
    --fader: #ffaa00;
    --warning: #ffaa00;
    --conflict-concurrent: rgba(255, 170, 0, 0.3);
    --conflict-me: rgba(255, 170, 0, 0.15);
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
