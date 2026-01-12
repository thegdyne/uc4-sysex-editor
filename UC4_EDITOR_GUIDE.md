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
8. [Conflict Detection](#conflict-detection)
9. [Context Menu Copy & Paste](#context-menu-copy--paste)
10. [Undo, Redo & Reset](#undo-redo--reset)
11. [Session Persistence](#session-persistence)
12. [Keyboard Shortcuts](#keyboard-shortcuts)
13. [Workflow Examples](#workflow-examples)
14. [Validation & Troubleshooting](#validation--troubleshooting)

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
│                             [Export JSON]  [↶ Undo] [↷ Redo] [⟲ Reset]         │
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
| **⟲ Reset** | Restore to originally imported SysEx |

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
│                      FADER DOMAIN                               │
│                                                                 │
│   Hardware: Hold Shift + Press Green Button 1-8                 │
│                                                                 │
│   Controls affected:                                            │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐     ┌─────────┐          │
│   │ Fader 1 │ │ Fader 2 │ │ Fader 3 │ ... │ Fader 8 │          │
│   │(+Green) │ │(+Green) │ │(+Green) │     │(+Green) │          │
│   └─────────┘ └─────────┘ └─────────┘     └─────────┘          │
│                         + Fader 9                               │
└─────────────────────────────────────────────────────────────────┘
```

### Link Mode

Click **🔗 Link** to synchronize both domains. When linked, changing one group changes both.

**Use cases:**
- **Linked:** All controls follow one group (simpler)
- **Unlinked:** Encoders on Group 1, Faders on Group 5 (advanced)

---

## Focused View — Detailed Editing

The default view. Edit every parameter for all controls in the selected group(s).

### Section Order

Controls are displayed in this order (same as Overview):

1. **Faders 1-8** — Main faders
2. **Fader 9** — Master/special fader
3. **Green Buttons** — Below faders on hardware
4. **Encoders** — Rotary encoders
5. **Push Buttons** — Press-down on encoders

### Control Cards

Each control shows all its parameters:

```
┌─ ENCODERS ──────────────────────────────── GrP1 ────┐
                                              ↑
                                    Bright accent color
                                    Matches UC4 display
```

### Encoder Types Explained

| Type | Description |
|------|-------------|
| CCr1 | Relative mode 1 (64 = no change) |
| CCr2 | Relative mode 2 (0 = no change) |
| CCAb | Absolute CC (standard 0-127) |
| PrGC | Program Change |
| CCAh | 14-bit high-resolution CC |
| Pbnd | Pitch Bend |
| AFtt | Aftertouch |

---

## Overview Mode — See Everything

Click **[Overview]** to see all controls in an 8-column grid (one column per group).

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   [ All ] [ Encoders ] [ Push ] [ Green ] [ Faders ]           │
│                                                                 │
│   [✓ Concurrent (3)] [Mutually-Exclusive (12)]   ← Filters     │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  Quick Paste:  [Off] [Copy] [Paste]   Scope: [Cell][Col][Row]  │
│  Source: Fad Column G1 (8)    Ch: [+1]  CC: [0]                │
├─────────────────────────────────────────────────────────────────┤
│           │ Group 1 │ Group 2 │ Group 3 │ Group 4 │ ...        │
│           │  GrP1   │  GrP2   │  GrP3   │  GrP4   │            │
│  ─────────┼─────────┼─────────┼─────────┼─────────┼────        │
│  Fader 1  │ 1:CC 32 │ 1:CC 32 │ 1:CC 32 │ 1:CC 32 │            │
│  Fader 2  │ 1:CC 33 │ 1:CC 33 │ 1:CC 33 │ 1:CC 33 │            │
│  ...      │         │         │         │         │            │
├─────────────────────────────────────────────────────────────────┤
│  Fader 9  │ 1:CC 40 │ 1:CC 40 │ 1:CC 40 │ 1:CC 40 │            │
├─────────────────────────────────────────────────────────────────┤
│  Green 1  │ 1:Nt 36 │ 1:Nt 36 │⚠1:Nt 36│ 1:Nt 36 │            │
│  ...      │         │         │         │         │            │
└─────────────────────────────────────────────────────────────────┘
```

### Tab Filters

| Tab | Shows |
|-----|-------|
| **All** | All control types stacked vertically |
| **Encoders** | Encoders only |
| **Push** | Push buttons only |
| **Green** | Green buttons only |
| **Faders** | Faders 1-8 and Fader 9 |

### Cell Format

Each cell shows: `Channel:Type Value`

```
┌─────────┐
│ 1:CC 64 │  ← Channel 1, CC type, CC# 64
└─────────┘

┌─────────┐
│⚠1:CC 64│  ← Warning icon = conflict detected
└─────────┘
```

### Interaction

| Action | Result |
|--------|--------|
| **Single-click** | Select cell (green outline) |
| **Double-click** | Jump to Focused view for that control |
| **Right-click** | Open context menu |
| **Arrow keys** | Move selection |
| **Enter** | Jump to Focused view |

---

## Quick Copy/Paste — Rapid Configuration

The Quick Paste toolbar enables rapid batch configuration. Press **Q** to activate.

### The Toolbar

```
┌─────────────────────────────────────────────────────────────────┐
│ Mode: [Off] [Copy] [Paste]   Scope: [Cell] [Column] [Row]      │
│ Source: Fad Column G1 (8)    Ch: [0 ▾]   CC: [0 ▾]             │
│ Click to paste • 3 pasted                                       │
└─────────────────────────────────────────────────────────────────┘
```

### How It Works

1. **Press Q** — Enter Copy mode
2. **Set Scope** — Cell, Column (group), or Row
3. **Set Multipliers** — Ch and CC offset multipliers
4. **Click source** — Highlights amber, switches to Paste mode
5. **Click targets** — Paste with automatic offset calculation
6. **Press Escape** — Exit and clear

### Relational Offset System

The key feature: offsets are **calculated automatically** based on position difference.

**Formula:**
```
offset = (target position - source position) × multiplier
```

**Example with Ch multiplier = +1:**

| Copy From | Paste To | Calculation | Result |
|-----------|----------|-------------|--------|
| G1 (Ch 1) | G2 | 1 + (2-1)×1 | Ch 2 |
| G1 (Ch 1) | G4 | 1 + (4-1)×1 | Ch 4 |
| G1 (Ch 1) | G8 | 1 + (8-1)×1 | Ch 8 |

**With multiplier = 0:** Exact copy, no offset applied.

**With multiplier = +2:** Double the offset (useful for CC blocks).

### Visual Feedback

| Color | Meaning |
|-------|---------|
| **Amber outline** | Source — what you copied |
| **Teal dashed outline** | Target — where you'll paste (on hover) |
| **Flash animation** | Just pasted successfully |

### Scopes Explained

| Scope | Copies | Pastes To |
|-------|--------|-----------|
| **Cell** | Single control | Single control |
| **Column** | All 8 controls in a group | Target group |
| **Row** | One control across all 8 groups | Same row position |

### Quick Paste Keyboard Shortcuts

| Key | Action |
|-----|--------|
| **Q** | Toggle mode: Off → Copy → Paste → Off |
| **1** | Set scope to Cell |
| **2** | Set scope to Column |
| **3** | Set scope to Row |
| **Escape** | Exit Quick Paste, clear source |

### Common Quick Paste Workflows

**8-Channel Mixer Setup:**
```
1. Configure G1 completely (Ch 1, your CCs)
2. Press Q, set Scope = Column, Ch = +1, CC = 0
3. Click G1 (copy)
4. Click G2, G3, G4, G5, G6, G7, G8
5. Result: Each group has incrementing channel
6. Press Escape
```

**CC Blocks of 8:**
```
1. Configure G1 faders with CC 0-7
2. Press Q, Scope = Column, Ch = 0, CC = +8
3. Click G1, then G2, G3, etc.
4. Result: G2 = CC 8-15, G3 = CC 16-23, etc.
```

**Duplicate Exactly:**
```
1. Leave Ch = 0, CC = 0
2. Copy and paste anywhere
3. Exact duplicate regardless of position
```

---

## Conflict Detection

The editor automatically detects when two controls send the **same MIDI message**.

### Conflict Types

**CONCURRENT CONFLICTS (Serious!)**
Two controls that are ACTIVE AT THE SAME TIME send the same MIDI message. These will fight each other!

**MUTUALLY-EXCLUSIVE CONFLICTS (Usually OK)**
Two controls in DIFFERENT GROUPS send the same message. Only one group is active at a time, so they won't conflict in practice.

### Conflict Filters

```
[✓ Concurrent (3)]  [□ Mutually-Exclusive (12)]
       ↑                      ↑
   Checked = shown       Unchecked = hidden
```

---

## Context Menu Copy & Paste

Right-click any cell in Overview for additional copy/paste options.

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
| **Copy Row** | One control across all 8 groups |
| **Copy Column** | All controls in one group |

### Paste Special — Power Features

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
└─────────────────────────────────────────────────────────────────┘
```

---

## Undo, Redo & Reset

### Undo & Redo

Every edit can be reversed. Rapid edits to the same parameter are **combined** into one undo step.

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z | Cmd+Shift+Z |
| Redo (alt) | Ctrl+Y | — |

### Reset

The **⟲ Reset** button restores everything to the originally imported SysEx:

- Reverts all changes since import
- Clears undo/redo history
- Clears Quick Paste source
- Shows confirmation dialog first

**Use case:** Made a mess? Reset and start fresh without re-importing.

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

### Quick Paste Shortcuts

| Action | Key |
|--------|-----|
| Toggle Quick Paste mode | Q |
| Set scope to Cell | 1 |
| Set scope to Column | 2 |
| Set scope to Row | 3 |
| Exit Quick Paste | Escape |

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

### Example 2: Set Up 8 Channels with Quick Paste

**Goal:** Each group on a different MIDI channel (Group 1 = Ch1, Group 2 = Ch2, etc.)

```
1. Configure Group 1 completely in Focused view
2. Switch to Overview
3. Press Q (Quick Paste)
4. Set Scope = Column, Ch = +1, CC = 0
5. Click any cell in Group 1 (copies entire column)
6. Click Group 2, 3, 4, 5, 6, 7, 8
7. Press Escape
8. Export
```

Time: ~1 minute

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

3. **Use Quick Paste (Q)** for repetitive setups — much faster than context menu

4. **Check conflicts before performing** — concurrent conflicts mean two controls fight each other

5. **Name your groups** on the UC4 hardware — the editor displays these names

6. **Keep your .syx and .json files** together in a folder with the date

7. **Test on hardware** after making significant changes — the editor can't catch everything

8. **Use Overview for big-picture checks**, Focused for detailed edits

9. **Reset button** is your friend if you make a mess — restores to imported state

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
│  QUICK PASTE (press Q to activate):                             │
│    Q         Toggle mode (Off → Copy → Paste → Off)            │
│    1 / 2 / 3 Cell / Column / Row scope                         │
│    Click     Copy (in Copy mode) or Paste (in Paste mode)      │
│    Escape    Exit Quick Paste                                   │
│    Ch/CC     Offset multiplier (0 = exact copy)                │
├─────────────────────────────────────────────────────────────────┤
│  Ctrl+Z      Undo                                               │
│  Ctrl+Y      Redo                                               │
│  Ctrl+C      Copy (in Overview)                                 │
│  Ctrl+V      Paste (in Overview)                                │
│  Arrows      Navigate grid                                      │
│  Enter       Jump to Focused                                    │
│  Escape      Clear selection / Exit Quick Paste                 │
│  Right-click Context menu                                       │
├─────────────────────────────────────────────────────────────────┤
│  ⚠ Concurrent    = Active at same time (fix these!)            │
│  ⚠ Mut-Excl      = Different groups (usually OK)               │
│  ● Amber dot     = Unsaved changes                              │
│  ⟲ Reset         = Restore to imported SysEx                    │
│  Session restore = Auto-saved, offered on reload               │
└─────────────────────────────────────────────────────────────────┘
```

---

*Built for the Faderfox UC4. Not affiliated with Faderfox.*
