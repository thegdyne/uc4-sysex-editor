# UC4 Editor: Quick Copy/Paste Specification v2

## Overview

A persistent toolbar-based copy/paste workflow for the UC4 Editor's Overview page. Replaces the context-menu + dialog approach with a "configure once, click many" experience for rapidly building UC4 configurations.

### Design Goals

1. **Relational offsets**: Pasting from G1 to G4 automatically calculates offset based on group difference
2. **Set and forget**: Configure multiplier once, then click repeatedly to paste
3. **Always visible**: Toolbar shows current mode, source, and transform settings
4. **Visual feedback**: Source highlighted (amber), target preview on hover (teal)
5. **Keyboard acceleration**: Q to toggle, 1/2/3 for scope, Escape to exit
6. **Undo-friendly**: Each paste is a separate undo entry

---

## Core Concepts

### Relational Offset System

When pasting between groups, the offset is **calculated from the position difference**, not set as a fixed value.

**Formula:**
- Cell/Column paste: `offset = (targetGroup - sourceGroup) × multiplier`
- Row paste: `offset = (targetRow - sourceRow) × multiplier`

Both Ch and CC use the same formula with independent multipliers.

**Examples with multiplier = 1:**

| Copy From | Paste To | Diff | Offset Applied |
|-----------|----------|------|----------------|
| G1 | G1 | 0 | 0 |
| G1 | G4 | 3 | +3 |
| G5 | G2 | -3 | -3 |
| G1 | G8 | 7 | +7 |

**Examples with multiplier = 2:**

| Copy From | Paste To | Diff | Offset Applied |
|-----------|----------|------|----------------|
| G1 | G4 | 3 | +6 |
| G1 | G8 | 7 | +14 |

**Examples with multiplier = 0:**

All pastes are exact copies, no offset applied regardless of position.

**Concrete example:**
- G1 Fader 1 has Ch 1, CC 32
- Copy G1, paste to G4 with Ch multiplier = 1, CC multiplier = 1
- Result: Ch 1 + 3 = Ch 4, CC 32 + 3 = CC 35

### Scopes

| Scope | Copies | Pastes To |
|-------|--------|-----------|
| **Cell** | Single control | Single control |
| **Column** | All controls in group (8, or 1 for Fader9) | Target group |
| **Row** | Control at index across all 8 groups | Same index, all groups |

Scope is **locked at copy time**. Changing scope clears the source.

### Clamping

Values are clamped to valid ranges:
- Channel: 1-16
- CC: 0-127

No wrap option - values hit boundaries and stop.

---

## UI Components

### Quick Paste Toolbar

Located at top of Overview page. Collapses to single row when mode is Off.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Mode: [Off] [Copy] [Paste]  Scope: [Cell] [Column] [Row]                    │
│ Source: Fad Column G1 (8)   Ch: [0 ▾]   CC: [0 ▾]   [Clear Source]          │
│ Click to paste • 3 pasted                                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Components:**
- **Mode buttons**: Off / Copy / Paste
- **Scope buttons**: Cell / Column / Row
- **Source display**: Shows what was copied and scope
- **Ch multiplier**: Dropdown -8 to +8, default 0
- **CC multiplier**: Dropdown -8 to +8, default 0
- **Clear Source**: Clears copied source
- **Status line**: Contextual guidance and paste count

### Visual Feedback

| Element | Style | Meaning |
|---------|-------|---------|
| Source cell(s) | Amber outline + background | What you copied |
| Target cell(s) on hover | Teal dashed outline + background | Where you'll paste |
| Just pasted | Flash animation | Confirms paste |

Source highlighting respects scope:
- Cell: single cell highlighted
- Column: entire column highlighted
- Row: entire row highlighted

### Toolbar States

**Collapsed (Off mode):**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Mode: [Off] [Copy] [Paste]                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Expanded (Copy/Paste mode):**
Full toolbar with all controls visible.

---

## Interaction Model

### Mode + Click Behavior

| Mode | Single Click | Double Click | Right Click |
|------|--------------|--------------|-------------|
| **Off** | Select cell | Jump to Focused View | Context menu |
| **Copy** | Copy source (scope-aware), switch to Paste | Copy source | Context menu |
| **Paste** | Paste to target | Ignored (toast once) | Context menu |

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| **Q** | Toggle Quick Paste (Off → Copy → Paste → Off) |
| **1** | Set scope to Cell |
| **2** | Set scope to Column |
| **3** | Set scope to Row |
| **Escape** | Exit to Off mode, clear source |

### State Transitions

```
Off ──Q──► Copy ──click──► Paste ──Q──► Off
            │                  │
            └──────Q───────────┘
```

- Clicking a cell in Copy mode captures source and auto-switches to Paste
- Changing scope clears source and stays in current mode
- Switching to Off clears source
- Reset button clears source

---

## Implementation Details

### State Object

```javascript
const quickPaste = {
    mode: 'off',           // 'off' | 'copy' | 'paste'
    scope: 'cell',         // 'cell' | 'column' | 'row'
    source: null,          // { controlType, group, index, data, lockedScope, count }
    chMultiplier: 0,       // Channel offset multiplier (-8 to +8)
    ccMultiplier: 0,       // CC offset multiplier (-8 to +8)
    pasteCount: 0,         // Total pastes this session
    doubleClickToastShown: false
};
```

### Source Object Structure

```javascript
source = {
    controlType: 'encoder',  // 'encoder' | 'push' | 'green' | 'fader' | 'fader9'
    group: 0,                // Source group index (0-7)
    index: 0,                // Source control index (0-7, or 0 for fader9)
    data: {...},             // Copied data (single object or array for column/row)
    lockedScope: 'column',   // Scope at time of copy
    count: 8                 // Number of controls copied
};
```

### Paste Logic

```javascript
function performQuickPaste(controlType, group, index) {
    // Same-type enforcement
    if (controlType !== quickPaste.source.controlType) {
        showToast("Can't paste encoder to fader", 'error');
        return;
    }
    
    const scope = quickPaste.source.lockedScope;
    
    if (scope === 'cell' || scope === 'column') {
        // Group-based offset
        const groupDiff = group - quickPaste.source.group;
        const chOffset = groupDiff * quickPaste.chMultiplier;
        const ccOffset = groupDiff * quickPaste.ccMultiplier;
        // Apply to data...
    } else if (scope === 'row') {
        // Row-based offset
        const rowDiff = index - quickPaste.source.index;
        const chOffset = rowDiff * quickPaste.chMultiplier;
        const ccOffset = rowDiff * quickPaste.ccMultiplier;
        // Apply to data...
    }
}
```

### Transform Application

```javascript
function transformControlData(data, options) {
    let newData = { ...data };
    
    // Apply channel offset (1-16 range)
    if (options.channelOffset !== 0) {
        newData.channel = clamp(newData.channel + options.channelOffset, 1, 16);
    }
    
    // Apply CC offset (0-127 range)
    if (options.numberOffset !== 0 && newData.cc !== undefined) {
        newData.cc = clamp(newData.cc + options.numberOffset, 0, 127);
    }
    
    return newData;
}
```

---

## Supported Control Types

| Type | Column Size | Notes |
|------|-------------|-------|
| Encoder | 8 | Full support |
| Push Button | 8 | Full support |
| Green Button | 8 | Full support |
| Fader 1-8 | 8 | Full support |
| Fader 9 | 1 | Column = single cell, separate table |

Same-type enforcement: can only paste encoder→encoder, fader→fader, etc.

---

## Reset Behavior

The Reset button (⟲) in the toolbar:
- Restores all data to originally imported SysEx
- Clears undo/redo history
- Clears Quick Paste source
- Shows confirmation dialog first

---

## Section Order

Both Focused and Overview views use consistent ordering:

1. Faders 1-8
2. Fader 9
3. Green Buttons
4. Encoders
5. Push Buttons

---

## Typical Workflows

### Workflow 1: Setup 8 Groups with Incrementing Channels

1. Configure G1 faders in Focused View (Ch 1, CC 32-39)
2. Switch to Overview, press Q (Copy mode)
3. Set scope to Column, Ch multiplier to +1, CC to 0
4. Click G1 faders (copies column, switches to Paste)
5. Click G2, G3, G4, G5, G6, G7, G8
6. Result: G2=Ch2, G3=Ch3, ... G8=Ch8, all with same CCs
7. Press Escape to exit

### Workflow 2: Copy Row with CC Offset

1. Configure Row 1 across all groups
2. Press Q, set scope to Row, CC multiplier to +1
3. Click any cell in Row 1 (copies row)
4. Click Row 2, Row 3, etc.
5. Result: Each row has CCs offset by row difference

### Workflow 3: Exact Copy (No Offset)

1. Leave both multipliers at 0 (default)
2. Copy any cell/column/row
3. Paste anywhere - exact duplicate regardless of position

### Workflow 4: Large CC Jumps Between Groups

For setups where each group uses a different CC range (e.g., G1=0-7, G2=8-15, G3=16-23):

1. Configure G1 with CC 0-7
2. Set CC multiplier to +8
3. Copy G1 column, paste to G2, G3, etc.
4. Result: G2=CC 8-15, G3=CC 16-23, etc.

---

## Future Considerations

These were considered but not implemented:

- **Batch mode**: Accumulate pastes into single undo. Deemed unnecessary - individual undos work fine.
- **Tab boundary enforcement**: Same-type already prevents cross-type paste.
- **Wrap warning**: Values clamp visibly. User sees result immediately.
- **Auto-increment counters**: Original spec had counters that increment between pastes. Replaced by simpler relational offset system.

The relational offset system covers most use cases more intuitively than fixed offsets with auto-increment counters.
