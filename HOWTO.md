# UC4 Editor HOWTO

Practical workflows for common UC4 configuration tasks.

---

## Getting Started

### First Time Setup

1. **Open the editor** — Load `uc4-editor.html` in your browser
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

**Using Copy/Paste:**
1. Switch to **Overview** mode
2. Click the **Encoders** tab
3. Right-click any cell in the target group column
4. Select **Copy Column**
5. Right-click the same column
6. Select **Paste Special...**
7. Set Channel offset (e.g., +1)
8. Paste to: **Entire column**
9. Click **Paste**
10. Repeat for Push, Green, Faders tabs

**Time:** ~2 minutes

### Set Up Sequential CCs (1, 2, 3... 8)

**Using Auto-Increment:**
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

1. Set up Group 1 exactly how you want it
2. **Overview** → **Encoders** tab
3. Right-click Group 1, Encoder 1 → **Copy Column**
4. For each Group 2-8:
   - Right-click Group N, Row 1
   - **Paste Special...**
   - Channel offset: `+(N-1)` (e.g., +1 for Group 2)
   - Paste to: Entire column
5. Repeat for Push, Green, Faders

**Time:** ~5 minutes for complete setup

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

## Navigation Tips

### Focused View

- Shows one group at a time (per domain)
- Best for detailed editing
- Section order: Faders → Green → Encoders → Push → Fader 9

### Overview Mode

- Shows all 8 groups simultaneously
- Best for comparison and bulk operations
- Tabs: **All** | Encoders | Push | Green | Faders

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
│  Found session from: Jan 11, 8:45 PM    │
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

### JSON Structure

```json
{
  "version": 1,
  "exportDate": "2026-01-11T20:00:00Z",
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

---

## Quick Reference

```
IMPORT/EXPORT
  [Import SysEx]     Load .syx from UC4
  [Export SysEx]     Save .syx for UC4
  [Import JSON]      Load human-readable backup
  [Export JSON]      Save human-readable backup

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

CONFLICT FILTERS
  [✓ Concurrent]     Show/hide serious conflicts
  [Mutually-Excl]    Show/hide cross-group conflicts

KEYBOARD
  Ctrl+Z             Undo
  Ctrl+Y             Redo
  Ctrl+C             Copy (Overview)
  Ctrl+V             Paste (Overview)
  Arrows             Navigate grid
  Enter              Jump to Focused
  Escape             Clear selection
  Right-click        Context menu
```

---

## Getting Help

- **[Usage Guide](UC4_EDITOR_GUIDE.html)** — Complete documentation
- **[? Guide]** button in editor header — Opens guide in new tab
