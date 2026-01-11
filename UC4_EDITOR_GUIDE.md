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
7. [Conflict Detection](#conflict-detection)
8. [Copy & Paste Operations](#copy--paste-operations)
9. [Undo & Redo](#undo--redo)
10. [Session Persistence](#session-persistence)
11. [Keyboard Shortcuts](#keyboard-shortcuts)
12. [Workflow Examples](#workflow-examples)
13. [Validation & Troubleshooting](#validation--troubleshooting)

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
│                             [Export JSON]  [↶ Undo] [↷ Redo]                   │
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
| **● Green dot** | All changes saved/exported |
| **● Amber dot** | You have unsaved changes! |
| **File size** | Confirms valid 100,640 byte file loaded |

---

## Working with Files

### Importing Your UC4 Configuration

**Step 1:** Dump your UC4 via MIDI
- Connect UC4 to computer via USB or MIDI
- Use a SysEx librarian (e.g., SysEx Librarian, MIDI-OX, your DAW)
- Request dump from UC4 (see UC4 manual)
- Save as .syx file

**Step 2:** Load into editor
```
┌──────────────────────────────────────┐
│  Click [Import SysEx]                │
│           ↓                          │
│  ┌────────────────────────────────┐  │
│  │  Select file:                  │  │
│  │                                │  │
│  │  📄 my_uc4_backup.syx          │  │
│  │     100,640 bytes  ✓           │  │
│  │                                │  │
│  └────────────────────────────────┘  │
│           ↓                          │
│  Editor loads all 18 setups          │
└──────────────────────────────────────┘
```

**Validation:** The status bar shows "100,640 bytes loaded" — this confirms a valid full dump.

### Exporting Back to UC4

```
┌──────────────────────────────────────┐
│  1. Make your edits                  │
│  2. Click [Export SysEx]             │
│  3. Save the .syx file               │
│  4. Send to UC4 via MIDI SysEx       │
└──────────────────────────────────────┘
```

**Important:** Exporting clears the "modified" indicator and your session backup.

---

### JSON Export/Import — Your Best Friend

JSON creates a **human-readable** backup of your entire configuration.

**Why use JSON?**

| Benefit | Description |
|---------|-------------|
| **Git-friendly** | Track changes with version control |
| **Readable** | Review exactly what changed |
| **Shareable** | Send configs to bandmates |
| **Backup** | Safety net before experiments |
| **Smaller** | ~50KB vs 100KB for .syx |

**JSON Structure:**
```json
{
  "version": 1,
  "exportDate": "2026-01-11T20:30:00.000Z",
  "setups": [
    {
      "index": 0,
      "groups": [
        {
          "index": 0,
          "name": "GrP1",
          "encoders": [
            { "channel": 1, "type": 2, "cc": 1, "min": 0, "max": 127, "acc": 1, "display": 1 },
            { "channel": 1, "type": 2, "cc": 2, "min": 0, "max": 127, "acc": 1, "display": 1 },
            ...
          ],
          "pushButtons": [...],
          "greenButtons": [...],
          "faders": [...],
          "fader9": { "channel": 1, "cc": 9 }
        },
        ... // Groups 2-8
      ]
    },
    ... // Setups 2-18
  ]
}
```

**Workflow tip:** Export JSON before making big changes. If something goes wrong, import the JSON to restore.

---

## Navigating Setups & Groups

### The UC4's Structure

```
UC4 Memory Structure
│
├── Setup 1
│   ├── Group 1: 8 encoders, 8 push, 8 green, 8 faders, fader9
│   ├── Group 2: 8 encoders, 8 push, 8 green, 8 faders, fader9
│   ├── ...
│   └── Group 8: 8 encoders, 8 push, 8 green, 8 faders, fader9
│
├── Setup 2
│   └── (same structure)
│
├── ...
│
└── Setup 18
    └── (same structure)

Total: 18 setups × 8 groups × 33 controls = 4,752 editable controls!
```

### Domain Mapping (Critical to Understand!)

The UC4 has **two independent group selectors** on the hardware:

```
┌─────────────────────────────────────────────────────────────────┐
│                      ENCODER DOMAIN                             │
│                                                                 │
│   Hardware: Hold Shift + Press Encoder 1-8                      │
│                                                                 │
│   Controls affected:                                            │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐     ┌─────────┐          │
│   │Encoder 1│ │Encoder 2│ │Encoder 3│ ... │Encoder 8│          │
│   │(+ Push) │ │(+ Push) │ │(+ Push) │     │(+ Push) │          │
│   └─────────┘ └─────────┘ └─────────┘     └─────────┘          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    FADER/BUTTON DOMAIN                          │
│                                                                 │
│   Hardware: Hold Shift + Press Green Button 1-8                 │
│                                                                 │
│   Controls affected:                                            │
│   ┌────────┐ ┌────────┐ ┌────────┐     ┌────────┐ ┌────────┐   │
│   │Fader 1 │ │Fader 2 │ │Fader 3 │ ... │Fader 8 │ │Fader 9 │   │
│   └────────┘ └────────┘ └────────┘     └────────┘ └────────┘   │
│   ┌────────┐ ┌────────┐ ┌────────┐     ┌────────┐              │
│   │Green 1 │ │Green 2 │ │Green 3 │ ... │Green 8 │              │
│   └────────┘ └────────┘ └────────┘     └────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**This is why the editor has TWO group selectors!**

### 🔗 Link Groups

When you want both domains on the same group (common for channel-per-group setups):

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   [🔗 Link ✓]     ← Check this box                              │
│                                                                 │
│   Encoder Grp  [1] [2] [③] [4] [5] [6] [7] [8]   GrP3          │
│   Fader/Btn    [1] [2] [③] [4] [5] [6] [7] [8]   GrP3          │
│                      ↑                                          │
│                 Both sync!                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Now clicking **any** group tab changes **both** selectors.

---

## Focused View — Detailed Editing

The Focused view shows full parameter cards for all controls in the selected groups.

```
┌─ FADERS ───────────────────────────────────────────── GrP1 ─────┐
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ Fader 1     │  │ Fader 2     │  │ Fader 3     │   ...       │
│  │ F1.1        │  │ F1.2        │  │ F1.3        │             │
│  │─────────────│  │─────────────│  │─────────────│             │
│  │ Chan [1 ▾]  │  │ Chan [1 ▾]  │  │ Chan [1 ▾]  │             │
│  │ CC   [ 1  ] │  │ CC   [ 2  ] │  │ CC   [ 3  ] │             │
│  │ Type [CCAb] │  │ Type [CCAb] │  │ Type [CCAb] │             │
│  │ Min  [ 0  ] │  │ Min  [ 0  ] │  │ Min  [ 0  ] │             │
│  │ Max  [127 ] │  │ Max  [127 ] │  │ Max  [127 ] │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─ GREEN BUTTONS ────────────────────────────────────── GrP1 ─────┐
│  ┌─────────────┐  ┌─────────────┐  ...                         │
│  │ Green 1     │  │ Green 2     │                              │
│  │ Chan [1 ▾]  │  │ Chan [1 ▾]  │                              │
│  │ CC   [64  ] │  │ CC   [65  ] │                              │
│  │ Type [Note] │  │ Type [Note] │                              │
│  │ Lower[ 0  ] │  │ Lower[ 0  ] │                              │
│  │ Upper[127 ] │  │ Upper[127 ] │                              │
│  │ Mode [Gate] │  │ Mode [Gate] │                              │
│  └─────────────┘  └─────────────┘                              │
└─────────────────────────────────────────────────────────────────┘

┌─ ENCODERS ─────────────────────────────────────────── GrP1 ─────┐
│  (8 encoder cards - these use the ENCODER group selector)      │
└─────────────────────────────────────────────────────────────────┘

┌─ PUSH BUTTONS ─────────────────────────────────────── GrP1 ─────┐
│  (8 push button cards - buttons under encoders)                │
└─────────────────────────────────────────────────────────────────┘

┌─ FADER 9 ──────────────────────────────────────────── GrP1 ─────┐
│  ┌─────────────┐                                               │
│  │ Fader 9     │  (Expression pedal / 9th fader)               │
│  │ Chan [1 ▾]  │                                               │
│  │ CC   [ 11 ] │                                               │
│  └─────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
```

### Section Headers — Know What You're Editing!

Each section shows the **group name in bright cyan**:

```
┌─ ENCODERS ──────────────────────────────── GrP3 ────┐
                                              ↑
                                    Bright accent color
                                    Matches UC4 display
```

This prevents accidentally editing the wrong group!

### Parameter Reference

**Encoder Parameters:**
| Param | Values | Description |
|-------|--------|-------------|
| Chan | 1-16 | MIDI channel |
| CC | 0-127 | CC number (or note for some types) |
| Type | CCr1, CCr2, CCAb, PrGC, CCAh, Pbnd, AFtt | Message type |
| Acc | Acc0-3 | Acceleration (higher = more sensitive) |
| Disp | OFF, Std, bPoL | LED ring display mode |
| Min | 0-127 | Minimum value |
| Max | 0-127 | Maximum value |

**Encoder Types Explained:**
| Type | Description |
|------|-------------|
| CCr1 | Relative mode 1 (64 = no change) |
| CCr2 | Relative mode 2 (0 = no change) |
| CCAb | Absolute CC (standard 0-127) |
| PrGC | Program Change (sends PC messages) |
| CCAh | 14-bit high-resolution CC |
| Pbnd | Pitch Bend |
| AFtt | Aftertouch |

**Button Parameters:**
| Param | Values | Description |
|-------|--------|-------------|
| Chan | 1-16 | MIDI channel |
| CC | 0-127 | CC or note number |
| Type | Note, CC, CC Toggle, Prog Chg, etc. | Message type |
| Lower | 0-127 | Value when released / off |
| Upper | 0-127 | Value when pressed / on |
| Mode | Gate, Toggle | Momentary vs latching |

**Fader Parameters:**
| Param | Values | Description |
|-------|--------|-------------|
| Chan | 1-16 | MIDI channel |
| CC | 0-127 | CC number |
| Type | CCAb, PrGC, Pbnd, AFtt | Message type |
| Min | 0-127 | Value at bottom position |
| Max | 0-127 | Value at top position |

---

## Overview Mode — See Everything

Click **[Overview]** to see all 64 controls in an 8×8 grid.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   [ Encoders ] [ Push Buttons ] [ Green Buttons ] [ Faders ]    │
│        ↑                                                        │
│   Tab selection                                                 │
│                                                                 │
│   [✓ Concurrent (3)] [Mutually-Exclusive (12)]   ← Filters     │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│           │ Group 1 │ Group 2 │ Group 3 │ Group 4 │ ...        │
│           │  GrP1   │  GrP2   │  GrP3   │  GrP4   │            │
│  ─────────┼─────────┼─────────┼─────────┼─────────┼────        │
│  Enc 1    │ 1:CC 1  │ 1:CC 1  │ 1:CC 1  │⚠1:CC 1 │            │
│  Enc 2    │ 1:CC 2  │ 1:CC 2  │ 1:CC 2  │ 1:CC 2  │            │
│  Enc 3    │ 1:CC 3  │ 1:CC 3  │ 1:CC 3  │ 1:CC 3  │            │
│  Enc 4    │ 1:CC 4  │ 1:CC 4  │ 1:CC 4  │ 1:CC 4  │            │
│  Enc 5    │ 1:CC 5  │ 1:CC 5  │ 1:CC 5  │ 1:CC 5  │            │
│  Enc 6    │ 1:CC 6  │ 1:CC 6  │ 1:CC 6  │ 1:CC 6  │            │
│  Enc 7    │ 1:CC 7  │ 1:CC 7  │ 1:CC 7  │ 1:CC 7  │            │
│  Enc 8    │ 1:CC 8  │ 1:CC 8  │ 1:CC 8  │ 1:CC 8  │            │
│  ─────────┴─────────┴─────────┴─────────┴─────────┴────        │
│                                                                 │
│   ⚠ Conflicts:                                                  │
│   ⚠ Ch1 CC 1: Enc G1.1 (CC), Enc G4.1 (CC)                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cell Format

Each cell shows: `Channel:Type CC#`

```
┌─────────┐
│ 1:CC 64 │  ← Channel 1, CC type, CC# 64
└─────────┘

┌─────────┐
│ 2:Nt 60 │  ← Channel 2, Note type, Note 60 (Middle C)
└─────────┘

┌─────────┐
│⚠1:CC 64│  ← Warning icon = conflict detected
└─────────┘
```

### Tab Navigation

```
[ Encoders ] [ Push Buttons ] [ Green Buttons ] [ Faders ]
     ↓
Click to switch between control types
```

### Interaction

| Action | Result |
|--------|--------|
| **Single-click** | Select cell (green outline) |
| **Double-click** | Jump to Focused view for that control |
| **Right-click** | Open copy/paste context menu |
| **Arrow keys** | Move selection |
| **Enter** | Jump to Focused view |

---

## Conflict Detection

The editor automatically detects when two controls send the **same MIDI message**.

### Conflict Types

```
┌─────────────────────────────────────────────────────────────────┐
│  CONCURRENT CONFLICTS (Serious!)                                │
│                                                                 │
│  Two controls that are ACTIVE AT THE SAME TIME send the        │
│  same MIDI message.                                             │
│                                                                 │
│  Example: Encoder 1 in Group 1 AND Fader 1 in Group 1          │
│           both send Ch1 CC 64                                   │
│                                                                 │
│  ⚠ These will fight each other!                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  MUTUALLY-EXCLUSIVE CONFLICTS (Usually OK)                      │
│                                                                 │
│  Two controls in DIFFERENT GROUPS send the same message.        │
│  Only one group is active at a time, so they won't conflict.    │
│                                                                 │
│  Example: Encoder 1 in Group 1 AND Encoder 1 in Group 4        │
│           both send Ch1 CC 64                                   │
│                                                                 │
│  ✓ This is often intentional (same layout, different group)    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Conflict Filters

```
[✓ Concurrent (3)]  [□ Mutually-Exclusive (12)]
       ↑                      ↑
   Checked = shown       Unchecked = hidden
   (bright amber)         (dimmed)
```

- **Default:** Concurrent ON, Mutually-Exclusive OFF
- Click chips to toggle visibility
- Numbers show count of each type

### Conflict Highlighting

In the Overview grid:

```
┌─────────────┐     ┌─────────────┐
│⚠ 1:CC 64   │     │⚠ 1:CC 64   │
│ [amber bg] │     │ [dim amber] │
└─────────────┘     └─────────────┘
   Concurrent        Mutually-Excl
```

### Conflict Panel

Below the grid, a panel lists all visible conflicts:

```
┌─────────────────────────────────────────────────────────────────┐
│ ⚠ Conflicts:                                                    │
│ ⚠ Ch1 CC 64: Enc G1.1 (CC), Fad G1.1 (CC)                      │
│ ⚠ Ch1 CC 65: Enc G1.2 (CC), Fad G1.2 (CC)                      │
│ ⚠ Ch2 PC 0-127: Enc G2.1 (PrGC), Enc G5.1 (PrGC)              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Copy & Paste Operations

The editor supports powerful copy/paste in Overview mode.

### Right-Click Context Menu

```
Right-click any cell in Overview:

┌─────────────────────────────────┐
│ Copy Control                    │
│ Copy Row (3 × 8 groups)         │
│ Copy Column (Group 2)           │
├─────────────────────────────────┤
│ Paste                           │
│ Paste Special...                │
└─────────────────────────────────┘
```

### Copy Scopes

| Scope | What's Copied |
|-------|---------------|
| **Copy Control** | Single cell — all parameters for one control |
| **Copy Row** | One control across all 8 groups (e.g., Encoder 3 in all groups) |
| **Copy Column** | All controls in one group (e.g., all 8 encoders in Group 2) |

### Basic Paste

Select a cell → Right-click → **Paste**

The copied control's parameters replace the target cell.

### Paste Special — Power Features!

```
┌─ Paste Special ─────────────────────────────────────────────────┐
│                                                                 │
│  Source: Enc G1.3 (Ch1 CC 45)                                  │
│                                                                 │
│  Paste to:                                                      │
│    ○ Current cell                                               │
│    ○ Entire row (all groups)                                    │
│    ○ Entire column (Group 2)                                    │
│                                                                 │
│  Transforms:                                                    │
│    Channel offset:    [ 0 ▾]  (-15 to +15)                     │
│    CC/Number offset:  [ 0 ▾]  (-127 to +127)                   │
│    [✓] Auto-increment CC by: [ 1 ]                             │
│    Out-of-range:      [Clamp ▾]  (Clamp / Wrap)                │
│                                                                 │
│                              [Cancel]  [Paste]                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Transform Examples

**Channel Offset:**
```
Source: Ch1 CC 64
Offset: +2
Result: Ch3 CC 64
```

**CC Offset:**
```
Source: Ch1 CC 64
Offset: +10
Result: Ch1 CC 74
```

**Auto-Increment (paste to row):**
```
Source: Ch1 CC 1
Auto-increment by: 1
Paste to row:
  Group 1: CC 1
  Group 2: CC 2
  Group 3: CC 3
  ...
  Group 8: CC 8
```

**Wrap vs Clamp:**
```
Source: Ch15, Offset: +3

Clamp: Ch16 (stops at max)
Wrap:  Ch2  (wraps around: 15+3=18 → 18-16=2)
```

---

## Undo & Redo

Every edit can be reversed.

### How It Works

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Edit stack:                                                   │
│                                                                │
│    [Initial] → [Edit 1] → [Edit 2] → [Edit 3]                 │
│                                          ↑                     │
│                                       Current                  │
│                                                                │
│  Click Undo: ←←←                                               │
│  Click Redo: →→→                                               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Coalescing

Rapid edits to the same parameter are **combined** into one undo step:

```
Typing CC value: 1 → 12 → 127

Without coalescing: 3 undo steps
With coalescing:    1 undo step (if typed within 1 second)
```

### Batch Operations

Copy/paste to multiple cells creates a **single undo step**:

```
Paste to entire row (8 cells) → 1 undo step to reverse all 8
```

### Keyboard Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z or Ctrl+Y | Cmd+Shift+Z |

---

## Session Persistence

Your work is **automatically saved** to browser storage.

### How It Works

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  You edit → 2 seconds pass → Auto-saved to localStorage        │
│                                                                │
│  You close browser → Reopen editor → Restore dialog appears   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Restore Dialog

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Restore Previous Session?                                     │
│                                                                 │
│   Found auto-saved session from:                                │
│   January 11, 2026 at 8:45 PM                                  │
│                                                                 │
│   [Discard]                    [Restore]                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When Session Clears

- **Export SysEx** — you've saved your work
- **Export JSON** — you've saved your work
- **Import new file** — starting fresh
- **Click Discard** — explicitly abandoning

---

## Keyboard Shortcuts

### Global Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z | Cmd+Shift+Z |
| Redo (alt) | Ctrl+Y | — |

### Overview Mode Shortcuts

| Action | Key |
|--------|-----|
| Move selection | Arrow keys |
| Move selection (alt) | Tab / Shift+Tab |
| Jump to Focused view | Enter |
| Clear selection | Escape |
| Copy selected | Ctrl+C / Cmd+C |
| Paste to selected | Ctrl+V / Cmd+V |
| Close context menu | Escape |

---

## Workflow Examples

### Example 1: Quick CC Remap

**Goal:** Change Encoder 1 from CC 1 to CC 74 (filter cutoff)

```
1. Open editor (factory defaults load)
2. Focused view → find Encoder 1 card
3. Change CC field: 1 → 74
4. Export SysEx → send to UC4
```

Time: ~30 seconds

### Example 2: Set Up 8 Channels

**Goal:** Each group on a different MIDI channel (Group 1 = Ch1, Group 2 = Ch2, etc.)

```
1. Overview mode → Encoders tab
2. Click Group 1, Encoder 1 cell
3. Right-click → Copy Column
4. Click Group 2, Encoder 1 cell
5. Right-click → Paste Special
   - Channel offset: +1
   - Paste to: Entire column
6. Repeat for Groups 3-8 (or use row paste with auto-increment)
7. Do same for Faders, Green Buttons
8. Export
```

Time: ~3 minutes

### Example 3: Clone a Setup

**Goal:** Copy Setup 1 to Setup 2 and modify

```
1. Edit Setup 1 as desired
2. Export JSON
3. Edit JSON: duplicate setup 0 data to setup 1
4. Import JSON
5. Select Setup 2 → make modifications
6. Export SysEx
```

### Example 4: Validate Before Gig

**Goal:** Check for conflicts before performing

```
1. Import your .syx
2. Overview mode
3. Check [✓ Concurrent] filter
4. If conflicts shown:
   - Review conflict panel
   - Click conflicting cells to investigate
   - Fix in Focused view
5. Export clean .syx
```

---

## Validation & Troubleshooting

### File Size Validation

```
Valid UC4 dump:   100,640 bytes  ✓
Wrong size:       [any other]    ✗ "Invalid file size" error
```

### Status Indicators

| Status | Meaning |
|--------|---------|
| "100,640 bytes loaded" | Valid file imported |
| "No data loaded" | No file imported yet |
| "● Modified" | Unsaved changes exist |

### Common Issues

**"Invalid file size" on import**
- Ensure you dumped ALL setups from UC4, not just one
- Check your SysEx librarian settings for "full dump"

**Changes not appearing on UC4**
- Did you export .syx? (Not just JSON)
- Did you send SysEx to UC4? (Export only saves file)
- Is UC4 in receive mode? (Check UC4 manual)

**Lost my edits**
- Check for restore dialog on page load
- Did you export before closing?
- JSON backups are your friend!

**Conflicts everywhere on factory dump**
- Factory defaults intentionally duplicate settings across groups
- This is normal — use Mutually-Exclusive filter to hide
- Only Concurrent conflicts need attention

### Verifying Your Configuration

```
Before export checklist:

[✓] Check Overview for unexpected conflicts
[✓] Spot-check a few controls in Focused view
[✓] Export JSON as backup
[✓] Export SysEx
[✓] Test on UC4 before the gig!
```

---

## Tips & Best Practices

1. **Always export JSON before major changes** — it's your safety net

2. **Use Link mode** when building channel-per-group layouts

3. **Check conflicts before performing** — concurrent conflicts mean two controls fight each other

4. **Name your groups** on the UC4 hardware — the editor displays these names

5. **Use Paste Special transforms** for repetitive setups — much faster than manual editing

6. **Keep your .syx and .json files** together in a folder with the date

7. **Test on hardware** after making significant changes — the editor can't catch everything

8. **Use Overview for big-picture checks**, Focused for detailed edits

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────┐
│  UC4 EDITOR QUICK REFERENCE                                     │
├─────────────────────────────────────────────────────────────────┤
│  IMPORT:     Load .syx from UC4 dump or .json backup           │
│  EXPORT:     Save .syx for UC4, .json for backup/git           │
│  SETUP:      18 total, independent configurations               │
│  GROUPS:     8 per setup, selected via Shift+Encoder/Green     │
│  LINK:       Sync encoder & fader group selectors              │
├─────────────────────────────────────────────────────────────────┤
│  FOCUSED:    Edit individual parameters                         │
│  OVERVIEW:   See 8×8 grid, copy/paste, find conflicts          │
├─────────────────────────────────────────────────────────────────┤
│  Ctrl+Z      Undo                                               │
│  Ctrl+Y      Redo                                               │
│  Ctrl+C      Copy (in Overview)                                 │
│  Ctrl+V      Paste (in Overview)                                │
│  Arrows      Navigate grid                                      │
│  Enter       Jump to Focused                                    │
│  Escape      Clear selection                                    │
│  Right-click Context menu                                       │
├─────────────────────────────────────────────────────────────────┤
│  ⚠ Concurrent    = Active at same time (fix these!)            │
│  ⚠ Mut-Excl      = Different groups (usually OK)               │
│  ● Amber dot     = Unsaved changes                              │
│  Session restore = Auto-saved, offered on reload               │
└─────────────────────────────────────────────────────────────────┘
```

---

*Built for the Faderfox UC4. Not affiliated with Faderfox.*
