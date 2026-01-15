# UC4 Setup Manager Specification v2.3

## Overview

Add a Setup Manager to enable high-level operations on the 18 setups: copy, swap, clear, import to specific slots, and export individual setups. Currently, these operations require manual JSON editing.

---

## Problem Statement

**Current state:**
- Setup selector only changes which setup is *viewed/edited*
- No way to copy Setup 5 â†’ Setup 12 within the editor
- No way to import a JSON file into a specific slot
- No way to swap, clear, or duplicate setups
- Workaround requires exporting JSON, manually editing, and re-importing

**User needs:**
- "I built a great setup for my Wavestate, I want to copy it to slots 3, 4, and 5"
- "I want to reorganize my setups so Drumlogue is in slot 1"
- "I downloaded someone's JSON config, I want to put it in slot 7 without overwriting everything"
- "I want to clear setup 18 back to factory defaults"

---

## Definitions

| Term | Definition |
|------|------------|
| `dirtySinceLoad` | Boolean. Set on any edit affecting a setup. Cleared on load/import. NOT cleared on export. Remains true once set, even if user undoes back to initial state (no baseline comparison). |
| `differsFromFactory` | Boolean. Computed lazily by comparing setup against factory template. Cached, invalidated on edit via `invalidateDiffCache()`. Compares **JSON-domain snapshots** (string names, channels 1â€“16), not internal buffer bytes. |
| Setup snapshot | Pure JSON-serializable object containing all 8 groups with all controls and group names. Uses JSON-domain values (1-indexed channels, string names). |

---

## Indexing Conventions (Normative)

### Channel Indexing

| Context | Range | Notes |
|---------|-------|-------|
| JSON files | 1â€“16 | Display values used in import/export |
| Snapshots | 1â€“16 | Snapshots use JSON-domain values |
| UI display | 1â€“16 | What user sees |
| Internal buffer | 0â€“15 | SysEx storage format |
| Validation | 1â€“16 | JSON validation uses display range |

**Conversion:** Internal â†” JSON conversion happens at serialization boundaries only. All spec examples use JSON-domain (1â€“16) unless explicitly noted.

### Setup/Group/Control Indexing

| Context | Range | Notes |
|---------|-------|-------|
| JSON `index` fields | 0â€“17 (setups), 0â€“7 (groups) | Optional, for compatibility |
| Array position | 0-based | Canonical ordering |
| UI display | 1â€“18 (setups), 1â€“8 (groups) | What user sees |

---

## Design Decision: Modal Dialog

**Chosen: Modal Dialog** ("Manage Setups" button)

Rationale:
- Setup management is an occasional, deliberate action (not continuous)
- Modal focuses attention and prevents accidental edits during reorg
- Grid view of all 18 setups enables multi-select operations
- Doesn't consume screen space during normal editing
- Clear entry/exit points

**Interaction model:** Immediate commit. Operations execute when clicked, all undoable via Ctrl+Z. No "Apply/Cancel" staging.

---

## UI Design

### Entry Point

Add button to header, after the setup dropdown:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ UC4 EDITOR    Setup: [â–¼ 1 ]  [Manage Setups]    [Import SysEx] [Export...] â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Modal Layout

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  SETUP MANAGER                                                     [Ã—]      â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                              â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â”‚ Setup 1  â”‚ â”‚ Setup 2  â”‚ â”‚ Setup 3  â”‚ â”‚ Setup 4  â”‚ â”‚ Setup 5  â”‚ â”‚ Setup 6  â”‚
â”‚  â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚
â”‚  â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚ Wvstate  â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚
â”‚  â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 10  â— â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
â”‚                                                                              â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â”‚ Setup 7  â”‚ â”‚ Setup 8  â”‚ â”‚ Setup 9  â”‚ â”‚ Setup 10 â”‚ â”‚ Setup 11 â”‚ â”‚ Setup 12 â”‚
â”‚  â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚
â”‚  â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚
â”‚  â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
â”‚                                                                              â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â”‚ Setup 13 â”‚ â”‚ Setup 14 â”‚ â”‚ Setup 15 â”‚ â”‚ Setup 16 â”‚ â”‚ Setup 17 â”‚ â”‚ Setup 18 â”‚
â”‚  â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚ â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚
â”‚  â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚(no label)â”‚ â”‚ Ableton  â”‚ â”‚ Ableton  â”‚
â”‚  â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1     â”‚ â”‚ Ch 1-8 â— â”‚ â”‚ Ch 1-8 â— â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
â”‚                                                                              â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚  Selected: Setup 4                                                           â”‚
â”‚                                                                              â”‚
â”‚  [Copy to...] [Swap with...] [Clear] [Edit Label] [Import...] [Export]      â”‚
â”‚                                                                              â”‚
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ â”‚
â”‚                                                               [Close]        â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Setup Card Content

Each card shows summary info:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Setup 4  â”‚  â† Header (number, highlight if selected)
â”‚ â”€â”€â”€â”€â”€â”€â”€â”€ â”‚
â”‚ Wvstate  â”‚  â† Label (user-defined, or "(no label)" if none)
â”‚ Ch 10  â— â”‚  â† Primary channel + modified indicator
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Primary channel algorithm:**
1. Collect all MIDI channels (1â€“16) from all controls in all 8 groups
2. Count occurrences; pick max frequency
3. Tie-break: smallest channel number
4. If set equals `{1,2,3,4,5,6,7,8}` exactly â†’ show `Ch 1-8`
5. If one channel dominates (>70% of controls) â†’ show `Ch N`
6. If channels form contiguous range with â‰¥80% coverage â†’ show `Ch X-Y`
7. Otherwise â†’ show `Mixed`

**Vote basis:** 33 controls per group (8 encoders + 8 pushButtons + 8 greenButtons + 8 faders + 1 fader9) = **264 per setup**. Each control contributes one channel vote.

**Modified indicator (â—):** Shown when `differsFromFactory` is true.

**Card states:**
- Default: `var(--bg-control)` background
- Hover: `var(--border-light)` border
- Selected: `var(--accent)` border, slight background tint
- Multi-selected: Same as selected

### Selection Model

- **Single-click:** Select setup (deselects others)
- **Ctrl+click:** Add/remove from selection (multi-select)
- **Shift+click:** Range select
- **Double-click:** Close modal, navigate to that setup in editor

### Keyboard Navigation

When modal opens, focus goes to the currently-active setup's card.

| Key | Action |
|-----|--------|
| Arrow keys | Move selection through grid |
| Enter | Close modal, navigate to selected setup |
| Escape | Close modal |
| C | Copy selected (opens sub-dialog) |
| S | Swap (direct if 2 selected, else opens picker) |
| Delete | Clear selected (opens confirmation) |
| E | Edit label (if exactly 1 selected) |

**Undo/redo while modal open:** Undo/redo is supported while the Setup Manager modal is open. Undo/redo MUST trigger a re-render of Setup Manager cards (labels, primary channel, modified indicator) if the modal is visible.

---

## Multi-Select Behavior

| Operation | Selection State | Behavior |
|-----------|-----------------|----------|
| Copy to... | 1+ selected | First selected = source, opens destination picker |
| Swap with... | Exactly 2 | Direct swap (swaps labels by default), no sub-dialog |
| Swap with... | Exactly 1 | Opens picker sub-dialog |
| Swap with... | 0 or 3+ | Disabled |
| Clear | 1+ selected | All selected cleared (confirmation lists each) |
| Edit Label | Exactly 1 | Opens label editor |
| Edit Label | 0 or 2+ | Disabled |
| Import... | Any | Opens import dialog (selection ignored) |
| Export | Exactly 1 | Exports selected setup |
| Export | 0 or 2+ | Disabled |

---

## Setup Labels

The UC4 hardware doesn't store setup names, but the editor stores them in JSON:

```json
{
  "format": "uc4-editor",
  "version": "1.1",
  "setupLabels": {
    "3": "Wavestate",
    "6": "Drumlogue"
  },
  "setups": [...]
}
```

**Rules:**
- Optional field; missing = no labels
- Max 12 characters per label
- JSON keys are strings `"0"`..`"17"`, interpreted as integers on load
- Internal representation: `Map<number, string>` or sparse array
- Valid characters: alphanumeric, space, hyphen, underscore
- Invalid characters stripped on input
- Displayed in Setup Manager cards and setup dropdown
- NOT sent to UC4 (SysEx doesn't support setup names)
- Survives JSON round-trip, lost on SysEx-only workflow

**UI display:** Shows `(no label)` when a setup has no label.

**Edit Label UX:**
- `[Edit Label]` button enabled when exactly 1 setup selected
- Opens small inline dialog with text input, live validation, Save/Cancel
- Double-click on label area in card also opens editor

---

## Operations

### 1. Copy Setup

**Trigger:** Select source setup â†’ Click "Copy to..." â†’ Select destination(s)

**Flow:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  COPY SETUP 4 TO...                                 â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                     â”‚
â”‚  Select destination slot(s):                        â”‚
â”‚                                                     â”‚
â”‚  [ ] 1  [ ] 2  [ ] 3  [â– ] 4  [ ] 5  [ ] 6          â”‚
â”‚  [ ] 7  [âœ“] 8  [âœ“] 9  [ ] 10 [ ] 11 [ ] 12         â”‚
â”‚  [ ] 13 [ ] 14 [ ] 15 [ ] 16 [ ] 17 [ ] 18         â”‚
â”‚                                                     â”‚
â”‚  [âœ“] Copy setup label ("Wavestate")                 â”‚
â”‚                                                     â”‚
â”‚  âš  This will overwrite setups 8, 9                 â”‚
â”‚                                                     â”‚
â”‚  [Copy]                            [Cancel]         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Behavior:**
- Source setup disabled in destination grid (can't copy to self)
- Multi-select destinations allowed
- Copies all 8 groups with all controls and group names
- Optionally copies setup label
- Single undo action (snapshot-based, stores before AND after)
- Executes immediately on "Copy" click

### 2. Swap Setups

**Trigger:** 
- Select exactly 2 setups â†’ Click "Swap with..." â†’ Direct swap
- Select exactly 1 setup â†’ Click "Swap with..." â†’ Opens picker

**Direct swap behavior:** Swaps all data AND labels by default. No confirmation dialog.

**Flow (when picker needed):**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  SWAP SETUP 4 WITH...                               â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                     â”‚
â”‚  Select setup to swap with:                         â”‚
â”‚                                                     â”‚
â”‚  ( ) 1  ( ) 2  ( ) 3  [â– ] 4  ( ) 5  ( ) 6          â”‚
â”‚  (â—) 7  ( ) 8  ( ) 9  ( ) 10 ( ) 11 ( ) 12         â”‚
â”‚  ( ) 13 ( ) 14 ( ) 15 ( ) 16 ( ) 17 ( ) 18         â”‚
â”‚                                                     â”‚
â”‚  [âœ“] Swap setup labels                              â”‚
â”‚                                                     â”‚
â”‚  Setup 4 (Wavestate) â†” Setup 7 (no label)          â”‚
â”‚                                                     â”‚
â”‚  [Swap]                            [Cancel]         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Behavior:**
- Single selection only in picker (swap is pairwise)
- Source disabled in grid
- Exchanges all data bidirectionally
- Label swap checkbox (default: checked)
- Single undo action (snapshot-based, stores before AND after for both setups)

### 3. Clear Setup

**Trigger:** Select setup(s) â†’ Click "Clear"

**Flow (single setup):**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  CLEAR SETUP 4                                      â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                     â”‚
â”‚  Reset Setup 4 to factory defaults?                 â”‚
â”‚                                                     â”‚
â”‚  This will:                                         â”‚
â”‚  â€¢ Reset all 264 controls to default values         â”‚
â”‚  â€¢ Reset group names to GrP1-GrP8                   â”‚
â”‚  â€¢ Remove setup label ("Wavestate")                 â”‚
â”‚                                                     â”‚
â”‚  This action can be undone.                         â”‚
â”‚                                                     â”‚
â”‚  [Clear Setup]                     [Cancel]         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Flow (multiple setups):**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  CLEAR 3 SETUPS                                     â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                     â”‚
â”‚  Reset the following setups to factory defaults?    â”‚
â”‚                                                     â”‚
â”‚  â€¢ Setup 4 (Wavestate)                              â”‚
â”‚  â€¢ Setup 7 (no label)                               â”‚
â”‚  â€¢ Setup 12 (Drumlogue)                             â”‚
â”‚                                                     â”‚
â”‚  This action can be undone.                         â”‚
â”‚                                                     â”‚
â”‚  [Clear All]                       [Cancel]         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 4. Import JSON to Slot

**Trigger:** Click "Import..." (no selection required)

**Flow:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  IMPORT JSON TO SLOT                                â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                     â”‚
â”‚  [Choose JSON file...]                              â”‚
â”‚                                                     â”‚
â”‚  File: band-configs.json                            â”‚
â”‚  Type: Full dump (18 setups)                        â”‚
â”‚                                                     â”‚
â”‚  Extract setup:                                     â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”  â”‚
â”‚  â”‚  1  â”‚ â”‚  2  â”‚ â”‚ (3) â”‚ â”‚  4  â”‚ â”‚  5  â”‚ â”‚  6  â”‚  â”‚
â”‚  â”‚     â”‚ â”‚     â”‚ â”‚Wvst â”‚ â”‚     â”‚ â”‚     â”‚ â”‚     â”‚  â”‚
â”‚  â””â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”˜  â”‚
â”‚  (showing setups from imported file)                â”‚
â”‚                                                     â”‚
â”‚  Import to slot:                                    â”‚
â”‚  ( ) 1  ( ) 2  ( ) 3  (â—) 4  ( ) 5  ( ) 6          â”‚
â”‚  ( ) 7  ( ) 8  ( ) 9  ( ) 10 ( ) 11 ( ) 12         â”‚
â”‚                                                     â”‚
â”‚  [âœ“] Import label ("Wavestate")                     â”‚
â”‚                                                     â”‚
â”‚  âš  This will overwrite Setup 4 (Drumlogue)         â”‚
â”‚                                                     â”‚
â”‚  [Import]                          [Cancel]         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Supported JSON formats:**

1. **Full dump** (`format: "uc4-editor"`) - Show mini-grid to pick which setup to extract
2. **Single-setup** (`format: "uc4-editor-setup"`) - Direct import, skip extraction picker

**Default selections:**
- **Extract setup (full dump):** First labeled setup if file has `setupLabels`, else Setup 1
- **Import to slot:** Currently selected setup if exactly 1 selected in modal, else current setup from main dropdown, else Setup 1

### 5. Export Single Setup

**Trigger:** Select exactly 1 setup â†’ Click "Export"

**Exports single-setup format** (see JSON Schema section).

**Filename generation:**
- Pattern: `uc4-setup-{N}-{label}.json` or `uc4-setup-{N}.json`
- Label sanitized: lowercase, replace invalid chars with hyphen
- Invalid characters: `/ \ : * ? " < > |` and control characters (0x00-0x1F)
- Multiple consecutive hyphens collapse to one
- Leading/trailing hyphens removed

**Examples:**
| Setup | Label | Filename |
|-------|-------|----------|
| 4 | "Wavestate" | `uc4-setup-4-wavestate.json` |
| 7 | None | `uc4-setup-7.json` |
| 3 | "My/Setup:Test" | `uc4-setup-3-my-setup-test.json` |

---

## Snapshot System

Setup Manager operations use **setup-level snapshots** for undo/redo, not per-control micro-actions.

### Snapshot Format

Snapshots use **JSON-domain values** (1-indexed channels, string names, `pushButtons`/`greenButtons` naming):

```javascript
// Pure JSON-serializable, matches export format structure
{
  groups: [
    {
      name: "GrP1",  // String (decoded from glyph values)
      encoders: [
        { channel: 1, type: 2, cc: 1, min: 0, max: 127, acc: 3, display: 1 },
        { channel: 1, type: 2, cc: 2, min: 0, max: 127, acc: 3, display: 1 },
        // ... 8 total
      ],
      pushButtons: [
        { channel: 1, typeNibble: 16, cc: 36, lower: 0, upper: 127, mode: 0, display: 1 },
        // ... 8 total
      ],
      greenButtons: [
        { channel: 1, typeNibble: 16, cc: 44, lower: 0, upper: 127, mode: 0, display: 1 },
        // ... 8 total
      ],
      faders: [
        { channel: 1, type: 0, cc: 32, min: 0, max: 127, mode: 0, display: 1 },
        // ... 8 total
      ],
      fader9: { channel: 1, cc: 112, min: 0, max: 127, mode: 0 }
    },
    // ... 8 groups total
  ]
}
```

### Helper Functions

```javascript
function captureSetupSnapshot(setupIdx) {
    return {
        groups: Array.from({ length: 8 }, (_, g) => ({
            name: getGroupNameString(setupIdx, g),  // Returns decoded string
            encoders: Array.from({ length: 8 }, (_, i) => getEncoderData(setupIdx, g, i)),
            pushButtons: Array.from({ length: 8 }, (_, i) => getPushData(setupIdx, g, i)),
            greenButtons: Array.from({ length: 8 }, (_, i) => getGreenData(setupIdx, g, i)),
            faders: Array.from({ length: 8 }, (_, i) => getFaderData(setupIdx, g, i)),
            fader9: getFader9Data(setupIdx, g),
        })),
    };
}

function restoreSetupSnapshot(setupIdx, snap) {
    for (let g = 0; g < 8; g++) {
        setGroupNameString(setupIdx, g, snap.groups[g].name);
        for (let i = 0; i < 8; i++) {
            setEncoderData(setupIdx, g, i, snap.groups[g].encoders[i]);
            setPushData(setupIdx, g, i, snap.groups[g].pushButtons[i]);
            setGreenData(setupIdx, g, i, snap.groups[g].greenButtons[i]);
            setFaderData(setupIdx, g, i, snap.groups[g].faders[i]);
        }
        setFader9Data(setupIdx, g, snap.groups[g].fader9);
    }
    // Note: cache invalidation handled by caller (batch operations invalidate all affected indices)
}

// New helpers for group name string conversion
function getGroupNameString(setupIdx, groupIdx) {
    const values = getGroupNameValues(setupIdx, groupIdx);
    return decodeGroupName(values);  // Converts glyph values to string
}

function setGroupNameString(setupIdx, groupIdx, str) {
    const values = encodeGroupName(str);  // Converts string to glyph values
    setGroupName(setupIdx, groupIdx, values);
}

// Cache invalidation - called by operations, not by restoreSetupSnapshot
function invalidateDiffCache(indices) {
    for (const idx of indices) {
        differsFromFactoryCache.delete(idx);
    }
}
```

### Undo Record Structure

All operations store **both** `beforeSnapshots` and `afterSnapshots` for correct undo/redo:

```javascript
// Copy operation
{
    type: 'setup-copy',
    description: 'Copy Setup 4 to 8, 9',
    sourceIdx: 3,
    destIndices: [7, 8],
    beforeSnapshots: { 7: {...}, 8: {...} },
    afterSnapshots: { 7: {...}, 8: {...} },
    labelsBefore: { 7: null, 8: 'OldLabel' },
    labelsAfter: { 7: 'Wavestate', 8: 'Wavestate' }
}

// Swap operation
{
    type: 'setup-swap',
    description: 'Swap Setup 4 â†” Setup 7',
    idxA: 3,
    idxB: 6,
    beforeSnapshots: { 3: {...}, 6: {...} },
    afterSnapshots: { 3: {...}, 6: {...} },
    labelsBefore: { 3: 'Wavestate', 6: null },
    labelsAfter: { 3: null, 6: 'Wavestate' }
}

// Clear operation
{
    type: 'setup-clear',
    description: 'Clear Setup 4, 7',
    indices: [3, 6],
    beforeSnapshots: { 3: {...}, 6: {...} },
    afterSnapshots: { 3: {...}, 6: {...} },  // Factory snapshots
    labelsBefore: { 3: 'Wavestate', 6: 'Drumlogue' },
    labelsAfter: { 3: null, 6: null }
}

// Import operation
{
    type: 'setup-import',
    description: 'Import to Setup 4',
    targetIdx: 3,
    beforeSnapshot: {...},
    afterSnapshot: {...},
    labelBefore: 'OldLabel',
    labelAfter: 'Wavestate'
}
```

### Undo/Redo Implementation

```javascript
function applySetupManagerUndo(action) {
    let affectedSetups = [];
    
    switch (action.type) {
        case 'setup-copy':
            affectedSetups = action.destIndices;
            for (const idx of action.destIndices) {
                restoreSetupSnapshot(idx, action.beforeSnapshots[idx]);
                setupLabels.set(idx, action.labelsBefore[idx]);
            }
            break;
            
        case 'setup-swap':
            affectedSetups = [action.idxA, action.idxB];
            restoreSetupSnapshot(action.idxA, action.beforeSnapshots[action.idxA]);
            restoreSetupSnapshot(action.idxB, action.beforeSnapshots[action.idxB]);
            setupLabels.set(action.idxA, action.labelsBefore[action.idxA]);
            setupLabels.set(action.idxB, action.labelsBefore[action.idxB]);
            break;
            
        case 'setup-clear':
            affectedSetups = action.indices;
            for (const idx of action.indices) {
                restoreSetupSnapshot(idx, action.beforeSnapshots[idx]);
                setupLabels.set(idx, action.labelsBefore[idx]);
            }
            break;
            
        case 'setup-import':
            affectedSetups = [action.targetIdx];
            restoreSetupSnapshot(action.targetIdx, action.beforeSnapshot);
            setupLabels.set(action.targetIdx, action.labelBefore);
            break;
    }
    
    invalidateDiffCache(affectedSetups);
    markModified();
    refreshMainEditorIfNeeded(affectedSetups);
    refreshSetupManagerIfOpen();
}

function applySetupManagerRedo(action) {
    let affectedSetups = [];
    
    switch (action.type) {
        case 'setup-copy':
            affectedSetups = action.destIndices;
            for (const idx of action.destIndices) {
                restoreSetupSnapshot(idx, action.afterSnapshots[idx]);
                setupLabels.set(idx, action.labelsAfter[idx]);
            }
            break;
            
        case 'setup-swap':
            affectedSetups = [action.idxA, action.idxB];
            restoreSetupSnapshot(action.idxA, action.afterSnapshots[action.idxA]);
            restoreSetupSnapshot(action.idxB, action.afterSnapshots[action.idxB]);
            setupLabels.set(action.idxA, action.labelsAfter[action.idxA]);
            setupLabels.set(action.idxB, action.labelsAfter[action.idxB]);
            break;
            
        case 'setup-clear':
            affectedSetups = action.indices;
            for (const idx of action.indices) {
                restoreSetupSnapshot(idx, action.afterSnapshots[idx]);
                setupLabels.set(idx, action.labelsAfter[idx]);
            }
            break;
            
        case 'setup-import':
            affectedSetups = [action.targetIdx];
            restoreSetupSnapshot(action.targetIdx, action.afterSnapshot);
            setupLabels.set(action.targetIdx, action.labelAfter);
            break;
    }
    
    invalidateDiffCache(affectedSetups);
    markModified();
    refreshMainEditorIfNeeded(affectedSetups);
    refreshSetupManagerIfOpen();
}

function refreshSetupManagerIfOpen() {
    if (setupManagerModalVisible) {
        renderSetupManagerCards();  // Updates labels, primary channel, modified indicators
    }
}
```

---

## Main Editor Refresh

Refresh main editor only when the currently-displayed setup is affected:

```javascript
function refreshMainEditorIfNeeded(affectedSetups) {
    if (affectedSetups.includes(currentSetupIdx)) {
        updateGroupNames();
        renderCurrentView();
    }
}
```

**Normative:** `applySetupManagerUndo/Redo()` MUST call `refreshMainEditorIfNeeded()` with affected indices:
- copy: `destIndices`
- swap: `[idxA, idxB]`
- clear: `indices`
- import: `[targetIdx]`

---

## Import Validation Rules

### Structural Validation (STRICT - hard fail)

Reject import if:
- Missing required top-level fields (`format`, `groups` or `setups`)
- Wrong `format` value (not `uc4-editor` or `uc4-editor-setup`)
- Wrong group count (must be exactly 8)
- Wrong control count per type (must be exactly 8, except fader9 which is 1)

**Required group keys:** Each group object MUST contain:
- `name` (string)
- `encoders` (array, length 8)
- `pushButtons` (array, length 8)
- `greenButtons` (array, length 8)
- `faders` (array, length 8)
- `fader9` (object)

**Error examples:**
- `"Not a UC4 configuration file (unknown format)"`
- `"Invalid structure: expected 8 groups, found 6"`
- `"Invalid structure: group 3 missing fader9"`
- `"Invalid structure: group 2 encoders array length 5, expected 8"`

### Value Validation (COERCE + WARN - soft fail)

For each field:
- **Known field, out-of-range:** Clamp to valid range + warning
- **Known field, wrong type:** Attempt parse if safe, else use factory default + warning
- **Unknown field:** Ignore + warning
- **Enum field invalid:** Use factory default + warning (don't clamp enums)

**Defaults source:** When coercing invalid values, defaults come from `FACTORY_DEFAULTS` for that control type and index.

**Partial objects:** If a control object is present but missing fields, fill from factory defaults with warning.

**Validation ranges (JSON domain):**

| Field | Range | Notes |
|-------|-------|-------|
| channel | 1â€“16 | Clamped |
| cc | 0â€“127 | Clamped |
| min, max | 0â€“127 | Clamped |
| lower, upper | 0â€“127 | Clamped |
| type (encoder) | 0â€“6 | Default to 2 (CCAb) if invalid |
| type (fader) | 0â€“3 | Default to 0 (CCAb) if invalid |
| typeNibble | 0â€“127 | Clamped. If NaN/invalid â†’ factory default. If not nibble-aligned (value % 16 â‰  0) â†’ warning |
| acc | 0â€“3 | Default to 3 if invalid |
| mode | 0â€“1 | Default to 0 if invalid |
| display | 0â€“2 | Default to 1 if invalid |

**Warning format:** Include JSONPath-like path
- `"groups[2].encoders[4].cc: value 200 clamped to 127"`
- `"groups[0].faders[1].type: invalid value 99, using default 0"`
- `"groups[5]: unknown field 'velocity' ignored"`
- `"groups[3].pushButtons[2].typeNibble: value 17 not nibble-aligned"`

### Pre-Import Validation Dialog

If validation produces warnings, show before import:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  IMPORT VALIDATION                                  â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                     â”‚
â”‚  âš  3 issues found:                                 â”‚
â”‚                                                     â”‚
â”‚  â€¢ groups[2].encoders[4].cc: 200 â†’ clamped to 127  â”‚
â”‚  â€¢ groups[0].faders[1]: unknown field "velocity"   â”‚
â”‚  â€¢ groups[7].fader9.mode: invalid value, using 0   â”‚
â”‚                                                     â”‚
â”‚  [Import Anyway]                   [Cancel]         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Zero warnings:** Skip dialog, import directly.

**Excessive warnings (>50):** Show special warning: "File produced excessive warnings and may be incompatible."

---

## Factory Defaults

**Validation requirement:** Factory defaults MUST be validated against `factory_default.syx` before release. Add test that clears a setup and compares resulting bytes against known factory dump.

All values are in **JSON domain** (1-indexed channels):

```javascript
const FACTORY_DEFAULTS = {
    encoder: (idx) => ({
        channel: 1,
        type: 2,      // CCAb
        cc: idx + 1,  // 1-8
        min: 0,
        max: 127,
        acc: 3,
        display: 1
    }),
    
    fader: (idx) => ({
        channel: 1,
        type: 0,       // CCAb
        cc: 32 + idx,  // 32-39
        min: 0,
        max: 127,
        mode: 0,
        display: 1
    }),
    
    pushButton: (idx) => ({
        channel: 1,
        typeNibble: 16,  // Note
        cc: 36 + idx,    // 36-43
        lower: 0,
        upper: 127,
        mode: 0,
        display: 1
    }),
    
    greenButton: (idx) => ({
        channel: 1,
        typeNibble: 16,  // Note
        cc: 44 + idx,    // 44-51
        lower: 0,
        upper: 127,
        mode: 0,
        display: 1
    }),
    
    fader9: () => ({
        channel: 1,
        cc: 112,
        min: 0,
        max: 127,
        mode: 0
    }),
    
    groupName: (groupIdx) => `GrP${groupIdx + 1}`  // "GrP1" through "GrP8"
};

function createFactorySetupSnapshot() {
    return {
        groups: Array.from({ length: 8 }, (_, g) => ({
            name: FACTORY_DEFAULTS.groupName(g),
            encoders: Array.from({ length: 8 }, (_, i) => FACTORY_DEFAULTS.encoder(i)),
            pushButtons: Array.from({ length: 8 }, (_, i) => FACTORY_DEFAULTS.pushButton(i)),
            greenButtons: Array.from({ length: 8 }, (_, i) => FACTORY_DEFAULTS.greenButton(i)),
            faders: Array.from({ length: 8 }, (_, i) => FACTORY_DEFAULTS.fader(i)),
            fader9: FACTORY_DEFAULTS.fader9(),
        })),
    };
}
```

**Note:** Default CC numbering follows actual factory dump. Encoder CCs are 1-8 (not 0-7). This must be validated against hardware in Phase 3.

---

## JSON Schema

### Existing Export Compatibility (Normative)

The editor MUST:
- Accept v1.0 structure exactly as currently exported
- Maintain 1-indexed channels (1â€“16) in all JSON
- Accept `index` fields in setups and groups (for v1.0 compatibility)
- Ignore unknown fields on import (do not preserve on round-trip)
- Unknown fields MAY emit warnings during validation

### Index Field Handling

`setups[*].index` and `groups[*].index` fields:
- **Optional** in v1.1+ (may be omitted)
- **If present:** Importer validates consistency with array position
- **If mismatch:** Warn and prefer array order (do not hard fail)
- **Canonical ordering:** Array position is authoritative
- **Export requirement:** v1.1 full-dump export MUST include `setups[*].index` and `groups[*].index` fields for backward compatibility

### Version 1.0 Full Dump (Current)

```json
{
  "format": "uc4-editor",
  "version": "1.0",
  "exported": "2026-01-15T12:00:00.000Z",
  "setups": [
    {
      "index": 0,
      "groups": [
        {
          "index": 0,
          "name": "GrP1",
          "encoders": [
            { "channel": 1, "type": 2, "cc": 1, "min": 0, "max": 127, "acc": 3, "display": 1 },
            { "channel": 1, "type": 2, "cc": 2, "min": 0, "max": 127, "acc": 3, "display": 1 }
            // ... 8 total (arrays shown truncated)
          ],
          "pushButtons": [
            { "channel": 1, "typeNibble": 16, "cc": 36, "lower": 0, "upper": 127, "mode": 0, "display": 1 }
            // ... 8 total
          ],
          "greenButtons": [
            { "channel": 1, "typeNibble": 16, "cc": 44, "lower": 0, "upper": 127, "mode": 0, "display": 1 }
            // ... 8 total
          ],
          "faders": [
            { "channel": 1, "type": 0, "cc": 32, "min": 0, "max": 127, "mode": 0, "display": 1 }
            // ... 8 total
          ],
          "fader9": { "channel": 1, "cc": 112, "min": 0, "max": 127, "mode": 0 }
        }
        // ... 8 groups total
      ]
    }
    // ... 18 setups total
  ]
}
```

### Version 1.1 Full Dump (With Labels)

```json
{
  "format": "uc4-editor",
  "version": "1.1",
  "exported": "2026-01-15T12:00:00.000Z",
  "setupLabels": {
    "3": "Wavestate",
    "6": "Drumlogue"
  },
  "setups": [
    // Same structure as v1.0, index fields included for compatibility
  ]
}
```

**Compatibility:**
- v1.0 import: No labels (missing `setupLabels` field)
- v1.1 import: Labels supported
- Older editors ignore unknown `setupLabels` field

### Single-Setup Export Format (NEW)

```json
{
  "format": "uc4-editor-setup",
  "version": "1.0",
  "exported": "2026-01-15T12:00:00.000Z",
  "sourceSetup": 3,
  "label": "Wavestate",
  "groups": [
    {
      "name": "FILt",
      "encoders": [
        { "channel": 1, "type": 2, "cc": 74, "min": 0, "max": 127, "acc": 3, "display": 1 }
        // ... 8 total
      ],
      "pushButtons": [
        // ... 8 total
      ],
      "greenButtons": [
        // ... 8 total
      ],
      "faders": [
        // ... 8 total
      ],
      "fader9": { "channel": 1, "cc": 11, "min": 0, "max": 127, "mode": 0 }
    }
    // ... 8 groups total (no index fields, array position is canonical)
  ]
}
```

**Notes:**
- `sourceSetup`: Informational only, **0â€“17 internal index**. UI displays `sourceSetup + 1`.
- `label`: Optional (omit if no label)
- Groups array is ordered (position 0-7), **no** `index` field (new format, no legacy)
- Control arrays are ordered (position 0-7)

---

## Test Matrix

### Copy Setup Tests

| ID | Source | Dest | Label | Expected |
|----|--------|------|-------|----------|
| C1 | 1 | 2 | Yes | All data + label copied |
| C2 | 1 | 2 | No | Data copied, dest label unchanged |
| C3 | 5 | 6,7,8 | Yes | Multi-copy works, single undo |
| C4 | 1 | 1 | - | Disabled (can't copy to self) |
| C5 | 1 | 2 | - | Undo restores Setup 2 exactly |
| C6 | 1 | 2 | - | Redo re-copies correctly |

### Swap Setup Tests

| ID | A | B | Labels | Expected |
|----|---|---|--------|----------|
| S1 | 1 | 2 | Yes | Complete bidirectional exchange |
| S2 | 3 | 3 | - | Disabled (can't swap with self) |
| S3 | 5 | 10 | No | Data swapped, labels unchanged |
| S4 | 1 | 18 | Yes | Swapping with Ableton preset works |
| S5 | 4 | 7 | - | Undo restores both setups |
| S6 | 4 | 7 | - | Redo re-swaps correctly |
| S7 | 4 | 7 | Direct | Direct swap (2 selected) swaps labels by default |

### Clear Setup Tests

| ID | Setup(s) | Has Label | Expected |
|----|----------|-----------|----------|
| CL1 | 1 | No | Reset to factory defaults |
| CL2 | 5 | Yes | Reset + label removed |
| CL3 | 4,7,12 | Mixed | All three cleared, single undo |
| CL4 | 1 | - | Undo restores previous state |
| CL5 | 17 | Yes | Ableton preset clearable |
| CL6 | 1 | - | Redo clears again |

### Import Tests

| ID | JSON Type | Extract | Target | Expected |
|----|-----------|---------|--------|----------|
| I1 | Full dump | Setup 3 | Slot 5 | Extracted setup imported |
| I2 | Single | N/A | Slot 3 | Direct import |
| I3 | Invalid JSON | - | - | Parse error shown |
| I4 | Wrong format | - | - | "Not a UC4 config" error |
| I5 | Valid w/ warnings | - | 4 | Validation dialog shown |
| I6 | Full dump | - | - | Default to first labeled setup |
| I7 | Valid w/ warnings | - | - | "Import Anyway" â†’ import proceeds |
| I8 | Valid w/ warnings | - | - | "Cancel" â†’ no import, no state change |
| I9 | Any | - | - | Target defaults to selected setup if 1 selected |

### Export Tests

| ID | Setup | Label | Expected Filename |
|----|-------|-------|-------------------|
| E1 | 4 | "Wavestate" | `uc4-setup-4-wavestate.json` |
| E2 | 7 | None | `uc4-setup-7.json` |
| E3 | 3 | "My/Setup:Test" | `uc4-setup-3-my-setup-test.json` |

### Undo/Redo Integrity Tests

| ID | Operation | Test |
|----|-----------|------|
| U1 | Copy | Undoâ†’Redo: dest matches source again |
| U2 | Swap | Undoâ†’Redo: positions swapped again |
| U3 | Clear | Undoâ†’Redo: factory state again |
| U4 | Import | Undoâ†’Redo: imported data again |
| U5 | Any | Undo after modal close works |
| U6 | Any | Multiple undo/redo cycles stable |
| U7 | Any | Undo/redo while modal open updates cards |
| U8 | Any | Undo to initial state: `dirtySinceLoad` remains true |

### Label Edge Cases

| ID | Test |
|----|------|
| L1 | Import with invalid label chars â†’ sanitized |
| L2 | Label > 12 chars â†’ truncated |
| L3 | Copy with copyLabel=false â†’ dest label unchanged |
| L4 | Clear removes label |
| L5 | Edit label with special chars â†’ sanitized |

### Current Setup Refresh

| ID | Test |
|----|------|
| R1 | Clear current setup â†’ main editor updates immediately |
| R2 | Swap involving current â†’ main editor updates |
| R3 | Import to current slot â†’ main editor updates |
| R4 | Copy to non-current setup â†’ main editor NOT refreshed |

### Index Field Compatibility

| ID | Test |
|----|------|
| X1 | Import v1.0 with index fields â†’ accepted |
| X2 | Import with mismatched index â†’ warn, use array order |
| X3 | Export v1.1 â†’ index fields present for compatibility |

---

## Implementation Phases

### Phase 1: Modal Infrastructure
1. Add "Manage Setups" button to header
2. Create modal overlay and basic layout
3. Render 18 setup cards (placeholder content)
4. Implement selection model (single/multi/range)
5. Wire close behavior and Escape key
6. Keyboard navigation (arrows, Enter)

### Phase 2: Setup Summary & Snapshots
1. Define `FACTORY_DEFAULTS` constants (provisional, validated in Phase 3)
2. Implement `captureSetupSnapshot()` / `restoreSetupSnapshot()`
3. Add `getGroupNameString()` / `setGroupNameString()` helpers
4. Implement `invalidateDiffCache()`
5. Calculate primary channel per setup
6. Implement `differsFromFactory` computation (lazy, cached)
7. Add setup labels storage (in-memory Map)
8. Update JSON export for labels (version 1.1)
9. Update JSON import for labels
10. Show labels in setup dropdown
11. Show modified indicator on cards

### Phase 3: Factory Defaults Validation
1. Create test comparing `FACTORY_DEFAULTS` against `factory_default.syx`
2. Adjust constants if discrepancies found
3. Document any differences from initial assumptions

### Phase 4: Copy Operation
1. Build "Copy to..." sub-dialog
2. Implement multi-destination selection
3. Implement `copySetup()` with snapshot-based undo (before + after)
4. Test undo/redo integrity

### Phase 5: Swap Operation
1. Build "Swap with..." sub-dialog
2. Implement direct swap when exactly 2 selected (swaps labels by default)
3. Implement `swapSetups()` with snapshot-based undo
4. Test bidirectional exchange

### Phase 6: Clear Operation
1. Build confirmation dialogs (single and multi)
2. Implement `clearSetups()` with snapshot-based undo
3. Handle label removal
4. Test against factory state

### Phase 7: Edit Label
1. Build inline label editor dialog
2. Input validation (12 chars, valid chars)
3. Double-click shortcut on card
4. Update card display on save

### Phase 8: Import to Slot
1. Define single-setup JSON format
2. Implement format detection in file picker
3. Build import sub-dialog with setup extraction picker
4. Implement validation rules (strict structure, coerce values)
5. Build validation warnings dialog
6. Handle both full-dump and single-setup imports
7. Implement target slot default logic
8. Implement with snapshot-based undo

### Phase 9: Export Single Setup
1. Implement single-setup JSON export
2. Generate sanitized filename from setup number + label
3. Add to modal actions

### Phase 10: Polish
1. Visual feedback (animations, transitions)
2. Error handling edge cases
3. Help tooltips
4. Accessibility review

### Future: Drag-and-Drop
- Drag to swap
- Ctrl+drag to copy
- Visual drop zone feedback

---

## Summary

The Setup Manager provides essential high-level operations:

| Operation | Before | After |
|-----------|--------|-------|
| Copy setup | Export JSON, edit, import | 3 clicks |
| Swap setups | Export, edit both, import | 2-3 clicks |
| Clear setup | Export, edit to defaults, import | 2 clicks |
| Import to slot | Not possible | Choose file, choose slot |
| Export single | Export all, extract manually | Select, export |

**Key design decisions:**
- Immediate commit model (no Apply/Cancel)
- Snapshot-based undo with before AND after states
- JSON-domain values throughout (1-indexed channels, string names)
- Consistent naming (`pushButtons`, `greenButtons`)
- v1.0 backward compatibility maintained
- Factory defaults validated against actual hardware
- Refresh gating prevents unnecessary repaints
- Undo/redo supported while modal open
- `dirtySinceLoad` persists even after undo to initial state
- Direct swap includes labels by default

All operations integrate with existing undo/redo system and maintain the single-source-of-truth architecture.
