# UC4 Factory Defaults Implementation

**Version:** 1.1  
**Status:** IMPLEMENTED  
**Date:** 2026-01-15

---

## Overview

This document describes the implemented factory defaults architecture for the UC4 Editor's Setup Manager, including UI refinements and bug fixes added during implementation.

---

## Core Implementation

### Per-Setup Factory Templates

Factory defaults are stored as 18 separate snapshots extracted from `factory_default.syx` at initialization.

```javascript
let FACTORY_TEMPLATES = null; // Array(18) of setup snapshots
```

#### Initialization

Called during SysEx load:

```javascript
function initFactoryTemplates() {
    FACTORY_TEMPLATES = [];
    for (let setupIdx = 0; setupIdx < 18; setupIdx++) {
        FACTORY_TEMPLATES[setupIdx] = captureSetupSnapshot(setupIdx);
    }
    console.log('Factory templates initialized for all 18 setups');
}
```

#### Factory Snapshot Creation

```javascript
function createFactorySetupSnapshot(setupIdx) {
    if (!FACTORY_TEMPLATES || !FACTORY_TEMPLATES[setupIdx]) {
        throw new Error(`Factory template not available for setup ${setupIdx + 1}`);
    }
    return JSON.parse(JSON.stringify(FACTORY_TEMPLATES[setupIdx]));
}
```

#### Difference Detection

```javascript
let differsFromFactoryCache = new Map();

function differsFromFactory(setupIdx) {
    if (!FACTORY_TEMPLATES) return false;
    
    if (differsFromFactoryCache.has(setupIdx)) {
        return differsFromFactoryCache.get(setupIdx);
    }
    
    const current = captureSetupSnapshot(setupIdx);
    const factory = FACTORY_TEMPLATES[setupIdx];
    const differs = JSON.stringify(current) !== JSON.stringify(factory);
    
    differsFromFactoryCache.set(setupIdx, differs);
    return differs;
}

function invalidateDiffCache(indices) {
    for (const idx of indices) {
        differsFromFactoryCache.delete(idx);
    }
}
```

---

## Factory Data Patterns

### Channel Assignment

| Setups | MIDI Channel |
|--------|--------------|
| 1-16 | Channel = Setup Number |
| 17 | Channel 13 (Ableton) |
| 18 | Channel 14 (Ableton) |

### Encoder CC Numbers (Generic Template, Setups 1-16)

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

### Ableton Template (Setups 17-18)

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

---

## UI Implementation

### Terminology

| Element | Text |
|---------|------|
| Button | "Reset to Factory" |
| Tooltip | "Restore selected setups to factory defaults" |
| Dialog title | "Reset to Factory" |
| Primary action | "Reset to Factory" |
| Toast (single) | "Reset 1 setup to factory defaults" |
| Toast (multiple) | "Reset N setups to factory defaults" |
| Undo description | "Reset Setup(s) X to factory" |

### Selection Behavior

| Selection Count | Button State | Footer Text |
|-----------------|--------------|-------------|
| 0 | Disabled | "Select one or more setups" |
| 1+ | Enabled | "Selected: Setup X" or "Selected: Setups X, Y, Z" |

### Dialog Features

- **Select All link**: Footer contains "Select All" link that selects all 18 setups and refreshes dialog
- **Setup list**: Shows comma-separated list of affected setups with labels
- **Undo notice**: "This action can be undone."

### Keyboard Shortcut

- **Delete key**: Opens Reset to Factory dialog (when 1+ setups selected)

### Double-Click Label Edit

- Double-clicking a setup card's label (including "(no label)") opens the label editor
- Label element has `cursor: pointer` and hover highlight

---

## State Mutation Hook

### afterSetupStateMutation(affectedSetupIdxs)

Unified post-mutation hook ensuring consistent UI updates:

```javascript
function afterSetupStateMutation(affectedSetupIdxs) {
    updateSetupDropdownLabels();
    refreshMainEditorIfNeeded(affectedSetupIdxs);
    if (setupManagerOpen) {
        renderSetupManager();
    }
}
```

### Call Sites

| Operation | Affected Indices |
|-----------|------------------|
| SysEx Import | All 18 (0-17) |
| Main Reset | All 18 (0-17) |
| Reset to Factory | Selected indices |
| Copy | Destination indices |
| Swap | Both indices |
| Label Edit | Single index |
| Undo/Redo | Action-specific indices |

---

## Undo Integration

### Action Type

```javascript
{
    type: 'setup-reset-factory',
    description: `Reset Setup(s) X to factory`,
    indices: [...],
    beforeSnapshots: { idx: snapshot, ... },
    afterSnapshots: { idx: snapshot, ... },
    labelsBefore: { idx: label|null, ... },
    labelsAfter: { idx: null, ... }
}
```

### Undo Handler

```javascript
case 'setup-reset-factory': {
    const snapshots = reverse ? action.beforeSnapshots : action.afterSnapshots;
    const labels = reverse ? action.labelsBefore : action.labelsAfter;
    for (const idx of action.indices) {
        restoreSetupSnapshot(idx, snapshots[idx]);
        if (labels[idx]) {
            setupLabels.set(idx, labels[idx]);
        } else {
            setupLabels.delete(idx);
        }
    }
    invalidateDiffCache(action.indices);
    afterSetupStateMutation(action.indices);
    break;
}
```

---

## Bug Fixes

### Issue: Setup dropdown not refreshing after Reset/Import

**Cause:** `setupLabels` not cleared, `updateSetupDropdownLabels()` not called.

**Fix:** Clear `setupLabels` and `differsFromFactoryCache` on Reset/Import, call `afterSetupStateMutation()`.

### Issue: Setup Manager not refreshing after main window Reset

**Cause:** `renderSetupManager()` not called after Reset.

**Fix:** `afterSetupStateMutation()` checks `setupManagerOpen` and re-renders if true.

---

## Test Results

| Test ID | Description | Result |
|---------|-------------|--------|
| DF-CHAN | Load factory_default.syx, all differsFromFactory = false | âœ“ Pass |
| DF-EDIT | Edit any control, differsFromFactory = true | âœ“ Pass |
| CL-SETUPCHAN | Clear Setup 5, all controls have channel = 5 | âœ“ Pass |
| CL-ABLETON17 | Clear Setup 17, encoder[0].cc = 0, channel = 13 | âœ“ Pass |
| CL-ABLETON18 | Clear Setup 18, channel = 14 | âœ“ Pass |
| CL-UNDO | Undo after Clear restores previous state | âœ“ Pass |
| UI-DROPDOWN | Reset/Import refreshes dropdown labels | âœ“ Pass |
| UI-MANAGER | Reset refreshes Setup Manager if open | âœ“ Pass |
| UI-DBLCLICK | Double-click label opens editor | âœ“ Pass |

---

## Files Modified

- `index.html` - All changes in single file

### Key Functions Added/Modified

| Function | Status |
|----------|--------|
| `initFactoryTemplates()` | Added |
| `createFactorySetupSnapshot()` | Modified |
| `differsFromFactory()` | Modified |
| `invalidateDiffCache()` | Added |
| `afterSetupStateMutation()` | Added |
| `showResetToFactoryDialog()` | Renamed from showClearDialog |
| `closeResetToFactoryDialog()` | Renamed from closeClearDialog |
| `performResetToFactory()` | Renamed from performClear |
| `selectAllSetups()` | Added |

### Removed

- `FACTORY_DEFAULTS` object (parametric functions)

---

## Summary

| Aspect | Implementation |
|--------|----------------|
| Factory data source | 18 snapshots from factory_default.syx |
| Channel handling | Per-setup (1-16 match setup#, 17=ch13, 18=ch14) |
| UI terminology | "Reset to Factory" |
| Selection UX | Disabled when 0 selected, "Select All" link |
| State sync | Unified `afterSetupStateMutation()` hook |
| Label editing | Double-click to edit |
