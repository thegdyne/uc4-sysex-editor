# UC4 SysEx Editor

A web-based configuration editor for the **Faderfox UC4** MIDI controller. Edit all 18 setups, 8 groups, and every parameter without navigating the hardware menus.

**[→ Open Editor](uc4-editor.html)** · **[→ Usage Guide](UC4_EDITOR_GUIDE.html)**

---

## Features

### Core Editing
- **18 Setups × 8 Groups** — Full access to all 4,752 controls
- **Focused View** — Detailed parameter cards for encoders, buttons, and faders
- **Overview Mode** — See all 64 controls per type in an 8×8 grid
- **All View** — See all control types stacked in one scrollable view

### Workflow
- **Import/Export SysEx** — Load from and save to your UC4
- **Import/Export JSON** — Human-readable backups (great for git)
- **Undo/Redo** — Full edit history with coalescing
- **Session Persistence** — Auto-saves to browser storage

### Power Features
- **Conflict Detection** — Find duplicate MIDI assignments automatically
- **Copy/Paste** — Single controls, rows, or columns
- **Paste Special** — Channel offset, CC offset, auto-increment
- **Keyboard Navigation** — Arrow keys, Enter, Tab in Overview mode
- **Link Groups** — Sync encoder and fader group selectors

---

## Quick Start

```
1. Open uc4-editor.html in your browser
2. Factory defaults load automatically
3. Edit in Focused view or navigate with Overview
4. Export SysEx → send to UC4 via MIDI
```

No installation required. Works entirely in the browser.

---

## Screenshots

### Focused View
Edit individual control parameters with full detail:

```
┌─ FADERS ─────────────────────────────────────── GrP1 ─┐
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

### Overview Mode
See all controls at once, spot conflicts instantly:

```
┌─────────────────────────────────────────────────────────┐
│ [All] [Encoders] [Push] [Green] [Faders]               │
│                                                         │
│ [✓ Concurrent (3)] [Mutually-Exclusive (12)]           │
├─────────────────────────────────────────────────────────┤
│         │ GrP1    │ GrP2    │ GrP3    │ GrP4    │ ...  │
│ Enc 1   │ 1:CC 1  │ 1:CC 1  │ 1:CC 1  │⚠1:CC 1 │      │
│ Enc 2   │ 1:CC 2  │ 1:CC 2  │ 1:CC 2  │ 1:CC 2  │      │
│ ...     │         │         │         │         │      │
└─────────────────────────────────────────────────────────┘
```

---

## File Formats

### SysEx (.syx)
- **Size:** 100,640 bytes (full dump of all 18 setups)
- **Use:** Transfer to/from UC4 hardware via MIDI

### JSON
- **Use:** Human-readable backup, version control, sharing
- **Structure:** All setups, groups, and control parameters

```json
{
  "version": 1,
  "exportDate": "2026-01-11T20:00:00Z",
  "setups": [
    {
      "index": 0,
      "groups": [
        {
          "name": "GrP1",
          "encoders": [...],
          "faders": [...],
          ...
        }
      ]
    }
  ]
}
```

---

## Conflict Detection

The editor automatically detects MIDI assignment conflicts:

| Type | Description | Severity |
|------|-------------|----------|
| **Concurrent** | Same message from controls active simultaneously | ⚠️ Fix these |
| **Mutually-Exclusive** | Same message in different groups | Usually OK |

Filter chips in Overview mode let you show/hide each type.

---

## Keyboard Shortcuts

| Action | Key |
|--------|-----|
| Undo | Ctrl+Z |
| Redo | Ctrl+Shift+Z / Ctrl+Y |
| Copy (Overview) | Ctrl+C |
| Paste (Overview) | Ctrl+V |
| Navigate grid | Arrow keys |
| Jump to Focused | Enter |
| Clear selection | Escape |

---

## Browser Compatibility

Tested in:
- Chrome 90+
- Firefox 90+
- Safari 15+
- Edge 90+

Requires JavaScript enabled. No server needed — runs entirely client-side.

---

## Files

| File | Description |
|------|-------------|
| `uc4-editor.html` | The editor application |
| `UC4_EDITOR_GUIDE.html` | Comprehensive usage guide |
| `UC4_EDITOR_GUIDE.md` | Guide in Markdown format |
| `factory_default.syx` | UC4 factory defaults (auto-loaded) |

---

## UC4 Domain Mapping

Understanding how the UC4 organizes controls:

```
ENCODER DOMAIN (Shift + Encoder 1-8 to switch groups)
├── 8 Encoders
└── 8 Push Buttons (under encoders)

FADER/BUTTON DOMAIN (Shift + Green 1-8 to switch groups)
├── 8 Faders
├── 8 Green Buttons
└── Fader 9 (expression)
```

The editor mirrors this with separate group selectors. Use **🔗 Link** to sync them.

---

## Documentation

- **[Usage Guide](UC4_EDITOR_GUIDE.html)** — Complete walkthrough with examples
- **[UC4 SysEx Protocol](UC4_SYSEX_PROTOCOL_COMPLETE.md)** — Technical protocol details
- **[Editor Specification](UC4_EDITOR_IMPROVEMENTS_SPEC.md)** — Feature specifications

---

## Version History

See [CHANGELOG.md](CHANGELOG.md) for detailed release notes.

### Current Features
- ✅ Full SysEx editing (all 18 setups, 8 groups)
- ✅ JSON import/export
- ✅ Undo/Redo with coalescing
- ✅ Session persistence
- ✅ Overview mode with All view
- ✅ Conflict detection (concurrent + mutually-exclusive)
- ✅ Copy/paste with transforms
- ✅ Keyboard navigation
- ✅ Link groups toggle

---

## License

MIT

---

## Acknowledgments

Built for the Faderfox UC4. Not affiliated with Faderfox.
