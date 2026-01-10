# Faderfox UC4 SysEx Editor - Complete Guide

## 🚀 Quick Links

| Resource | Link |
|----------|------|
| **UC4 SysEx Editor** | [thegdyne.github.io/uc4-sysex-editor](https://thegdyne.github.io/uc4-sysex-editor/) |
| **GitHub Repository** | [github.com/thegdyne/uc4-sysex-editor](https://github.com/thegdyne/uc4-sysex-editor) |
| **Faderfox Official Site** | [faderfox.de](https://www.faderfox.de/) |
| **UC4 Product Page** | [faderfox.de/uc4.html](http://www.faderfox.de/uc4.html) |
| **UC4 Manual (PDF)** | [faderfox.de/PDF/UC4-Manual-V03.pdf](http://www.faderfox.de/PDF/UC4-Manual-V03.pdf) |

---

## What Is This?

The **UC4 SysEx Editor** is a free, browser-based tool for creating custom MIDI mappings for the [Faderfox UC4](http://www.faderfox.de/uc4.html) controller. Instead of manually programming each parameter on the device, you can:

1. **Visually edit** all 264 control parameters
2. **Download a .syx file** with your configuration
3. **Send it to your UC4** using a SysEx transfer tool

No installation required - it runs entirely in your web browser.

---

## What You'll Need

### Hardware
- Faderfox UC4 controller
- USB cable (included with UC4)

### Software (choose one SysEx transfer tool)

| Platform | Tool | Link |
|----------|------|------|
| **macOS** | SysEx Librarian (recommended) | [snoize.com/sysexlibrarian](https://www.snoize.com/sysexlibrarian/) |
| **macOS** | MIDI Tools | [App Store](https://apps.apple.com/app/midi-tools/id1615instruments) |
| **Windows** | MIDI-OX | [midiox.com](http://www.midiox.com/) |
| **Windows** | SendSX | [bome.com/products/sendsx](https://www.bome.com/products/sendsx) |
| **Linux** | amidi (ALSA) | Included with ALSA utils |
| **Cross-platform** | Elektron Transfer | [elektron.se/transfer](https://www.elektron.se/support/transfer) |

---

## Step-by-Step Guide

### Step 1: Open the Editor

Go to **[thegdyne.github.io/uc4-sysex-editor](https://thegdyne.github.io/uc4-sysex-editor/)**

You'll see the editor interface with tabs for:
- **Encoders** (1-8)
- **Faders** (1-9)
- **Green Buttons** (1-8)
- **Push Buttons** (1-8 encoder push buttons)

### Step 2: Select Your Setup

The UC4 has **16 user setups** (plus 2 Ableton setups). Use the dropdown in the top-right to select which setup you're editing.

> **Tip:** Start with Setup 1 for testing. You can always factory reset individual setups on the UC4.

### Step 3: Configure Your Controls

#### Encoders
Each encoder can be configured with:

| Parameter | Description |
|-----------|-------------|
| **CC/Note** | MIDI CC number (0-127) |
| **Channel** | MIDI channel (1-16) |
| **Type** | CC Absolute, CC Relative 1/2, 14-bit, Program Change, Pitch Bend, Aftertouch |
| **Acceleration** | How fast values change when turning quickly (None/Low/Medium/Maximum) |
| **Display** | How values appear on UC4's display (Standard 0-127, Bipolar -63 to +63, Off) |
| **Lower/Upper** | Value range limits |

#### Faders
| Parameter | Description |
|-----------|-------------|
| **CC/Note** | MIDI CC number (0-127) |
| **Channel** | MIDI channel (1-16) |
| **Type** | CC Absolute, Program Change, Pitch Bend, Aftertouch |
| **Mode** | Jump (immediate) or Snap (catch current value first) |
| **Lower/Upper** | Value range limits |

#### Buttons (Green & Push)
| Parameter | Description |
|-----------|-------------|
| **Note/CC** | Note or CC number (0-127) |
| **Channel** | MIDI channel (1-16) |
| **Type** | Note, CC, Program Change, Aftertouch, Off |
| **Mode** | Momentary (press/release) or Toggle (on/off) |
| **Lower/Upper** | Off/On values |

### Step 4: Work with Groups

The UC4 has **8 groups** for each control type. Switch between groups using the numbered buttons (1-8) in each section.

> **Pro Tip:** Use the "Copy to All Groups" buttons to quickly copy channel or CC settings across all 8 groups.

### Step 5: Use Presets (Optional)

| Preset | Description |
|--------|-------------|
| **Factory Defaults** | Loads the original Faderfox factory settings |
| **Channel Per Group** | Sets each group to a different MIDI channel (Group 1 = Ch 1, Group 2 = Ch 2, etc.) |

### Step 6: Download Your Configuration

1. Click the green **"Download .syx"** button
2. Save the file (e.g., `UC4_Setup1.syx`)

---

## Transferring to Your UC4

### Using SysEx Librarian (macOS)

1. **Download & Install** [SysEx Librarian](https://www.snoize.com/sysexlibrarian/)
2. **Connect** your UC4 via USB
3. **Open** SysEx Librarian
4. **Set Destination** to "Faderfox UC4" in the dropdown
5. **Add** your .syx file (drag & drop or File → Add)
6. **Prepare UC4:**
   - Hold **Shift + Edit** twice to enter Setup mode
   - Push & hold **Encoder 7** until "rEc" appears (receive mode)
7. **Send** from SysEx Librarian (click Play or press ⌘P)
8. UC4 display will show progress, then the setup number when complete

### Using MIDI-OX (Windows)

1. **Download & Install** [MIDI-OX](http://www.midiox.com/)
2. **Connect** your UC4 via USB
3. **Configure** MIDI-OX:
   - Options → MIDI Devices
   - Select "Faderfox UC4" as Output
4. **Prepare UC4** for receiving (see above)
5. **Send** the file:
   - SysEx → Send SysEx File
   - Select your .syx file
   - Click Send

### Using amidi (Linux)

```bash
# List MIDI devices
amidi -l

# Send SysEx (replace hw:1,0 with your UC4's device)
amidi -p hw:1,0 -s UC4_Setup1.syx
```

---

## Backing Up Your UC4

Before making changes, **always backup your current settings:**

### On the UC4:
1. Hold **Shift + Edit** twice → Setup mode
2. Push & hold **Encoder 8** → "SndA" (Send All)
3. Wait for progress bar to complete

### In SysEx Librarian:
1. Click **Record**
2. Trigger the send from UC4
3. Save the received data as your backup

---

## Troubleshooting

### "SysEx not received" or no response

1. **Check USB connection** - try a different cable or port
2. **Check MIDI routing** on UC4:
   - Setup mode → Encoder 2 → Set to "Rou5" or "Rou6"
3. **Check SysEx Librarian destination** - make sure UC4 is selected
4. **Reduce SysEx speed** in your transfer tool if available

### Values don't match what I set

- The editor generates new configurations - it doesn't modify existing files
- Some parameters may have slight variations due to encoding

### UC4 shows "Err"

- Transfer was interrupted - try again
- Make sure UC4 is in receive mode before sending
- Don't send to both USB and MIDI simultaneously

---

## UC4 Quick Reference

### Enter Edit Mode
Hold **Shift** + Press **Edit**

### Enter Setup Mode  
Hold **Shift** + Press **Edit** twice

### Switch Encoder Group
Hold **Shift** + Push **Encoder 1-8**

### Switch Fader/Button Group
Hold **Shift** + Press **Green Button 1-8**

### Factory Reset Single Setup
Setup mode → Push & hold **Encoder 5** ("rESc")

### Factory Reset All Setups
Setup mode → Push & hold **Encoder 6** ("rESA")

---

## Example Configurations

### 8 Synth Modules (Channel Per Group)
Use the "Channel Per Group" preset to control 8 different synths:
- Group 1 → Synth on Ch 1
- Group 2 → Synth on Ch 2
- etc.

### DAW Mixer (Same Channel, Different CCs)
Keep all controls on Channel 1 with sequential CCs:
- Encoders: CC 1-8 (pan, sends, etc.)
- Faders: CC 11-19 (volume)
- Buttons: Notes 60-67 (mute/solo)

### Live Performance (Grouped by Song)
Use each group as a different song/scene:
- Group 1: Song 1 parameters
- Group 2: Song 2 parameters
- etc.

---

## Links & Resources

### Official Faderfox
- [Faderfox Website](https://www.faderfox.de/)
- [UC4 Product Page](http://www.faderfox.de/uc4.html)
- [UC4 Manual PDF](http://www.faderfox.de/PDF/UC4-Manual-V03.pdf)
- [Faderfox Support](mailto:info@faderfox.de)

### SysEx Tools
- [SysEx Librarian (macOS)](https://www.snoize.com/sysexlibrarian/)
- [MIDI-OX (Windows)](http://www.midiox.com/)
- [SendSX (Windows)](https://www.bome.com/products/sendsx)

### MIDI Resources
- [MIDI CC List](https://www.midi.org/specifications-old/item/table-3-control-change-messages-data-bytes-2)
- [MIDI Note Numbers](https://www.inspiredacoustics.com/en/MIDI_note_numbers_and_center_frequencies)

### This Project
- [UC4 SysEx Editor](https://thegdyne.github.io/uc4-sysex-editor/)
- [GitHub Repository](https://github.com/thegdyne/uc4-sysex-editor)
- [SysEx Format Specification](https://github.com/thegdyne/uc4-sysex-editor/blob/main/SPECIFICATION.md)

---

## Contributing

Found a bug? Want to add a feature? Contributions welcome!

1. Fork the [repository](https://github.com/thegdyne/uc4-sysex-editor)
2. Make your changes
3. Submit a pull request

---

## License

MIT License - Free to use, modify, and distribute.

**Disclaimer:** This is an unofficial community tool. Not affiliated with Faderfox. Use at your own risk. Always backup your UC4 before loading new configurations.
