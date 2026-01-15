# UC4 Editor: Quick Copy/Paste How-To

A practical guide to rapidly configuring your UC4 using Quick Copy/Paste.

---

## Getting Started

### Activating Quick Paste Mode

1. Go to **Overview** (click the Overview button in the header)
2. Press **Q** or click the **Copy** button in the toolbar

The toolbar expands to show all Quick Paste options.

### The Toolbar

```
Mode: [Off] [Copy] [Paste]   Scope: [Cell] [Column] [Row]
Source: No source            Ch: [0]   CC: [0]   [Clear Source]
```

- **Mode**: Off (normal), Copy (select source), Paste (apply to targets)
- **Scope**: What to copy - single cell, entire column (group), or entire row
- **Ch/CC**: Offset multipliers (explained below)

---

## Basic Copy/Paste

### Copy and Paste a Single Cell

1. Press **Q** to enter Copy mode
2. Click a cell - it highlights amber and you switch to Paste mode
3. Click another cell of the same type - done!
4. Press **Escape** or **Q** to exit

### Copy and Paste a Column (Group)

1. Press **Q** to enter Copy mode
2. Press **2** or click **Column** to set scope
3. Click any cell in the source group - entire column highlights amber
4. Click any cell in the target group - entire column is pasted
5. Repeat for more groups, or press **Escape** to exit

### Copy and Paste a Row

1. Press **Q** to enter Copy mode
2. Press **3** or click **Row** to set scope
3. Click any cell in the source row - entire row (all 8 groups) highlights amber
4. Click any cell in the target row - entire row is pasted

---

## Using Offset Multipliers

The magic of Quick Paste is **relational offsets**. Instead of setting a fixed offset, you set a multiplier that's applied to the *difference* between source and target positions.

### Formula

```
offset = (target position - source position) × multiplier
```

### Example: Incrementing Channels Across Groups

**Goal**: G1 = Ch1, G2 = Ch2, G3 = Ch3, etc.

1. Configure G1 with Channel 1
2. Press **Q**, set Scope to **Column**
3. Set **Ch** multiplier to **+1**
4. Click G1 (copy)
5. Click G2 → Channel becomes 1 + (2-1)×1 = **Ch 2**
6. Click G3 → Channel becomes 1 + (3-1)×1 = **Ch 3**
7. Click G4 → Channel becomes 1 + (4-1)×1 = **Ch 4**
8. Continue through G8

**One setup, seven clicks.**

### Example: CC Blocks of 8

**Goal**: G1 = CC 0-7, G2 = CC 8-15, G3 = CC 16-23, etc.

1. Configure G1 faders with CC 0, 1, 2, 3, 4, 5, 6, 7
2. Press **Q**, set Scope to **Column**
3. Set **CC** multiplier to **+8**
4. Click G1 (copy)
5. Click G2 → CCs become 0+8, 1+8, 2+8... = **CC 8-15**
6. Click G3 → CCs become 0+16, 1+16, 2+16... = **CC 16-23**

### Example: Exact Copy (No Offset)

**Goal**: Duplicate settings exactly, regardless of position

1. Leave both **Ch** and **CC** at **0**
2. Copy and paste anywhere - values stay identical

### Example: Reverse Direction

**Goal**: Decreasing channels (G8 = Ch1, G7 = Ch2, etc.)

1. Configure G8 with Channel 1
2. Set **Ch** multiplier to **-1**
3. Copy G8, paste to G7 → Ch 1 + (7-8)×(-1) = **Ch 2**
4. Paste to G6 → Ch 1 + (6-8)×(-1) = **Ch 3**

---

## Visual Feedback

| What You See | What It Means |
|--------------|---------------|
| **Amber outline** | Source - what you copied |
| **Teal dashed outline** (on hover) | Target - where you'll paste |
| **Flash animation** | Just pasted successfully |

The highlighting respects your scope:
- Cell scope: single cell highlighted
- Column scope: entire column highlighted
- Row scope: entire row highlighted

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| **Q** | Cycle modes: Off → Copy → Paste → Off |
| **1** | Set scope to Cell |
| **2** | Set scope to Column |
| **3** | Set scope to Row |
| **Escape** | Exit Quick Paste, clear source |
| **Ctrl+Z** | Undo last paste |
| **Ctrl+Y** | Redo |

---

## Control Types

You can only paste to the same control type:

| Type | Can Paste To |
|------|--------------|
| Encoder | Encoder only |
| Push Button | Push Button only |
| Green Button | Green Button only |
| Fader 1-8 | Fader 1-8 only |
| Fader 9 | Fader 9 only |

Trying to paste across types shows an error toast.

---

## Tips & Tricks

### Start Fresh
Click **Clear Source** or press **Escape** to reset and start a new copy operation.

### Check Before You Paste
Hover over target cells to see the teal preview before clicking.

### Undo is Your Friend
Each paste is a separate undo entry. Made a mistake? Ctrl+Z.

### Use Reset for Major Mistakes
The **⟲ Reset** button in the header restores everything to the originally imported SysEx.

### Column Scope for Groups
"Column" means "all controls of this type in this group" - it's vertical in the Overview table.

### Row Scope for Positions
"Row" means "this control position across all 8 groups" - it's horizontal in the Overview table.

### Fader 9 is Special
Fader 9 only has one control per group, so Column scope = Cell scope for Fader 9.

---

## Common Workflows

### Workflow: 8-Channel Mixer

Set up 8 groups where each group controls a different MIDI channel:

1. In Focused View, configure G1 completely (Ch 1, your preferred CCs)
2. Switch to Overview
3. Press **Q**, Scope = **Column**, Ch = **+1**, CC = **0**
4. Click G1, then click G2, G3, G4, G5, G6, G7, G8
5. Done - each group now has incrementing channels

### Workflow: Duplicate a Setup

Copy an entire group's settings exactly:

1. Press **Q**, Scope = **Column**, Ch = **0**, CC = **0**
2. Click source group, click target group
3. Exact copy, no offset

### Workflow: Build Rows First

Configure one control across all groups, then copy down:

1. In Focused View, set Fader 1 in each group (or use Row paste)
2. Switch to Overview
3. Press **Q**, Scope = **Row**, Ch = **0**, CC = **+1**
4. Click Row 1, paste to Row 2, 3, 4, etc.
5. Each row gets incrementing CCs

---

## Troubleshooting

**"Can't paste encoder to fader"**
You're trying to paste between different control types. Copy from the same type.

**Nothing happens when I click**
Check you're in Paste mode (not Copy or Off). Check you have a source (amber highlight visible).

**Wrong offset applied**
Remember: offset = (target - source) × multiplier. If multiplier is 0, there's no offset.

**Values seem clamped**
Channel clamps to 1-16, CC clamps to 0-127. You can't exceed these ranges.

**Source highlight won't go away**
Press Escape, or click Off, or click Clear Source.

---

## Quick Reference

```
Q           → Toggle mode
1 / 2 / 3   → Cell / Column / Row scope
Click       → Copy (in Copy mode) or Paste (in Paste mode)
Escape      → Exit and clear
Ctrl+Z      → Undo

Ch/CC multiplier:
  0  = exact copy
  +1 = offset by position difference
  +2 = offset by 2× position difference
  -1 = reverse direction
```
