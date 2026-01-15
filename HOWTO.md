# UC4 Editor HOWTO

Practical workflows for common UC4 configuration tasks.

---

## Getting Started

### First Time Setup

1. **Open the editor** — Load `index.html` in your browser
2. **Factory defaults load** — Editor is immediately usable
3. **Optional:** Import your existing UC4 dump to edit your actual config

### Import Your UC4 Configuration

```
1. Connect UC4 to computer (USB or MIDI)
2. Use SysEx librarian to dump UC4 → save as .syx
3. Click [Import SysEx] in editor
4. Select your .syx file (must be 100,640 bytes)
```

### Save Your Work

```
[Export SysEx]  → .syx file to send back to UC4
[Export JSON]   → Human-readable backup (recommended!)
```

**Tip:** Always export JSON before major changes. It's your undo safety net.

---

## Common Tasks

### Change a Single CC Number

**Fastest method:**
1. Stay in **Focused View**
2. Find the control card (e.g., "Encoder 3")
3. Change the CC field directly
4. Export SysEx

**Time:** ~10 seconds

### Change MIDI Channel for All Controls in a Group

**Using Quick Paste (fastest):**
1. Switch to **Overview** mode → **Encoders** tab
2. Press **Q** to enable Quick Paste
3. Click **Column** scope
4. Set **Ch** multiplier to **+1**
5. Click any cell in Group 1 (copies column)
6. Click Group 2 column header → pastes with Ch+1
7. Repeat for Groups 3-8

**Time:** ~30 seconds

**Using Context Menu:**
1. Switch to **Overview** mode
2. Click the **Encoders** tab
3. Right-click any cell in the target group column
4. Select **Copy Column**
5. Right-click the same column
6. Select **Paste Special...**
7. Set Channel offset (e.g., +1)
8. Paste to: **Entire column**
9. Click **Paste**

**Time:** ~2 minutes

### Set Up Sequential CCs (1, 2, 3... 8)

**Using Context Menu:**
1. **Overview** → select control type tab
2. Click first cell (e.g., Encoder 1, Group 1)
3. Right-click → **Copy Control**
4. Right-click → **Paste Special...**
5. Check **Auto-increment CC by:** `1`
6. Paste to: **Entire column**
7. Click **Paste**

Result: CC 1, 2, 3, 4, 5, 6, 7, 8

### Clone Group 1 to All Groups with Channel Offset

**Goal:** Group 1 = Ch1, Group 2 = Ch2, etc.

**Using Quick Paste:**
1. Set up Group 1 exactly how you want
2. **Overview** → **Encoders** tab
3. Press **Q** → select **Column** scope
4. Set **Ch** to **+1**, **CC** to **0**
5. Click any cell in Group 1 (copies)
6. Click Groups 2-8 in sequence (each gets Ch offset)
7. Repeat for other control type tabs

**Time:** ~1 minute for complete setup

### Label Your Setups

1. Click **[Manage Setups]** button
2. Select a setup (click the number)
3. Click **[Label]** button
4. Enter a name (e.g., "Synth", "Drums", "FX")
5. Labels appear in the setup dropdown

**Tip:** Labels are stored in the editor only (not in SysEx).

### Copy an Entire Setup

1. Click **[Manage Setups]**
2. Select the source setup
3. Click **[Copy]**
4. Select destination setup(s)
5. Click **Copy**
6. Optionally copy the label too

### Swap Two Setups

1. Click **[Manage Setups]**
2. Select both setups (Ctrl+click)
3. Click **[Swap]**
4. Confirm swap

### Reset a Setup to Factory Defaults

1. Click **[Manage Setups]**
2. Select the setup(s) to reset
3. Click **[Reset]**
4. Confirm reset

**Note:** This restores all 264 controls to factory_default.syx values.

### Export a Single Setup

1. Click **[Manage Setups]**
2. Select exactly one setup
3. Click **[Export]**
4. Enter filename
5. Save JSON file

### Import to a Specific Setup Slot

1. Click **[Manage Setups]**
2. Click **[Import]**
3. Select single-setup JSON file
4. Choose target slot
5. Click **Import**

### Find All Conflicts

1. Switch to **Overview** mode
2. Look at filter chips: `[✓ Concurrent (N)]`
3. If N > 0, you have conflicts to fix
4. Conflicting cells show ⚠️ icon and amber highlight
5. Click conflict to select, double-click to edit in Focused view

### Check Conflicts Before a Gig

```
1. Open editor
2. Import your .syx
3. Overview mode → check Concurrent count
4. If 0: You're good!
5. If >0: Review and fix
6. Export clean .syx
```

---

## Quick Paste Guide

Quick Paste is the fastest way to copy/paste in Overview mode.

### Enable Quick Paste

- Press **Q** key, or
- Click **Mode** buttons in the Quick Paste toolbar

### Workflow

1. **Q** → Enable Quick Paste (starts in Copy mode)
2. **Select scope**: Cell (1), Column (2), or Row (3)
3. **Click source** → Copies and switches to Paste mode
4. **Click targets** → Pastes with offsets
5. **Q** again → Exit Quick Paste

### Channel/CC Multipliers

The **Ch** and **CC** dropdowns apply offsets based on group difference:

| Source | Target | Ch=+1 | CC=0 |
|--------|--------|-------|------|
| G1 | G2 | Ch + 1 | CC + 0 |
| G1 | G3 | Ch + 2 | CC + 0 |
| G1 | G5 | Ch + 4 | CC + 0 |

**Example:** Source is G1 Ch1 CC64. Paste to G3 with Ch=+1, CC=0:
- Result: Ch3 CC64 (group diff = 2, so channel offset = 2×1 = 2)

### Visual Feedback

- **Green highlight**: Source cells
- **Cyan outline on hover**: Paste preview
- **Toast messages**: Confirm each paste

---

## Tooltips

Hover over any parameter label or dropdown option to see explanations.

### Examples

| Hover Target | Tooltip |
|--------------|---------|
| **Chan** label | MIDI channel (1-16). Controls which channel receives encoder messages. |
| **CCr1** option | Relative mode 1. Sends 1 for clockwise, 127 for counter-clockwise. Best for Ableton, Bitwig. |
| **Acc** label | Acceleration sensitivity. Higher values = faster response to quick turns. |

---

## Navigation Tips

### Focused View

- Shows one group at a time (per domain)
- Best for detailed editing
- Section order: Faders → Green → Encoders → Push → Fader 9
- Hover over labels for contextual help

### Overview Mode

- Shows all 8 groups simultaneously
- Best for comparison and bulk operations
- Tabs: **All** | Encoders | Push | Green | Faders
- Quick Paste toolbar appears above grid

### Link Groups 🔗

When checked:
- Encoder group and Fader group selectors move together
- Click Group 3 anywhere → both show Group 3

When unchecked:
- Independent selection (matches UC4 hardware behavior)
- Can view Encoder Group 1 + Fader Group 5 simultaneously

### Keyboard Navigation (Overview)

| Key | Action |
|-----|--------|
| Arrow keys | Move selection |
| Enter | Jump to Focused view |
| Tab | Move right |
| Shift+Tab | Move left |
| Escape | Clear selection |
| Ctrl+C | Copy selected |
| Ctrl+V | Paste to selected |
| Q | Toggle Quick Paste |

---

## Copy/Paste Reference

### Copy Scopes

| Menu Item | What's Copied |
|-----------|---------------|
| Copy Control | Single cell (one control's params) |
| Copy Row | One row across all 8 groups |
| Copy Column | All 8 controls in one group |

### Paste Options

| Option | Effect |
|--------|--------|
| Paste | Simple paste, no transforms |
| Paste Special... | Opens transform dialog |

### Paste Special Transforms

| Transform | Range | Example |
|-----------|-------|---------|
| Channel offset | -15 to +15 | Ch1 + 2 = Ch3 |
| CC/Number offset | -127 to +127 | CC64 + 10 = CC74 |
| Auto-increment | Any value | CC1, CC2, CC3... |
| Wrap mode | Clamp / Wrap | Ch15 + 3 = Ch2 (wrap) or Ch16 (clamp) |

---

## Undo/Redo

### How It Works

- Every edit is tracked
- Rapid edits to same field are coalesced (combined)
- Batch operations (paste to row/column) = single undo step
- Setup Manager operations are fully undoable

### Shortcuts

| Action | Windows/Linux | Mac |
|--------|---------------|-----|
| Undo | Ctrl+Z | Cmd+Z |
| Redo | Ctrl+Shift+Z | Cmd+Shift+Z |
| Redo (alt) | Ctrl+Y | — |

### Limits

- 100 undo steps maximum
- Oldest actions dropped when limit reached

---

## Session Persistence

### Auto-Save

- Edits auto-save to browser storage every 2 seconds
- Survives page refresh and browser restart

### Restore Dialog

On page load, if unsaved session exists:
```
┌─────────────────────────────────────────┐
│  Restore Previous Session?              │
│                                         │
│  Found session from: Jan 15, 8:45 PM    │
│                                         │
│  [Discard]              [Restore]       │
└─────────────────────────────────────────┘
```

### When Session Clears

- After Export SysEx
- After Export JSON
- After Import (new file)
- After clicking Discard

---

## Setup Manager Workflows

### Organize Setups by Project

1. Import your UC4 dump
2. Open Setup Manager
3. Label setups: "Studio", "Live Show", "Recording"
4. Use Copy to duplicate a template setup
5. Export JSON as backup

### Create Template Setup

1. Configure Setup 1 as your ideal starting point
2. Open Setup Manager
3. Select Setup 1
4. Copy to Setups 2-8
5. Modify each copy as needed

### Reset After Experimentation

1. Open Setup Manager
2. Select the setup(s) you messed up
3. Click Reset
4. Factory defaults restored

### Share Setup with Bandmate

1. Open Setup Manager
2. Select the setup to share
3. Click Export
4. Send the JSON file
5. They import using Setup Manager → Import

---

## JSON Workflow

### Why Use JSON?

| Benefit | How |
|---------|-----|
| Version control | `git diff` shows exact changes |
| Backup | Save before experiments |
| Share | Send config to bandmates |
| Review | Human-readable format |

### JSON + Git Workflow

```bash
# Before making changes
cd uc4-configs/
# Export JSON from editor, save as my-setup.json

git add my-setup.json
git commit -m "Baseline config"

# Make changes in editor, export again
git diff my-setup.json  # See exactly what changed
git commit -m "Changed encoders to Ch2"
```

### JSON Structure (Full Export)

```json
{
  "version": 1,
  "exportDate": "2026-01-15T20:00:00Z",
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

### JSON Structure (Single Setup)

```json
{
  "format": "uc4-editor-setup",
  "version": "1.0",
  "exported": "2026-01-15T12:00:00Z",
  "sourceSetup": 0,
  "label": "Synth",
  "groups": [...]
}
```

---

## Troubleshooting

### "Invalid file size" on Import

**Cause:** File isn't a complete UC4 dump

**Fix:** Dump ALL setups from UC4, not just current setup

### Changes Don't Appear on UC4

**Checklist:**
1. Did you **Export SysEx**? (not just JSON)
2. Did you **send** the .syx to UC4 via MIDI?
3. Is UC4 in receive mode?

### Lost My Edits

**Check:**
1. Refresh page — look for restore dialog
2. Check Downloads folder for exported files
3. Check for JSON backups

**Prevention:** Export JSON frequently!

### Conflicts Showing Everywhere

**This is normal for factory defaults.**

Factory config has identical settings across groups (intentionally).

- **Mutually-Exclusive conflicts** = Different groups, same message (usually OK)
- **Concurrent conflicts** = Same group, same message (fix these!)

Use filter chips to show only Concurrent.

### Editor Feels Slow

**Try:**
1. Use individual tabs (Encoders, Faders) instead of All view
2. Reduce browser tabs
3. Hard refresh (Ctrl+Shift+R)

### Quick Paste Not Working

**Check:**
1. Are you in Overview mode? (Quick Paste only works there)
2. Press Q to toggle — check the toolbar state
3. Make sure you've clicked a source cell first

---

## Quick Reference

```
IMPORT/EXPORT
  [Import SysEx]     Load .syx from UC4
  [Export SysEx]     Save .syx for UC4
  [Import JSON]      Load human-readable backup
  [Export JSON]      Save human-readable backup

SETUP MANAGER
  [Manage Setups]    Open Setup Manager
  [Label]            Name a setup
  [Clear]            Zero out a setup
  [Copy]             Duplicate setup data
  [Swap]             Exchange two setups
  [Reset]            Restore factory defaults
  [Export]           Save single setup JSON
  [Import]           Load single setup JSON

NAVIGATION
  Setup [01-18]      Select setup to edit
  [🔗 Link]          Sync group selectors
  Encoder Grp [1-8]  Select encoder/push group
  Fader/Btn [1-8]    Select fader/green group
  [Focused]          Detailed editing view
  [Overview]         Grid comparison view

OVERVIEW TABS
  [All]              All control types stacked
  [Encoders]         8×8 encoder grid
  [Push Buttons]     8×8 push grid
  [Green Buttons]    8×8 green grid
  [Faders]           8×8 fader grid + fader9

QUICK PASTE
  Q                  Toggle Quick Paste
  1 / 2 / 3          Cell / Column / Row scope
  Ch dropdown        Channel offset multiplier
  CC dropdown        CC offset multiplier

CONFLICT FILTERS
  [✓ Concurrent]     Show/hide serious conflicts
  [Mutually-Excl]    Show/hide cross-group conflicts

KEYBOARD
  Ctrl+Z             Undo
  Ctrl+Y             Redo
  Ctrl+C             Copy (Overview)
  Ctrl+V             Paste (Overview)
  Q                  Quick Paste mode
  Arrows             Navigate grid
  Enter              Jump to Focused
  Escape             Clear selection
  Right-click        Context menu
```

---

## Getting Help

- **[Usage Guide](UC4_EDITOR_GUIDE.html)** — Complete documentation
- **[? Guide]** button in editor header — Opens guide in new tab
