# UC4 SysEx Editor — Complete Usage Guide

A web-based configuration editor for the **Faderfox UC4** MIDI controller. Edit all 18 setups, 8 groups, and every parameter without navigating the hardware menus.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Interface Overview](#interface-overview)
3. [Working with Files](#working-with-files)
4. [Navigating Setups & Groups](#navigating-setups--groups)
5. [Focused View — Detailed Editing](#focused-view--detailed-editing)
6. [Overview Mode — See Everything](#overview-mode--see-everything)
7. [Quick Copy/Paste — Rapid Configuration](#quick-copypaste--rapid-configuration)
8. [Setup Manager — Organize Your Setups](#setup-manager--organize-your-setups)
9. [Conflict Detection](#conflict-detection)
10. [Context Menu Copy & Paste](#context-menu-copy--paste)
11. [Undo & Redo](#undo--redo)
12. [Session Persistence](#session-persistence)
13. [Keyboard Shortcuts](#keyboard-shortcuts)
14. [Workflow Examples](#workflow-examples)
15. [Validation & Troubleshooting](#validation--troubleshooting)

---

## Quick Start

```
┌────────────────────────────────────────────────────────────────┐
│  1. Open editor        →  Factory defaults load automatically  │
│  2. Select Setup 1-18  →  Pick which setup to edit            │
│  3. Edit in Focused    →  Change individual parameters        │
│  4. Check in Overview  →  See all 64 controls at once         │
│  5. Export .syx        →  Send to UC4 via MIDI                │
└────────────────────────────────────────────────────────────────┘
```

**That's it!** The editor auto-loads factory defaults so you can start immediately.

---

## Interface Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│   UC4 SysEx Editor          [Import SysEx] [Export SysEx] [Import JSON]        │
│                             [Export JSON]  [↶ Undo] [↷ Redo] [Manage Setups]   │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   Setup [01 ▾]    [🔗 Link]    Encoder Grp [1][2][3][4][5][6][7][8]  GrP1      │
│                                Fader/Btn   [1][2][3][4][5][6][7][8]  GrP1      │
│                                                                                 │
│                                                       [Focused] [Overview]      │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                                                                                 │
│                                                                                 │
│                          ┌───────────────────────┐                              │
│                          │                       │                              │
│                          │    MAIN EDITING       │                              │
│                          │        AREA           │                              │
│                          │                       │                              │
│                          │  (Focused View or     │                              │
│                          │   Overview Mode)      │                              │
│                          │                       │                              │
│                          └───────────────────────┘                              │
│                                                                                 │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│   ● Modified                                             100,640 bytes loaded   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Header Elements

| Button | What It Does |
|--------|--------------|
| **Import SysEx** | Load a .syx dump from your UC4 |
| **Export SysEx** | Save changes as .syx to send back to UC4 |
| **Import JSON** | Load a human-readable JSON backup |
| **Export JSON** | Save human-readable JSON (great for git!) |
| **↶ Undo** | Reverse your last change |
| **↷ Redo** | Replay an undone change |
| **Manage Setups** | Open Setup Manager for bulk operations |

### Navigation Elements

| Element | What It Does |
|---------|--------------|
| **Setup [▾]** | Choose which of the 18 setups to edit |
| **🔗 Link** | Sync encoder and fader groups together |
| **Encoder Grp [1-8]** | Select group for encoders + push buttons |
| **Fader/Btn [1-8]** | Select group for faders + green buttons |
| **GrP1** | Shows the 4-character group name from UC4 |
| **Focused / Overview** | Toggle between edit modes |

### Status Bar

| Indicator | Meaning |
|-----------|---------|
| **● Modified** | You have unsaved changes |
| **○** (outline) | Data matches last saved state |
| **100,640 bytes** | Valid UC4 dump is loaded |

---

## Working with Files

### SysEx Files (.syx)

The native format for UC4 data.

**Import:**
1. Click **Import SysEx**
2. Select a .syx file (must be exactly 100,640 bytes)
3. File loads immediately

**Export:**
1. Click **Export SysEx**
2. File downloads to your Downloads folder
3. Send this file to your UC4 via MIDI

### JSON Files

Human-readable format, great for backups and version control.

**Import:**
1. Click **Import JSON**
2. Select a .json file exported from this editor
3. All 18 setups are loaded

**Export:**
1. Click **Export JSON**
2. File downloads with timestamp in filename
3. Can be opened in any text editor

### Single Setup JSON

Export and import individual setups via the Setup Manager.

**Export Single Setup:**
1. Click **Manage Setups**
2. Select one setup
3. Click **Export**
4. Choose filename and save

**Import to Slot:**
1. Click **Manage Setups**
2. Click **Import**
3. Select single-setup JSON file
4. Choose target slot
5. Click **Import**

---

## Navigating Setups & Groups

### The UC4's Two-Domain System

The UC4 has **two independent group selectors**:

```
ENCODER DOMAIN (Shift + Encoder 1-8)
├── 8 Encoders
└── 8 Push Buttons

FADER/BUTTON DOMAIN (Shift + Green 1-8)
├── 8 Faders (1-8)
├── 8 Green Buttons
└── Fader 9
```

You can be on Encoder Group 3 while Fader Group 7 is active. They're independent!

### Link Groups Toggle

**Enabled (🔗 checked):**
- Clicking any group tab sets BOTH domains to that group
- Useful when your groups are organized by channel (Group 1 = Ch1, etc.)

**Disabled:**
- Encoder and Fader groups are independent
- Matches actual UC4 hardware behavior

### Setup Selector

- Dropdown shows setups 1-18
- If you've labeled setups, labels appear: "1: Synth"
- Switch freely—changes are tracked per setup

---

## Focused View — Detailed Editing

The default editing mode, showing one group at a time with full parameter access.

### Layout

```
┌─ FADERS ───────────────────────────────────── GrP1 ─┐
│                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │ Fader 1     │  │ Fader 2     │  │ Fader 3     │   │
│  │ Chan [1 ▾]  │  │ Chan [1 ▾]  │  │ Chan [1 ▾]  │   │
│  │ CC   [ 1  ] │  │ CC   [ 2  ] │  │ CC   [ 3  ] │   │
│  │ Type [CCAb] │  │ Type [CCAb] │  │ Type [CCAb] │   │
│  │ Min  [ 0  ] │  │ Min  [ 0  ] │  │ Min  [ 0  ] │   │
│  │ Max  [127 ] │  │ Max  [127 ] │  │ Max  [127 ] │   │
│  └─────────────┘  └─────────────┘  └─────────────┘   │
└───────────────────────────────────────────────────────┘
```

### Section Order

1. **Faders** (from Fader group)
2. **Green Buttons** (from Fader group)
3. **Encoders** (from Encoder group)
4. **Push Buttons** (from Encoder group)
5. **Fader 9** (from Fader group)

### Contextual Tooltips

**Hover over any parameter label or dropdown option to see explanations!**

| Example | Tooltip |
|---------|---------|
| Hover over **Chan** | "MIDI channel (1-16). Controls which channel receives messages." |
| Hover over **CCr1** option | "Relative mode 1. Sends 1 for clockwise, 127 for counter-clockwise. Best for Ableton, Bitwig." |
| Hover over **Acc** | "Acceleration sensitivity. Higher values = faster response to quick turns." |

### Control Parameters

**Encoder:**
| Param | Range | Description |
|-------|-------|-------------|
| Chan | 1-16 | MIDI channel |
| CC | 0-127 | Controller number |
| Type | CCr1/CCr2/CCAb/PrGC/CCAh/Pbnd/AFtt | Message type |
| Acc | 0-4 | Acceleration sensitivity |
| Disp | 0-4 | Display mode on UC4 |
| Min | 0-127 | Minimum output value |
| Max | 0-127 | Maximum output value |

**Push/Green Button:**
| Param | Range | Description |
|-------|-------|-------------|
| Chan | 1-16 | MIDI channel |
| Type | Note/CC/PrGC/AFtt/Off | Message type |
| Note/CC | 0-127 | Note or CC number |
| Lower | 0-127 | Value on release |
| Upper | 0-127 | Value on press |
| Mode | Momentary/Toggle/Step | Button behavior |

**Fader:**
| Param | Range | Description |
|-------|-------|-------------|
| Chan | 1-16 | MIDI channel |
| CC | 0-127 | Controller number |
| Type | CCAb/PrGC/Pbnd/AFtt | Message type |
| Min | 0-127 | Value at bottom |
| Max | 0-127 | Value at top |

**Fader 9:**
| Param | Range | Description |
|-------|-------|-------------|
| Chan | 1-16 | MIDI channel |
| CC | 0-127 | Controller number |

---

## Overview Mode — See Everything

Grid view showing all controls across all 8 groups simultaneously.

### Layout

```
┌─────────────────────────────────────────────────────────────────┐
│ [All] [Encoders] [Push] [Green] [Faders]                        │
│                                                                  │
│ [✓ Concurrent (3)] [□ Mutually-Exclusive (12)]                  │
├─────────────────────────────────────────────────────────────────┤
│         │ GrP1    │ GrP2    │ GrP3    │ ...     │ GrP8    │     │
├─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────│
│ Enc 1   │ 1:CC 1  │ 1:CC 1  │ 1:CC 1  │         │ 1:CC 1  │     │
│ Enc 2   │ 1:CC 2  │ 1:CC 2  │ 1:CC 2  │         │ 1:CC 2  │     │
│ Enc 3   │ 1:CC 3  │⚠1:CC 64 │ 1:CC 3  │         │ 1:CC 3  │     │
│ ...     │         │         │         │         │         │     │
└─────────────────────────────────────────────────────────────────┘
```

### Tabs

| Tab | What It Shows |
|-----|---------------|
| **All** | All control types stacked vertically |
| **Encoders** | 8×8 grid of encoders only |
| **Push** | 8×8 grid of push buttons |
| **Green** | 8×8 grid of green buttons |
| **Faders** | 8×8 fader grid + fader9 row |

### Cell Format

Each cell shows: `Channel:Type CC#`

| Display | Meaning |
|---------|---------|
| `1:CC 64` | Channel 1, CC, number 64 |
| `2:Nt 60` | Channel 2, Note, number 60 |
| `3:PC 0` | Channel 3, Program Change, number 0 |
| `1:PB --` | Channel 1, Pitch Bend |
| `1:AT --` | Channel 1, Aftertouch |

### Cell Interactions

| Action | Result |
|--------|--------|
| **Single-click** | Select cell (green outline) |
| **Double-click** | Jump to Focused view |
| **Right-click** | Context menu + select |
| **Hover (in Quick Paste)** | Preview paste target |

### Conflict Highlighting

- **Amber ⚠️**: Conflicting assignment
- **Bright amber**: Concurrent conflict (fix these!)
- **Dim amber**: Mutually-exclusive conflict (usually OK)

---

## Quick Copy/Paste — Rapid Configuration

The fastest way to copy/paste in Overview mode. Press **Q** to activate.

### Quick Paste Toolbar

```
┌──────────────────────────────────────────────────────────────────┐
│ Mode [Off][Copy][Paste]   Scope [Cell][Column][Row]              │
│ Source: Enc G1.1          Ch [+1▾]  CC [0▾]   [Clear Source]     │
│ Status: Click to paste • 3 pasted                                │
└──────────────────────────────────────────────────────────────────┘
```

### Workflow

1. **Press Q** or click **Copy** — Enter Quick Paste mode
2. **Select scope**: Cell (1), Column (2), or Row (3)
3. **Click a cell** — Copies source and switches to Paste mode
4. **Click target cells** — Pastes with offsets applied
5. **Press Q** again — Exit Quick Paste mode

### Channel/CC Multipliers

The multipliers apply offsets based on the group difference between source and target.

**Formula:** `offset = (targetGroup - sourceGroup) × multiplier`

**Example with Ch=+1, CC=0:**

| Source | Target | Group Diff | Channel Offset |
|--------|--------|------------|----------------|
| G1 | G2 | 1 | +1 |
| G1 | G3 | 2 | +2 |
| G1 | G5 | 4 | +4 |

So copying from G1 Ch1 to G3 with Ch=+1 results in Ch3.

### Scope Options

| Scope | What's Copied | What's Pasted |
|-------|---------------|---------------|
| **Cell** | One control | One control |
| **Column** | All 8 controls in group | All 8 controls in target group |
| **Row** | One control from all 8 groups | Same row across 8 groups |

### Visual Feedback

- **Green highlight**: Source cells
- **Cyan outline**: Paste preview (hover)
- **Toast message**: Confirms each paste

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| **Q** | Toggle Quick Paste on/off |
| **1** | Cell scope |
| **2** | Column scope |
| **3** | Row scope |

---

## Setup Manager — Organize Your Setups

Manage entire setups: label, copy, swap, clear, reset, export, import.

### Opening Setup Manager

Click **[Manage Setups]** in the header.

### Setup Grid

```
┌─────────────────────────────────────────────────────────────────┐
│ Setup Manager                                            [×]   │
├─────────────────────────────────────────────────────────────────┤
│  1: Synth      2: Drums     3: (factory)  4: FX               │
│  5            6: Keys       7            8                     │
│  9            10           11           12                     │
│  13           14           15           16                     │
│  17: Ableton  18: Ableton                                      │
├─────────────────────────────────────────────────────────────────┤
│ [Label] [Clear] [Copy] [Swap] [Reset] [Export] [Import]        │
└─────────────────────────────────────────────────────────────────┘
```

### Visual Indicators

- **Accent border**: Setup has been modified
- **(factory)**: Setup matches factory defaults
- **Label text**: Custom label you've assigned

### Selection

- **Click**: Select one setup
- **Ctrl+Click**: Multi-select (add/remove from selection)
- **Click empty area**: Deselect all

### Operations

| Button | Requires | Action |
|--------|----------|--------|
| **Label** | 1 setup | Assign a name (stored in browser, not SysEx) |
| **Clear** | 1+ setups | Zero out all parameters |
| **Copy** | 1 setup | Duplicate to other slot(s) |
| **Swap** | 2 setups | Exchange two setups |
| **Reset** | 1+ setups | Restore factory defaults |
| **Export** | 1 setup | Save as single-setup JSON |
| **Import** | — | Load single-setup JSON to a slot |

### Copy Dialog

1. Select source setup
2. Click **Copy**
3. Select destination setup(s)
4. Choose whether to copy labels
5. Click **Copy**

### Swap Dialog

1. Select two setups (Ctrl+click), or
2. Select one, click **Swap**, select destination
3. Confirm swap

### Reset to Factory

1. Select setup(s) to reset
2. Click **Reset**
3. Confirm — restores from `factory_default.syx`

### Labels

Labels are stored in your browser (localStorage), not in the SysEx file. They appear in:
- Setup dropdown: "1: Synth"
- Setup Manager grid
- Export filenames (suggested)

---

## Conflict Detection

Automatically finds MIDI assignment conflicts.

### Conflict Types

| Type | Description | Severity |
|------|-------------|----------|
| **Concurrent** | Same message from controls active at the same time | ⚠️ Fix these! |
| **Mutually-Exclusive** | Same message in different groups | Usually OK |

### Understanding Conflicts

**Concurrent conflicts** happen when two controls in the same active domain send identical messages. Examples:
- Encoder 1 and Encoder 3 both send Ch1 CC64 (same group)
- Fader 1 and Green Button 1 both send Ch1 CC64 (same fader group)

**Mutually-exclusive conflicts** happen across different groups—only one can be active at a time, so usually intentional.

### Filter Chips

```
[✓ Concurrent (3)] [□ Mutually-Exclusive (12)]
```

Click to show/hide each conflict type in the grid.

### Conflict Key Format

```
Ch{channel} {type} {number}
```

Examples:
- `Ch1 CC 64`
- `Ch2 Note 60`
- `Ch1 PB` (pitch bend has no number)
- `Ch3 PC 5` (program change)

---

## Context Menu Copy & Paste

Right-click in Overview mode for advanced copy/paste operations.

### Context Menu

```
┌─────────────────────────────────────┐
│ Copy Control                        │
│ Copy Row (Enc 3 × 8 groups)         │
│ Copy Column (Group 2)               │
├─────────────────────────────────────┤
│ Paste                               │
│ Paste Special...                    │
└─────────────────────────────────────┘
```

### Copy Scopes

| Menu Item | What's Copied |
|-----------|---------------|
| **Copy Control** | Single cell (one control) |
| **Copy Row** | Same control index across all 8 groups |
| **Copy Column** | All controls in one group |

### Paste Special Dialog

Opens a dialog with transform options:

| Option | Range | Description |
|--------|-------|-------------|
| **Channel offset** | -15 to +15 | Shift MIDI channel |
| **CC/Number offset** | -127 to +127 | Shift CC or note number |
| **Auto-increment** | Any integer | Add sequential offset per target |
| **Wrap mode** | Clamp / Wrap | Out-of-range behavior |

### Paste Targets

| Target | Effect |
|--------|--------|
| **Selected cell** | Paste to one cell |
| **Entire column** | Paste down the column |
| **Entire row** | Paste across all 8 groups |

---

## Undo & Redo

Full edit history with intelligent coalescing.

### What's Tracked

- Individual parameter changes
- Batch operations (paste to row/column)
- Setup Manager operations (copy, swap, clear, reset)
- Label changes

### Coalescing

Rapid edits to the same parameter (within 1 second) are combined into one undo step. This means dragging a slider doesn't create 50 undo entries.

### Limits

- 100 undo steps maximum
- Oldest actions are dropped when limit reached

### Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z | Cmd+Shift+Z |
| Redo (alt) | Ctrl+Y | — |

---

## Session Persistence

Auto-saves your work to browser storage.

### How It Works

- Changes are saved every 2 seconds
- Survives page refresh and browser restart
- Only prompts to restore if you have unsaved changes

### Restore Dialog

When you open the editor with unsaved changes:

```
┌─────────────────────────────────────────┐
│  Restore Previous Session?              │
│                                         │
│  Found session from: 2 hours ago        │
│  Modified setups: 1, 3, 5               │
│                                         │
│  □ Don't ask again this session         │
│                                         │
│  [Discard]              [Restore]       │
└─────────────────────────────────────────┘
```

### When Session Clears

- After Export SysEx
- After Export JSON
- After Import (new file replaces session)
- After clicking Discard

---

## Keyboard Shortcuts

### Global

| Key | Action |
|-----|--------|
| Ctrl+Z | Undo |
| Ctrl+Shift+Z | Redo |
| Ctrl+Y | Redo (alternate) |

### Overview Mode

| Key | Action |
|-----|--------|
| ↑↓←→ | Move selection |
| Enter | Jump to Focused view for selected cell |
| Tab | Move right |
| Shift+Tab | Move left |
| Escape | Clear selection |
| Ctrl+C | Copy selected control |
| Ctrl+V | Paste to selected |

### Quick Paste

| Key | Action |
|-----|--------|
| Q | Toggle Quick Paste mode |
| 1 | Cell scope |
| 2 | Column scope |
| 3 | Row scope |

---

## Workflow Examples

### Create a Channel-Per-Group Setup

**Goal:** Group 1 = Channel 1, Group 2 = Channel 2, etc.

**Method (Quick Paste):**
1. Configure Group 1 exactly as you want (all controls on Ch1)
2. Switch to **Overview** → **Encoders** tab
3. Press **Q** → select **Column** scope
4. Set **Ch** to **+1**, **CC** to **0**
5. Click any cell in Group 1 (copies whole column)
6. Click Group 2, 3, 4... headers (pastes with channel offset)
7. Repeat for Push, Green, Faders tabs

**Time:** ~2 minutes for complete setup

### Sequential CCs Across a Row

**Goal:** Encoder 1 in all groups = CC1, CC2, CC3...CC8

**Method (Context Menu):**
1. Overview → Encoders tab
2. Right-click Enc 1, Group 1 → **Copy Control**
3. Right-click same cell → **Paste Special...**
4. Set **Auto-increment CC by:** 1
5. **Paste to:** Entire row
6. Click **Paste**

**Result:** G1=CC1, G2=CC2, G3=CC3...

### Prepare Setups for Live Show

1. **Label your setups:** Setup Manager → select setup → Label → "Opener", "Main", "Encore"
2. **Check for conflicts:** Overview mode → ensure Concurrent count is 0
3. **Export backup:** Export JSON → save as "live-show-backup.json"
4. **Export SysEx:** Export SysEx → send to UC4
5. **Test on hardware:** Verify all setups work correctly

### Share Configuration with Bandmate

1. **Export the setup:** Setup Manager → select setup → Export → save JSON
2. **Send the file:** Email, Dropbox, etc.
3. **They import:** Setup Manager → Import → select file → choose slot

---

## Validation & Troubleshooting

### "Invalid file size" Error

**Cause:** The file isn't a complete UC4 dump.

**Fix:** Use "Send All Setups" from UC4, not single-setup dump.

### Changes Don't Appear on UC4

**Checklist:**
1. Did you click **Export SysEx**? (JSON won't work)
2. Did you **send** the .syx file to UC4 via MIDI?
3. Is UC4 ready to receive? (Check manual for receive mode)

### Lost My Edits

1. **Refresh the page** — Look for restore dialog
2. **Check Downloads** — Look for exported files
3. **Check for JSON backup** — You did export JSON, right?

**Prevention:** Export JSON frequently as backup!

### Conflicts Everywhere

**This is normal for factory defaults!**

Factory config uses identical settings across groups intentionally.

- **Mutually-Exclusive:** Different groups, same message (OK)
- **Concurrent:** Same group, same message (fix these)

Use filter chips to show only Concurrent conflicts.

### Quick Paste Not Working

1. **Are you in Overview mode?** Quick Paste only works there.
2. **Is the toolbar visible?** Press Q to toggle.
3. **Did you set a source?** Click a cell in Copy mode first.
4. **Same control type?** Can't paste encoders to faders.

### Editor Feels Slow

1. Use individual tabs (Encoders, Faders) instead of All view
2. Close other browser tabs
3. Hard refresh: Ctrl+Shift+R (Windows) or Cmd+Shift+R (Mac)

### Tooltips Not Showing

1. **Wait 500ms** — There's a delay before tooltips appear
2. **Focused view only** — Tooltips only show in Focused view
3. **Hover over labels** — Not the input fields

---

## UC4 Parameter Reference

### Encoder Types

| Type | Description |
|------|-------------|
| **CCr1** | Relative mode 1 (1/127). Best for Ableton, Bitwig. |
| **CCr2** | Relative mode 2 (65/63). Alternative relative format. |
| **CCAb** | Absolute 7-bit CC. Standard 0-127 range. |
| **PrGC** | Program Change. Turn to increment/decrement. |
| **CCAh** | High-resolution 14-bit CC. Uses CC and CC+32. |
| **Pbnd** | Pitch Bend. Full -8192 to +8191 range. |
| **AFtt** | Channel Aftertouch. |

### Button Types

| Type | Description |
|------|-------------|
| **Note** | Standard MIDI note on/off. |
| **CC** | Control Change message. |
| **PrGC** | Program Change on press. |
| **AFtt** | Channel Aftertouch. |
| **Off** | Button disabled. |

### Button Modes

| Mode | Description |
|------|-------------|
| **Momentary** | Upper on press, Lower on release. |
| **Toggle** | Alternates between Upper and Lower. |
| **Step** | Increments through range. |

### Fader Types

| Type | Description |
|------|-------------|
| **CCAb** | Absolute CC. Standard fader mode. |
| **PrGC** | Program Change across range. |
| **Pbnd** | Pitch Bend. |
| **AFtt** | Channel Aftertouch. |

---

## Getting More Help

- **[? Guide]** button in editor → Opens this guide
- **[HOWTO.md](HOWTO.md)** → Practical workflow examples
- **[SPECIFICATION.md](SPECIFICATION.md)** → Technical details
- **UC4 Manual** → Official Faderfox documentation
