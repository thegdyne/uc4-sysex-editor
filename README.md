# Faderfox UC4 SysEx Editor

A web-based editor for creating and editing Faderfox UC4 MIDI controller configuration files.

**[🚀 Launch Editor](https://thegdyne.github.io/uc4-sysex-editor/)**

![UC4 Editor Screenshot](screenshot.png)

## Features

- **Visual Editor** - Edit all UC4 parameters through an intuitive interface
- **All Control Types** - Encoders, faders, green buttons, and push buttons
- **8 Groups** - Full support for all 8 encoder and fader/button groups
- **Verified Encodings** - Based on systematic hardware testing
- **Presets** - Factory defaults and "Channel Per Group" configurations
- **Export** - Download configurations as .syx files

## Supported Parameters

### Encoders
| Parameter | Options |
|-----------|---------|
| Type | CC Absolute, CC Relative 1/2, CC 14-bit, Program Change, Pitch Bend, Aftertouch |
| Acceleration | None, Low, Medium, Maximum |
| Display | Standard, Bipolar, Off |
| CC Number | 0-127 |
| Channel | 1-16 |
| Value Range | Lower/Upper 0-127 |

### Faders
| Parameter | Options |
|-----------|---------|
| Type | CC Absolute, Program Change, Pitch Bend, Aftertouch |
| Mode | Jump, Snap |
| CC Number | 0-127 |
| Channel | 1-16 |
| Value Range | Lower/Upper 0-127 |

### Buttons (Green & Push)
| Parameter | Options |
|-----------|---------|
| Type | Note, CC, Program Change, Aftertouch, Off |
| Mode | Momentary, Toggle |
| Note/CC Number | 0-127 |
| Channel | 1-16 |
| Value Range | Lower/Upper 0-127 |

## Encoding Reference

These encodings were verified through systematic testing of UC4 hardware:

### Encoder Types (4D XX YY - XX low nibble)
| Value | Type |
|-------|------|
| 0 | CCr1 (Relative Mode 1) |
| 1 | CCr2 (Relative Mode 2) |
| 2 | CCAb (Absolute) ✓ |
| 3 | PrGC (Program Change) ✓ |
| 4 | CCAh (14-bit High Res) ✓ |
| 5 | Pbnd (Pitch Bend) ✓ |
| 6 | AFtt (Aftertouch) ✓ |

### Button Types (4D XX YY - XX low nibble)
| Value | Type |
|-------|------|
| 0 | OFF |
| 1 | Note ✓ |
| 2 | CC ✓ |
| 3 | PrGC ✓ |
| 4 | AFtt ✓ |

### Fader Types
| Value | Type |
|-------|------|
| 0 | CCAb (CC Absolute) ✓ |
| 1 | PrGC (Program Change) |
| 2 | Pbnd (Pitch Bend) ✓ |
| 3 | AFtt (Aftertouch) ✓ |

### Acceleration Modes
| Value | Mode |
|-------|------|
| 0 | Acc0 (None) ✓ |
| 1 | Acc1 (Low) ✓ |
| 2 | Acc2 (Medium) ✓ |
| 3 | Acc3 (Maximum) ✓ |

### Display Scale
| Value | Mode |
|-------|------|
| 0 | OFF ✓ |
| 1 | Std (Standard) ✓ |
| 2 | bPoL (Bipolar) ✓ |

### Button/Encoder Modes
| Value | Mode |
|-------|------|
| 0 | Momentary ✓ |
| 1 | Toggle ✓ |

## Usage

### Online
Visit the [GitHub Pages site](https://thegdyne.github.io/uc4-sysex-editor/) to use the editor directly in your browser.

### Local
1. Clone this repository
2. Open `index.html` in any modern web browser
3. No build step or server required

### Workflow
1. Select your setup number (1-16)
2. Edit parameters for each control type
3. Switch between groups using the group selector buttons
4. Use presets for quick configuration
5. Click "Download .syx" to save your configuration
6. Send the .syx file to your UC4 using SysEx Librarian or similar tool

## Technical Details

See [SPECIFICATION.md](SPECIFICATION.md) for the complete reverse-engineered SysEx format documentation.

### File Structure
```
├── index.html          # Main editor application
├── README.md           # This file
├── SPECIFICATION.md    # SysEx format specification
└── LICENSE             # MIT License
```

### Browser Compatibility
- Chrome/Chromium ✓
- Firefox ✓
- Safari ✓
- Edge ✓

## Contributing

Contributions are welcome! If you discover additional encoding details or find bugs:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

### Areas for Improvement
- [ ] SysEx file loading/parsing
- [ ] Group name editing
- [ ] Full 18-setup support
- [ ] MIDI WebMIDI integration for direct transfer
- [ ] Fader mode (Jump/Snap) encoding verification

## Disclaimer

This is an unofficial, community-developed tool based on reverse-engineered specifications. It is not affiliated with or endorsed by Faderfox.

**Use at your own risk.** Always backup your UC4 configurations before loading new SysEx files.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- Faderfox for creating the excellent UC4 controller
- The MIDI community for SysEx documentation resources
- Everyone who contributed to testing and verification

---

Made with ☕ and reverse engineering.
