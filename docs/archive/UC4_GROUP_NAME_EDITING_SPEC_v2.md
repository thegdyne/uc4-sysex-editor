# UC4 Editor: Group Name Editing Specification

**Version:** 2.2 (Final)  
**Date:** 2026-01-14  
**Status:** Ready for Implementation

## Overview

Add the ability to view and edit the 4-character group names for each of the 8 groups within each setup. Group names are displayed on the UC4's 7-segment display when switching groups and help users identify what each group controls.

### Goals
1. **Fix bug:** Group names currently read from setup 0 only; should be per-setup
2. **Display correctly:** Show decoded 4-byte name for current setup (not defaults)
3. **Enable editing:** Allow users to change group names in the editor
4. **Lossless round-trip:** Preserve exact byte values through edit/undo/export cycles

---

## Current State Analysis

### What Works
- `getGroupName(setupIdx, group)` function exists (line 2032)
- `decode7Seg(val)` converts stored values to display characters (line 2056)
- Group names are displayed in the UI (nav bar, Overview headers, Focused view)
- `computeActualLocation()` correctly handles the complex group name storage pattern (line 1829)
- `getValue()` and `setValue()` work correctly with the addressing scheme

### Bug #1: Wrong Setup Reference (CRITICAL)

**Location:** `index.html` line 2039

**Current code:**
```javascript
function getGroupName(setupIdx, group) {
    // Group names are stored globally (same for all setups), always read from setup 0
    const baseIdx = group * 4;
    let name = '';
    let hasValidData = true;
    
    for (let i = 0; i < 4; i++) {
        const val = getValue(0, 0x14, 0x80, baseIdx + i);  // â† BUG: Hardcoded 0
        // ...
    }
}
```

**Problem:** UC4 manual states *"you can edit the group names in edit mode for each setup separately"* â€” group names are **per-setup**, not global.

**Impact:** 
- All 18 setups display the same group names (from setup 0)
- JSON export exports incorrect names for setups 1-17
- Setup 17/18 Ableton presets (Snd1, trAC, RAC, PAn, GLOb) don't display correctly

### Bug #2: Fallback Logic Masks Real Data

**Current code:**
```javascript
if (!hasValidData || name.includes('?')) {
    return `GrP${group + 1}`;
}
```

**Problem:** This silently replaces custom names with defaults whenever:
- Read partially fails
- Name contains unknown character codes (decoded as `?`)

This masks real data and hides mapping incompleteness from the user.

### Bug #3: JSON Import Ignores Group Names

**Location:** `index.html` lines 3473-3483

Group names are exported but never imported back.

### Missing Feature: No `setGroupName()` Function

There is no function to write group names back to the SysEx buffer.

---

## Requirements

### R1: Fix Per-Setup Reading
Group names must be read from the current setup, not setup 0.

### R2: Display Decoded Names (No Silent Fallback)
Display the **decoded 4-byte name for the current setup**. Do not substitute defaults unless the read completely fails. If undecodable bytes exist, display placeholder glyphs and warn the user.

### R3: Enable Name Editing
Users must be able to edit group names directly in the editor.

### R4: Character Validation
Only allow characters from the canonical glyph set that can be displayed on the UC4's 7-segment display.

### R5: Undo/Redo Support
Group name changes must integrate with the existing undo/redo system.

### R6: SysEx Compatibility
Edited names must be written back to the correct SysEx locations with proper checksum recalculation.

### R7: Lossless Name Storage (NEW)
Editing, undo/redo, JSON import/export must preserve the exact underlying 4-byte values (except when user explicitly changes the name). Undo state stores raw byte values, not rendered strings.

### R8: Single Source of Truth Mapping (NEW)
A single mapping table defines `valueâ†”glyph`. Both encode and decode derive from it to avoid drift. `GLYPH_TABLE` is conceptually a **32-element array indexed 0-31** with no gaps.

### R9: Non-Canonical Byte Preservation (NEW)
Non-canonical bytes (values 32-255) are preserved in the buffer unless the user explicitly changes the name. Opening and closing the editor without changes must not modify any bytes.

### R10: No-Op Save Rule (NEW)
Save is a no-op (no buffer write, no undo action) when the resulting bytes would be identical to the existing bytes. Comparison is always **byte-based**, not string-based.

---

## SysEx Storage Format

### Location (Logical)
| Parameter | Value | Notes |
|-----------|-------|-------|
| Section | 0x14 | GROUP_NAMES section (logical) |
| Bank | 0x80 | GROUP_NAMES bank (logical) |
| Values | 32 | 4 characters Ã— 8 groups per setup |
| Per-setup | Yes | Each of 18 setups has its own names |

### Physical Storage Pattern

Group names use a **packed storage scheme**. The existing `computeActualLocation()` function (line 1829) handles this:

```javascript
} else if (secId === 0x14 && bank === 0x80) {
    // Group Names: 2 setups per bank, 4 setups per section
    actualSecId = 0x14 + Math.floor(setupIdx / 4);
    const bankOffset = Math.floor((setupIdx % 4) / 2);
    actualBank = 0x80 + bankOffset * 0x40;
    const valueOffset = (setupIdx % 2) * 32;
    actualValueIdx = valueIdx + valueOffset;
}
```

**Physical layout by setup:**

| Setup | Actual Section | Actual Bank | Value Offset |
|-------|----------------|-------------|--------------|
| 0 | 0x14 | 0x80 | 0 |
| 1 | 0x14 | 0x80 | 32 |
| 2 | 0x14 | 0xC0 | 0 |
| 3 | 0x14 | 0xC0 | 32 |
| 4 | 0x15 | 0x80 | 0 |
| ... | ... | ... | ... |
| 16 | 0x18 | 0x80 | 0 |
| 17 | 0x18 | 0x80 | 32 |

### Verification Step (Pre-Implementation)

Before implementing, verify with hardware:
1. Create two setups with different group names on UC4
2. Dump SysEx
3. Confirm bytes differ at the expected offsets for section 0x14 bank 0x80
4. This validates that `getValue(setupIdx, ...)` addresses correctly

### Data Layout (per setup)
```
Index 0-3:   Group 1 name (4 bytes)
Index 4-7:   Group 2 name (4 bytes)
Index 8-11:  Group 3 name (4 bytes)
Index 12-15: Group 4 name (4 bytes)
Index 16-19: Group 5 name (4 bytes)
Index 20-23: Group 6 name (4 bytes)
Index 24-27: Group 7 name (4 bytes)
Index 28-31: Group 8 name (4 bytes)
```

---

## Character Mapping (Single Source of Truth)

### Canonical Glyph Table

This is the **single source of truth** for both encode and decode operations:

| Value | Glyph | Notes |
|-------|-------|-------|
| 0 | `0` | |
| 1 | `1` | |
| 2 | `2` | |
| 3 | `3` | |
| 4 | `4` | |
| 5 | `5` | |
| 6 | `6` | |
| 7 | `7` | |
| 8 | `8` | |
| 9 | `9` | |
| 10 | `A` | |
| 11 | `b` | Lowercase (7-seg limitation) |
| 12 | `C` | |
| 13 | `d` | Lowercase (7-seg limitation) |
| 14 | `E` | |
| 15 | `F` | |
| 16 | `G` | |
| 17 | `H` | |
| 18 | `I` | |
| 19 | `J` | |
| 20 | `L` | |
| 21 | `n` | Lowercase (7-seg limitation) |
| 22 | `O` | |
| 23 | `t` | Lowercase (7-seg limitation) |
| 24 | `P` | |
| 25 | `S` | |
| 26 | `r` | Lowercase (7-seg limitation) |
| 27 | `U` | |
| 28 | `Y` | |
| 29 | `-` | Hyphen |
| 30 | `_` | Underscore |
| 31 | ` ` | Space (blank segment) |

**Canonical glyph string:** `0123456789AbCdEFGHIJLnOtPSrUY-_ `

### Input Normalization Rules

User input is normalized to canonical glyphs:

| User Types | Stored As | Rationale |
|------------|-----------|-----------|
| `B` or `b` | `b` (11) | 7-seg shows lowercase |
| `D` or `d` | `d` (13) | 7-seg shows lowercase |
| `N` or `n` | `n` (21) | 7-seg shows lowercase |
| `T` or `t` | `t` (23) | 7-seg shows lowercase |
| `R` or `r` | `r` (26) | 7-seg shows lowercase |
| `a` | `A` (10) | 7-seg shows uppercase |
| `c` | `C` (12) | 7-seg shows uppercase |
| `e` | `E` (14) | 7-seg shows uppercase |
| `f` | `F` (15) | 7-seg shows uppercase |
| `g` | `G` (16) | 7-seg shows uppercase |
| `h` | `H` (17) | 7-seg shows uppercase |
| `i` | `I` (18) | 7-seg shows uppercase |
| `j` | `J` (19) | 7-seg shows uppercase |
| `l` | `L` (20) | 7-seg shows uppercase |
| `o` | `O` (22) | 7-seg shows uppercase |
| `p` | `P` (24) | 7-seg shows uppercase |
| `s` | `S` (25) | 7-seg shows uppercase |
| `u` | `U` (27) | 7-seg shows uppercase |
| `y` | `Y` (28) | 7-seg shows uppercase |
| Other | **REJECT** | Invalid character |

### Non-Canonical Byte Handling

When decoding bytes from SysEx:

| Byte Value | Decode Result | UI Behavior |
|------------|---------------|-------------|
| 0-31 | Canonical glyph | Normal display |
| 32-126 | ASCII character | Display with warning badge |
| 127+ or null | `ï¿½` (replacement char) | Display with warning badge |

**Key principles:** 
- Never silently replace with defaults. Show what's there + warn if unexpected.
- Warning text: "Contains non-standard codes; editor will preserve them unless you overwrite the name."
- Non-canonical bytes are **preserved losslessly** unless user explicitly saves a new name.

### Factory Defaults
```
Group 1: GrP1 â†’ [16, 26, 24, 1]
Group 2: GrP2 â†’ [16, 26, 24, 2]
...
Group 8: GrP8 â†’ [16, 26, 24, 8]
```

---

## Storage Rules (Hard Rules)

### Rule 1: Always 4 Bytes
Storage is always exactly 4 bytes. No exceptions.

### Rule 2: UI Normalization
- UI accepts 0-4 typed characters
- On save, pad with spaces (value 31) to exactly 4 characters
- Truncate if longer than 4 characters

### Rule 3: Empty Name = 4 Spaces
- An "empty" name is stored as `[31, 31, 31, 31]` (4 spaces)
- UI displays empty names as `----` (visual indicator of blank)
- JSON exports as `"    "` (4 spaces)

### Rule 4: JSON Format
- JSON `name` field is always exactly 4 characters (space-padded)
- Uses canonical glyph charset only
- **Only ASCII space (0x20) is valid whitespace** â€” reject `\t`, `\n`, non-breaking space, etc.
- **Reject any character with codepoint > 127** â€” includes NBSP (`\u00A0`), all Unicode whitespace, emoji, etc.
- Import rejects invalid characters with detailed error message (see below)

### Rule 5: Encoding Totality
After input normalization, `encodeGlyph()` must succeed for all canonical glyphs. If encoding fails for a normalized glyph, treat as internal error and block save.

---

## Implementation Design

### Phase 1: Fix Reading (Bug Fix)

**Change `getGroupName()` to return raw values + decoded string:**

```javascript
function getGroupNameData(setupIdx, group) {
    const baseIdx = group * 4;
    const values = [];
    const chars = [];
    let hasNonCanonical = false;
    
    for (let i = 0; i < 4; i++) {
        const val = getValue(setupIdx, SECTIONS.GROUP_NAMES, BANKS.GROUP_NAMES, baseIdx + i);
        
        if (val === null) {
            // Read failed - return null to indicate failure
            return null;
        }
        
        values.push(val);
        
        if (val >= 0 && val <= 31) {
            chars.push(GLYPH_TABLE[val]);
        } else if (val >= 32 && val < 127) {
            chars.push(String.fromCharCode(val));
            hasNonCanonical = true;  // Valid ASCII but not in our expected range
        } else {
            chars.push('ï¿½');
            hasNonCanonical = true;
        }
    }
    
    return {
        values: values,           // Raw bytes [v0, v1, v2, v3]
        display: chars.join(''),  // Decoded string for display
        hasNonCanonical: hasNonCanonical    // True if any bytes outside 0-31 range
    };
}

// Convenience wrapper for display-only contexts
function getGroupName(setupIdx, group) {
    const data = getGroupNameData(setupIdx, group);
    if (data === null) {
        return `GrP${group + 1}`;  // Only fallback on complete read failure
    }
    return data.display;
}
```

**Update comment (line 2033):**
```javascript
// Group names are stored PER-SETUP in section 0x14 bank 0x80
```

### Phase 2: Single Source of Truth Mapping

**Define canonical table and derive both directions:**

```javascript
// Single source of truth - value to glyph
// Conceptually a 32-element array indexed 0-31 with NO GAPS
// Implementation uses object for clarity but must be total for 0-31
const GLYPH_TABLE = {
    0: '0', 1: '1', 2: '2', 3: '3', 4: '4', 5: '5', 6: '6', 7: '7', 8: '8', 9: '9',
    10: 'A', 11: 'b', 12: 'C', 13: 'd', 14: 'E', 15: 'F', 16: 'G', 17: 'H',
    18: 'I', 19: 'J', 20: 'L', 21: 'n', 22: 'O', 23: 't', 24: 'P', 25: 'S',
    26: 'r', 27: 'U', 28: 'Y', 29: '-', 30: '_', 31: ' '
};

// Invariant check (run once at startup)
for (let i = 0; i < 32; i++) {
    if (!(i in GLYPH_TABLE)) {
        throw new Error(`GLYPH_TABLE missing entry for value ${i}`);
    }
}

// Derived: glyph to value (built from GLYPH_TABLE)
const VALUE_TABLE = {};
for (const [val, glyph] of Object.entries(GLYPH_TABLE)) {
    VALUE_TABLE[glyph] = parseInt(val);
}

// Input normalization: user input -> canonical glyph
const INPUT_NORMALIZE = {
    // Lowercase that map to lowercase glyphs (7-seg limitation)
    'B': 'b', 'b': 'b',
    'D': 'd', 'd': 'd',
    'N': 'n', 'n': 'n',
    'T': 't', 't': 't',
    'R': 'r', 'r': 'r',
    // Lowercase that map to uppercase glyphs
    'a': 'A', 'A': 'A',
    'c': 'C', 'C': 'C',
    'e': 'E', 'E': 'E',
    'f': 'F', 'F': 'F',
    'g': 'G', 'G': 'G',
    'h': 'H', 'H': 'H',
    'i': 'I', 'I': 'I',
    'j': 'J', 'J': 'J',
    'l': 'L', 'L': 'L',
    'o': 'O', 'O': 'O',
    'p': 'P', 'P': 'P',
    's': 'S', 'S': 'S',
    'u': 'U', 'U': 'U',
    'y': 'Y', 'Y': 'Y',
    // Direct mappings
    '0': '0', '1': '1', '2': '2', '3': '3', '4': '4',
    '5': '5', '6': '6', '7': '7', '8': '8', '9': '9',
    '-': '-', '_': '_', ' ': ' '
};

function normalizeInputChar(char) {
    return INPUT_NORMALIZE[char] ?? null;  // null = invalid
}

function encodeGlyph(glyph) {
    return VALUE_TABLE[glyph] ?? null;  // null = not in table
}

function decodeValue(val) {
    if (val >= 0 && val <= 31) {
        return { glyph: GLYPH_TABLE[val], canonical: true };
    } else if (val >= 32 && val < 127) {
        return { glyph: String.fromCharCode(val), canonical: false };
    } else {
        return { glyph: 'ï¿½', canonical: false };
    }
}
```

### Phase 3: Add Writing Function

```javascript
function setGroupName(setupIdx, group, values) {
    // values must be array of exactly 4 byte values
    if (!Array.isArray(values) || values.length !== 4) {
        console.error('Group name must be exactly 4 byte values');
        return false;
    }
    
    const baseIdx = group * 4;
    
    for (let i = 0; i < 4; i++) {
        const val = values[i];
        if (typeof val !== 'number' || val < 0 || val > 255) {
            console.error(`Invalid byte value ${val} at position ${i}`);
            return false;
        }
        setValue(setupIdx, SECTIONS.GROUP_NAMES, BANKS.GROUP_NAMES, baseIdx + i, val);
    }
    
    return true;
}

// Helper: convert user input string to byte values
function stringToGroupNameValues(input) {
    // Normalize and pad to 4 characters
    let normalized = '';
    for (const char of input.substring(0, 4)) {
        // Reject any character with codepoint > 127 (includes Unicode whitespace, NBSP, etc.)
        if (char.charCodeAt(0) > 127) {
            return { error: `Invalid character: ${char} (non-ASCII)` };
        }
        const norm = normalizeInputChar(char);
        if (norm === null) {
            return { error: `Invalid character: ${char}` };
        }
        normalized += norm;
    }
    
    // Pad with spaces
    while (normalized.length < 4) {
        normalized += ' ';
    }
    
    // Convert to values (Rule 5: Encoding Totality)
    const values = [];
    for (const glyph of normalized) {
        const val = encodeGlyph(glyph);
        if (val === null) {
            // This should never happen if normalization is correct
            return { error: `Internal error: glyph not encodable: ${glyph}` };
        }
        values.push(val);
    }
    
    return { values, display: normalized };
}
```

### Phase 3b: Fix JSON Import

```javascript
// In JSON import loop, after fader9:
if (group.name && typeof group.name === 'string') {
    const result = stringToGroupNameValues(group.name);
    if (result.error) {
        // Detailed error message per AI2 requirements
        console.warn(
            `JSON Import Error: Setup ${setup.index + 1}, Group ${group.index + 1}: ` +
            `${result.error} in name "${group.name}"`
        );
        // Don't fail entire import, just skip this name
    } else {
        setGroupName(setup.index, group.index, result.values);
    }
}
```

**Error Message Requirements:**
- Setup index (1-based for display)
- Group index (1-based for display)  
- Offending character (from result.error)
- Original string value

### Phase 4: Undo/Redo with Raw Byte Storage

```javascript
function changeGroupNameWithUndo(setupIdx, group, newValues) {
    const oldData = getGroupNameData(setupIdx, group);
    if (oldData === null) return false;
    
    const oldValues = oldData.values;
    
    // Check if actually changed
    if (oldValues.every((v, i) => v === newValues[i])) {
        return false;
    }
    
    // Get display strings for description
    const oldDisplay = oldData.display;
    const newDisplay = newValues.map(v => decodeValue(v).glyph).join('');
    
    // Record undo action with RAW BYTE VALUES (not strings)
    const action = {
        type: 'group-name',
        description: `Rename group ${group + 1}: "${oldDisplay}" â†’ "${newDisplay}"`,
        setupIdx,
        group,
        beforeValues: oldValues,   // [v0, v1, v2, v3]
        afterValues: newValues     // [v0, v1, v2, v3]
    };
    
    recordUndo(action);
    setGroupName(setupIdx, group, newValues);
    markModified();
    
    return true;
}

// In applyAction() switch statement:
case 'group-name': {
    const values = reverse ? action.beforeValues : action.afterValues;
    setGroupName(action.setupIdx, action.group, values);
    break;
}
```

### Phase 5: UI Modal with Live Validation

**Key UI Behaviour Rules:**

1. **Input field is "trim-right view"** â€” trailing spaces stripped for editing convenience
2. **All comparisons are byte-based** â€” not string-based
3. **No-op save rule** â€” if resulting bytes match original, do nothing (no undo action)
4. **Non-canonical preservation** â€” if user doesn't change input, original bytes preserved
5. **Preview dots** â€” `Â·` shown for spaces is purely visual, never written
6. **Non-canonical + unchanged = no write** â€” If user input is unchanged (after normalization/pad-to-4) and original bytes are non-canonical, Save performs **no write** and preserves original bytesâ€”even if the canonical encoding would differ from the stored bytes
7. **Read failure handling** â€” If originalValues is null (read failure), Save attempts re-read; if still null, show error toast and do not write

```javascript
function showGroupNameEditor(setupIdx, group) {
    const data = getGroupNameData(setupIdx, group);
    const currentDisplay = data ? data.display : `GrP${group + 1}`;
    const hasNonCanonical = data?.hasNonCanonical || false;
    const originalValues = data?.values || null;  // Store for comparison
    
    const dialog = document.createElement('div');
    dialog.className = 'modal-overlay';
    dialog.id = 'groupNameModal';
    dialog.dataset.originalValues = JSON.stringify(originalValues);  // Stash for save comparison
    dialog.dataset.setupIdx = setupIdx;
    dialog.dataset.group = group;
    
    // Trim trailing spaces for input field (trim-right view)
    const inputValue = currentDisplay.replace(/\s+$/, '');
    
    dialog.innerHTML = `
        <div class="modal-dialog group-name-dialog">
            <h3>Edit Group ${group + 1} Name</h3>
            <p class="info">Setup ${setupIdx + 1}</p>
            
            ${hasNonCanonical ? '<p class="warning">âš ï¸ Contains non-standard codes; editor will preserve them unless you overwrite the name.</p>' : ''}
            
            <input type="text" 
                   id="groupNameInput" 
                   value="${escapeHtml(inputValue)}"
                   maxlength="4"
                   autocomplete="off"
                   spellcheck="false">
            
            <p class="preview">UC4 will show: <span id="groupNamePreview">${escapeHtml(currentDisplay)}</span></p>
            
            <p class="hint">Valid: 0-9 A b C d E F G H I J L n O t P S r U Y - _ space</p>
            <p class="error" id="groupNameError"></p>
            
            <div class="modal-actions">
                <button class="btn" onclick="closeGroupNameDialog()">Cancel</button>
                <button class="btn btn-primary" id="groupNameSaveBtn" onclick="saveGroupNameFromDialog()">Save</button>
            </div>
        </div>
    `;
    
    document.body.appendChild(dialog);
    
    const input = document.getElementById('groupNameInput');
    input.focus();
    input.select();
    
    // Live validation
    input.addEventListener('input', validateGroupNameInput);
    
    // Keyboard shortcuts
    input.addEventListener('keydown', (e) => {
        if (e.key === 'Enter' && !document.getElementById('groupNameSaveBtn').disabled) {
            saveGroupNameFromDialog();
        } else if (e.key === 'Escape') {
            closeGroupNameDialog();
        }
    });
    
    validateGroupNameInput();
}

function validateGroupNameInput() {
    const input = document.getElementById('groupNameInput');
    const preview = document.getElementById('groupNamePreview');
    const error = document.getElementById('groupNameError');
    const saveBtn = document.getElementById('groupNameSaveBtn');
    
    const result = stringToGroupNameValues(input.value);
    
    if (result.error) {
        error.textContent = result.error;
        saveBtn.disabled = true;
        preview.textContent = '----';
    } else {
        error.textContent = '';
        saveBtn.disabled = false;
        // Show what UC4 will actually display (Â· for spaces)
        preview.textContent = result.display.replace(/ /g, 'Â·');
    }
}

function saveGroupNameFromDialog() {
    const modal = document.getElementById('groupNameModal');
    const input = document.getElementById('groupNameInput');
    const setupIdx = parseInt(modal.dataset.setupIdx);
    const group = parseInt(modal.dataset.group);
    const originalValues = JSON.parse(modal.dataset.originalValues);
    
    const result = stringToGroupNameValues(input.value);
    
    if (result.error) {
        showToast(result.error, 'error');
        return;
    }
    
    // Handle read failure case (Rule 7: Read failure handling)
    if (originalValues === null) {
        // Re-read to see if it's still failing
        const reread = getGroupNameData(setupIdx, group);
        if (reread === null) {
            showToast('Cannot save: unable to read group name bytes', 'error');
            return;
        }
        // Read succeeded now, proceed with comparison against fresh data
        const currentValues = reread.values;
        const bytesChanged = result.values.some((v, i) => v !== currentValues[i]);
        if (!bytesChanged) {
            closeGroupNameDialog();
            return;
        }
    } else {
        // BYTE-BASED COMPARISON: Check if values actually changed
        const newValues = result.values;
        const bytesChanged = newValues.some((v, i) => v !== originalValues[i]);
        
        if (!bytesChanged) {
            // No-op: bytes unchanged, just close without undo action
            closeGroupNameDialog();
            return;
        }
    }
    
    // Bytes changed: apply with undo
    changeGroupNameWithUndo(setupIdx, group, result.values);
    closeGroupNameDialog();
    
    // Update all UI locations
    renderCurrentView();
    updateGroupNameDisplays();
    
    showToast(`Group ${group + 1} renamed`, 'success');
}

function closeGroupNameDialog() {
    const modal = document.getElementById('groupNameModal');
    if (modal) {
        modal.remove();
    }
    // Note: Cancel does NOT write any bytes (R9 compliance)
}
```

### Phase 6: Non-Canonical Character Warning Badge

```javascript
function renderGroupNameWithWarning(setupIdx, group) {
    const data = getGroupNameData(setupIdx, group);
    
    if (data === null) {
        return `<span class="group-name">GrP${group + 1}</span>`;
    }
    
    const displayText = data.display === '    ' ? '----' : escapeHtml(data.display);
    
    if (data.hasNonCanonical) {
        return `<span class="group-name has-warning" title="Contains non-standard codes; preserved unless overwritten">${displayText} âš ï¸</span>`;
    }
    
    return `<span class="group-name">${displayText}</span>`;
}
```

---

## Testing Checklist

### Reading (Phase 1)
- [ ] Verify group names read correctly from each setup (not just setup 0)
- [ ] Verify factory default names display as GrP1-GrP8 for setups 1-16
- [ ] **Test Ableton setups 17/18** - verify Snd1, Snd2, trAC, RAC, PAn, GLOb display correctly
- [ ] Switch between setups and verify names change appropriately

### Verification (Pre-Implementation)
- [ ] Dump two setups with different group names from UC4 hardware
- [ ] Confirm bytes differ at `setupIdx` boundaries for section 0x14 bank 0x80
- [ ] Document actual byte offsets observed vs expected

### Non-Canonical Byte Handling
- [ ] Import SysEx with byte values outside 0-31 range in group names
- [ ] Confirm warning badge displays (not silent fallback to defaults)
- [ ] Confirm decoded character shows (ASCII for 32-126, ï¿½ for others)
- [ ] Confirm warning text says "will preserve them unless you overwrite"

### Cancel Preserves Bytes (R9)
- [ ] Open modal on name with non-canonical bytes
- [ ] Press Cancel (or Escape)
- [ ] Export SysEx
- [ ] Confirm bytes unchanged from original

### No-Op Save Preserves Bytes (R10)
- [ ] Name bytes contain non-canonical value (e.g., 65 = 'A' ASCII)
- [ ] Open modal, don't change input text, press Save
- [ ] Confirm NO undo action created
- [ ] Export SysEx, confirm bytes unchanged

### Byte-Based Comparison (R10)
- [ ] Name is `"A   "` (A + 3 spaces)
- [ ] Open modal (shows "A" due to trim-right view)
- [ ] Press Save without typing anything
- [ ] Confirm NO undo action created (bytes would be identical)
- [ ] Name is `"A   "`, type "A" explicitly, press Save
- [ ] Confirm NO undo action created (result is same bytes)

### All-Spaces Name
- [ ] Set name to `[31,31,31,31]` (all spaces)
- [ ] Confirm UI displays `----` consistently across all views
- [ ] Confirm JSON exports as `"    "` (4 spaces)
- [ ] Confirm re-import preserves all-spaces

### Lossless Round-Trip
- [ ] Import SysEx with unusual byte values in group name
- [ ] Edit something else (encoder CC, etc.)
- [ ] Export SysEx
- [ ] Confirm group name bytes unchanged

### Undo Integrity
- [ ] Change name when original has non-canonical codes
- [ ] Undo
- [ ] Confirm exact original bytes restored (not re-encoded from display string)

### JSON Import (Phase 3b)
- [ ] Export JSON, verify `name` field is exactly 4 characters for all groups
- [ ] Import JSON with valid names, verify applied
- [ ] Import JSON with invalid characters, verify error message includes:
  - Setup index (1-based)
  - Group index (1-based)
  - Offending character
  - Original string value
- [ ] Import JSON with `\t` or non-breaking space, verify rejected
- [ ] Import JSON with missing `name` fields, verify existing values preserved
- [ ] **Unicode rejection**: Import JSON with NBSP (`\u00A0`) or figure space (`\u2007`) in name â†’ must reject with setup/group and original string

### Validation (Phases 2-3)
- [ ] Type `BASS` â†’ verify normalized to `bASS` and saved correctly
- [ ] Type `Test` â†’ verify normalized to `tESt` and saved correctly  
- [ ] Type `ab` â†’ verify padded to `Ab  ` on save
- [ ] Type `ABCDE` â†’ verify truncated to `AbCd` on save
- [ ] Type `@#$%` â†’ verify rejected with clear error

### Undo/Redo (Phase 4)
- [ ] Verify undo restores previous name
- [ ] Verify redo re-applies new name
- [ ] Verify undo description shows both old and new display strings

### UI (Phases 5-6)
- [ ] Verify clicking group name opens editor modal
- [ ] Verify live validation shows errors immediately
- [ ] Verify preview shows what UC4 will display (with normalized case)
- [ ] Test Enter key saves (only when valid)
- [ ] Test Escape key cancels
- [ ] Verify warning badge appears for non-canonical codes

### Read Failure Handling
- [ ] Simulate `getValue` returning null for one byte in group name
- [ ] Open modal â†’ should show fallback label (`GrP#`)
- [ ] Attempt Save â†’ must fail gracefully with error toast
- [ ] Verify no bytes written and no undo action created

### Regression Test (Original Bug)
- [ ] Load fixture dump with different names in setup 0 vs setup 1
- [ ] Assert `getGroupName(0, g) !== getGroupName(1, g)` for at least one group

### Setup 16/17 Boundary Test
- [ ] Verify packed addressing works at setup 16â†’17 boundary
- [ ] Setup 16 should use section 0x18, bank 0x80, offset 0
- [ ] Setup 17 should use section 0x18, bank 0x80, offset 32
- [ ] Confirm Ableton preset names (Snd1, trAC, etc.) display correctly for setup 17

---

## JSON Schema

```json
{
  "format": "uc4-editor",
  "version": "1.0",
  "setups": [
    {
      "index": 0,
      "groups": [
        {
          "index": 0,
          "name": "GrP1",  // Exactly 4 characters, space-padded, canonical glyphs only
          "encoders": [...],
          "pushButtons": [...],
          "greenButtons": [...],
          "faders": [...],
          "fader9": {...}
        }
      ]
    }
  ]
}
```

**JSON Name Rules:**
- Always exactly 4 characters
- Space-padded if shorter
- Uses canonical glyph charset: `0123456789AbCdEFGHIJLnOtPSrUY-_ `
- Import rejects invalid characters with error identifying setup/group
- Import with missing `name` field preserves existing buffer value

---

## Summary of Required Changes

### Bug Fixes (Phases 1, 3b)

| Location | Line | Change |
|----------|------|--------|
| `getGroupName()` | 2039 | Change `getValue(0,` to `getValue(setupIdx,` |
| `getGroupName()` | 2049 | Remove `name.includes('?')` fallback condition |
| Comment | 2033 | Update to "per-setup" |
| JSON import loop | ~3481 | Add `setGroupName()` call |

### New Code (Phases 2-6)

| Item | Purpose |
|------|---------|
| `GLYPH_TABLE` | Single source of truth valueâ†’glyph |
| `VALUE_TABLE` | Derived glyphâ†’value |
| `INPUT_NORMALIZE` | User inputâ†’canonical glyph |
| `getGroupNameData()` | Returns {values, display, hasNonCanonical} |
| `setGroupName()` | Write 4 byte values to buffer |
| `stringToGroupNameValues()` | Convert input string to byte array |
| `changeGroupNameWithUndo()` | Wrapper storing raw bytes in undo |
| `showGroupNameEditor()` | Modal with live validation |
| `renderGroupNameWithWarning()` | Display with warning badge |

### Modifications to Existing Code

| Function | Change |
|----------|--------|
| `applyAction()` | Add `case 'group-name':` using `beforeValues`/`afterValues` |
| `decode7Seg()` | Replace with `decodeValue()` or update to use `GLYPH_TABLE` |
| HTML | Add click handlers to group name displays |
| CSS | Add warning badge styles, modal styles |

---

## Implementation Priority

1. **Phase 1: Fix Bug #1** â€” Critical, changes `0` to `setupIdx`
2. **Phase 2: Single source mapping** â€” Required foundation
3. **Phase 3: `setGroupName()`** â€” Required for any writing
4. **Phase 3b: Fix JSON import** â€” Important for round-trip
5. **Phase 4: Undo with raw bytes** â€” Important for lossless editing
6. **Phase 5: UI modal** â€” User-facing feature
7. **Phase 6: Warning badges** â€” Polish for edge cases

**Minimum viable fix:** Phase 1 only (fixes display bug)  
**Complete feature:** All phases

---

## Decisions Made (from Open Questions)

1. **Character map accuracy** â€” Treat as "can be incomplete". Design for lossless handling: raw bytes in undo, warning badges for non-canonical codes, never silent fallback.

2. **Case sensitivity** â€” Canonical glyph set defined. Input normalized to match what UC4 actually displays. No "preserve user case" promise.

3. **Empty names** â€” Allowed. Stored as 4 spaces `[31,31,31,31]`. UI displays as `----`. JSON exports as `"    "`.

4. **Per-setup vs linked** â€” Per-setup only. If "apply to all" needed later, make it explicit batch action.

5. **No-op save rule (R10)** â€” Save only applies if resulting bytes differ from original. Comparison is byte-based. No undo action for unchanged saves.

6. **Non-canonical preservation (R9)** â€” Cancel and no-op save preserve original bytes exactly. Non-canonical bytes only overwritten when user explicitly changes the name.

7. **Input field trim-right view** â€” Trailing spaces stripped for editing convenience. Internal comparison always uses full 4-byte values.

8. **Preview dots** â€” `Â·` shown for spaces in preview is purely visual affordance, never written to buffer.
