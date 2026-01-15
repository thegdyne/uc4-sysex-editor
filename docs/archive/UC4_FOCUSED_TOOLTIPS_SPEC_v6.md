# UC4 Editor: Focused View Tooltips Specification v6 (Final)

## Overview

Add contextual help tooltips to all parameter labels and dropdown options in the Focused View. Users hover over labels or options to understand what each setting does without consulting external documentation.

### Design Goals

1. **Discoverable**: Users naturally hover over unfamiliar terms
2. **Non-intrusive**: Tooltips don't interfere with editing workflow
3. **Comprehensive**: Every cryptic abbreviation gets an explanation
4. **Consistent**: Same styling and behaviour throughout
5. **Accessible**: Works with keyboard navigation and screen readers

---

## Tooltip Content

> **Canonical source**: The tables in this section are the authoritative source for tooltip text. The runtime `TOOLTIPS` object must match exactly. Any discrepancy is a bug.
>
> **Update process**: Any tooltip text change must update the spec tables first. Runtime data is then synced to match (manual copy initially; automated generation acceptable later).

### Key Naming Convention

All tooltip keys follow `<group>.<identifier>` pattern:

- **Label keys**: `<controlType>.<paramName>` â€” e.g., `encoder.chan`, `fader.mode`
- **Option keys**: `<optionGroup>.<optionValue>` â€” e.g., `encoderType.CCr1`, `faderMode.Snap`

**Shared option groups**: `acc.*` and `display.*` are global option groups used by multiple control types (encoders and faders). Do not create control-specific variants like `encoderAcc.*`.

### Encoder Parameters

| Key | Label | Tooltip Text |
|-----|-------|-------------|
| `encoder.chan` | Chan | MIDI channel (1-16). Controls which channel receives encoder messages. |
| `encoder.cc` | CC | Controller number (0-127). The MIDI CC number sent when encoder is turned. For CCAh mode, also occupies CC+32 for the LSB. |
| `encoder.type` | Type | Message type sent when encoder is turned. |
| `encoder.acc` | Acc | Acceleration sensitivity. Higher values = faster response to quick turns. |
| `encoder.disp` | Disp | Display mode. How values appear on the UC4's 7-segment display. |
| `encoder.min` | Min | Minimum output value (0-127). Encoder won't send values below this. |
| `encoder.max` | Max | Maximum output value (0-127). Encoder won't send values above this. |

#### Encoder Type Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `encoderType.CCr1` | CCr1 | Relative mode 1. Sends 1 for clockwise, 127 for counter-clockwise. Best for Ableton Live, Bitwig. Endless rotation. |
| `encoderType.CCr2` | CCr2 | Relative mode 2. Sends 65 for clockwise, 63 for counter-clockwise. Alternative relative format for some DAWs. Endless rotation. |
| `encoderType.CCAb` | CCAb | Absolute 7-bit CC. Standard MIDI CC (0-127). Value jumps to match encoder position. Factory default. |
| `encoderType.PrGC` | PrGC | Program Change. Sends MIDI program change messages (0-127 internally, some instruments display as 1-128). Turn to increment/decrement. |
| `encoderType.CCAh` | CCAh | Absolute 14-bit high-resolution CC. Uses CC N for MSB and CC N+32 for LSB, providing 16384 steps. CC number must be 0-31. Best for sensitive parameters like filter cutoff. |
| `encoderType.Pbnd` | Pbnd | Pitch Bend. Sends 14-bit Pitch Bend message on the selected MIDI channel (-8192 to +8191). No CC number needed. |
| `encoderType.AFtt` | AFtt | Aftertouch (Channel Pressure). Sends channel pressure messages (0-127). Use for expression control. |

#### Acceleration Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `acc.0` | Acc0 | No acceleration. 1:1 response regardless of turn speed. Most precise, slowest to traverse full range. |
| `acc.1` | Acc1 | Low acceleration. Slight speed boost for faster turns. Good balance of precision and speed. |
| `acc.2` | Acc2 | Medium acceleration. Moderate speed boost. Faster traversal with some precision trade-off. |
| `acc.3` | Acc3 | Maximum acceleration. Fastest response to quick turns. Best for quickly sweeping through values. Factory default. |

#### Encoder Display Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `display.OFF` | OFF | Display disabled. 7-segment shows nothing when this control is active. |
| `display.Std` | Std | Standard display. Shows current value 0-127 on the 7-segment. Factory default. |
| `display.bPoL` | bPoL | Bipolar display. Shows value as Â±63, centered at 64. Good for pan, detune, or other bipolar parameters. |

---

### Fader Parameters (Faders 1-8)

| Key | Label | Tooltip Text |
|-----|-------|-------------|
| `fader.chan` | Chan | MIDI channel (1-16). Controls which channel receives fader messages. |
| `fader.cc` | CC | Controller number (0-127). The MIDI CC number sent when fader is moved. |
| `fader.type` | Type | Message type sent when fader is moved. |
| `fader.mode` | Mode | Pickup behaviour. How the fader responds when its position doesn't match the current value. |
| `fader.disp` | Disp | Display mode. How values appear on the UC4's 7-segment display. |
| `fader.min` | Min | Value sent at bottom position (0-127). Fader travel maps to Minâ†’Max range. |
| `fader.max` | Max | Value sent at top position (0-127). Set Min > Max to invert fader direction. |

#### Fader Type Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `faderType.CCAb` | CCAb | Absolute 7-bit CC. Standard MIDI CC (0-127). Factory default. Most common choice. |
| `faderType.PrGC` | PrGC | Program Change. Sends MIDI program change messages (0-127). Fader position selects program number. |
| `faderType.Pbnd` | Pbnd | Pitch Bend. Sends 14-bit Pitch Bend message on the selected MIDI channel. Full range mapped to fader travel. No CC number needed. |
| `faderType.AFtt` | AFtt | Aftertouch (Channel Pressure). Sends channel pressure messages. Fader controls expression/pressure amount. |

#### Fader Mode Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `faderMode.Jump` | Jump | Immediate response. Fader sends its current position immediately when moved. May cause value jumps if position doesn't match current parameter value. |
| `faderMode.Snap` | Snap | Pickup/catch mode. Fader only sends after physical position "catches" the current value. Prevents jumps when switching groups or presets. |

> **Hardware note (non-normative)**: On UC4 hardware, holding Shift while moving a fader temporarily uses Jump mode regardless of this setting. This is device behaviour per the UC4 Manual V03, not an editor feature.

#### Fader Display Options

Same as Encoder Display Options (OFF, Std, bPoL) â€” use `display.*` keys.

---

### Fader 9 Parameters

Fader 9 (expression pedal input) has a simplified parameter set â€” always sends CC, no type or display selection.

| Key | Label | Tooltip Text |
|-----|-------|-------------|
| `fader9.chan` | Chan | MIDI channel (1-16). Controls which channel receives expression pedal messages. |
| `fader9.cc` | CC | Controller number (0-127). Typically CC 11 (Expression) or CC 7 (Volume). |
| `fader9.mode` | Mode | Pickup behaviour. Jump sends immediately; Snap waits to catch current value. |
| `fader9.min` | Min | Value sent at heel position (0-127). |
| `fader9.max` | Max | Value sent at toe position (0-127). Set Min > Max to invert pedal direction. |

---

### Push Button Parameters

| Key | Label | Tooltip Text |
|-----|-------|-------------|
| `push.chan` | Chan | MIDI channel (1-16). Controls which channel receives button messages. |
| `push.note` | Note | Note or CC number (0-127). The MIDI note/CC sent when button is pressed. |
| `push.type` | Type | Message type sent when button is pressed/released. |
| `push.mode` | Mode | Button behaviour. Momentary sends on press/release; Toggle latches on/off. |
| `push.lo` | Lo | Lower/release value (0-127). Sent on button release (Momentary) or when toggled off (Toggle). |
| `push.hi` | Hi | Upper/press value (0-127). Sent on button press (Momentary) or when toggled on (Toggle). |

#### Button Type Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `buttonType.OFF` | OFF | Button disabled. No MIDI message sent when pressed. |
| `buttonType.Note` | Note | Note On/Off. Sends Note On (Hi velocity) on press, Note Off (Lo velocity) on release. Factory default. Best for triggering clips/samples in DAWs. |
| `buttonType.CC` | CC | Control Change. Sends CC with Hi value on press, Lo value on release. Good for momentary effects, hold-to-activate controls. |
| `buttonType.PrGC` | PrGC | Program Change. Sends program change (0-127) on press. Lo value typically ignored. Use to switch presets/patches. |
| `buttonType.AFtt` | AFtt | Aftertouch (Channel Pressure). Sends channel pressure Hi on press, Lo on release. |

#### Button Mode Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `buttonMode.Momentary` | Momentary | Gate mode. Sends Hi value on press, Lo value on release. Button active only while held. |
| `buttonMode.Toggle` | Toggle | Latch mode. First press sends Hi value (on), next press sends Lo value (off). Alternates each press. |

---

### Green Button Parameters

| Key | Label | Tooltip Text |
|-----|-------|-------------|
| `green.chan` | Chan | MIDI channel (1-16). Controls which channel receives button messages. |
| `green.note` | Note | Note or CC number (0-127). The MIDI note/CC sent when button is pressed. Also used for LED feedback. |
| `green.type` | Type | Message type sent when button is pressed/released. |
| `green.mode` | Mode | Button behaviour. Momentary sends on press/release; Toggle latches on/off. |
| `green.led` | LED | LED illumination mode. Controls when the green button lights up. |
| `green.lo` | Lo | Lower/release value (0-127). Sent on button release (Momentary) or when toggled off (Toggle). |
| `green.hi` | Hi | Upper/press value (0-127). Sent on button press (Momentary) or when toggled on (Toggle). |

#### Green Button Type Options

Same as Push Button types (OFF, Note, CC, PrGC, AFtt) â€” use `buttonType.*` keys.

#### Green Button Mode Options

Same as Push Button modes (Momentary, Toggle) â€” use `buttonMode.*` keys.

#### Green Button LED Options

| Key | Option | Tooltip Text |
|-----|--------|-------------|
| `greenLed.OFF` | OFF | LED always off. Button never illuminates regardless of state. |
| `greenLed.Std` | Std | Standard mode. LED reflects button state (on when Hi sent) AND responds to incoming MIDI feedback on same Note/CC number. Factory default. |
| `greenLed.EXt` | EXt | External only. LED controlled exclusively by incoming MIDI messages on the same Note/CC number. Button presses don't change LED state. Best for DAW sync where software controls indicator state. |

---

## Label Count Summary

| Control Type | Parameters | Count |
|--------------|------------|-------|
| Encoder | chan, cc, type, acc, disp, min, max | 7 |
| Fader 1-8 | chan, cc, type, mode, disp, min, max | 7 |
| Fader 9 | chan, cc, mode, min, max | 5 |
| Push Button | chan, note, type, mode, lo, hi | 6 |
| Green Button | chan, note, type, mode, led, lo, hi | 7 |
| **Total** | | **32** |

---

## UI Implementation

### Tooltip Mechanism: Rendered Overlay

Tooltips use a **single global overlay element** positioned dynamically, not native `title` attributes. This approach supports:

- Controlled show/hide delays
- Viewport clamping and flip behaviour
- Consistent styling
- Keyboard focus triggers
- Screen reader accessibility

**Why not native `title`**: Inconsistent keyboard and screen reader behaviour across browsers; no reliable delay or positioning control; cannot style.

**Why single overlay vs per-label children**: Simpler DOM, no z-index stacking issues, easier viewport calculations.

### Tooltip Element Structure

```html
<!-- Single tooltip element, appended to body -->
<div id="tooltip" class="tooltip" role="tooltip" aria-hidden="true">
    <div class="tooltip-content"></div>
    <div class="tooltip-arrow"></div>
</div>
```

### Trigger Element Structure

```html
<!-- Labels are hover/tap targets but NOT in keyboard tab order -->
<span class="param-label has-tooltip" 
      data-tooltip-key="encoder.chan">Chan</span>
```

**Label interaction modes**:
- **Mouse**: Hover to show tooltip
- **Touch**: Tap to toggle tooltip
- **Keyboard**: Labels are excluded from tab order; see "Keyboard Accessibility" for alternative

### Tooltip Styling

```css
.tooltip {
    position: fixed;
    min-width: 240px;
    max-width: 320px;
    padding: 0.75rem;
    background: var(--bg-panel);
    border: 1px solid var(--accent);
    color: var(--text);
    font-size: 0.75rem;
    line-height: 1.5;
    z-index: 10000;
    opacity: 0;
    visibility: hidden;
    transition: opacity 0.1s ease-out;  /* 100ms fade */
    pointer-events: none;  /* Clicks pass through on desktop */
}

.tooltip.visible {
    opacity: 1;
    visibility: visible;
}

.tooltip-arrow {
    position: absolute;
    width: 12px;
    height: 12px;
    background: var(--bg-panel);
    border: 1px solid var(--accent);
    transform: rotate(45deg);
    /* Horizontal position set via inline style: left: Npx */
}

/* Vertical arrow positioning based on tooltip placement */
.tooltip.above .tooltip-arrow {
    bottom: -7px;
    border-top: none;
    border-left: none;
}

.tooltip.below .tooltip-arrow {
    top: -7px;
    border-bottom: none;
    border-right: none;
}

/* Dotted underline indicates tooltip available */
.has-tooltip {
    border-bottom: 1px dotted var(--text-dim);
    cursor: help;
}

/* Reduced motion: skip animations */
@media (prefers-reduced-motion: reduce) {
    .tooltip {
        transition: none;
    }
}
```

**Notes**:
- `var(--text-dim)` is `#888898` in the existing theme
- `visibility` is toggled instantly with the `.visible` class; only `opacity` animates
- Users with `prefers-reduced-motion` get instant show/hide (delays still apply, only animation is skipped)
- `pointer-events: none` ensures desktop clicks pass through to underlying UI

### Positioning Algorithm

Tooltip position is calculated deterministically:

1. **Trigger rect**: The **label element** is always the trigger rect for positioning and arrow tracking, whether tooltip is triggered by label hover or control focus

2. **Default placement**: Tooltip appears **above** the trigger element, with tooltip's left edge aligned to trigger's left edge

3. **Right overflow**: If tooltip's right edge would exceed `viewport.right - 16px`, shift tooltip left until `tooltip.right = viewport.right - 16px`

4. **Left overflow**: If tooltip's left edge would be less than `16px`, clamp to `tooltip.left = 16px`

5. **Top overflow / flip**: If tooltip's top edge would be less than `16px` from viewport top, flip tooltip to **below** the trigger element (with 8px gap)

6. **Arrow horizontal positioning**:
   - Calculate trigger element's horizontal center: `triggerCenterX = trigger.left + (trigger.width / 2)`
   - Arrow X position (relative to tooltip) = `triggerCenterX - tooltip.left`
   - Clamp arrow X to range `[16px, tooltipContentWidth - 16px]` where `tooltipContentWidth` is the post-layout content box width (not the CSS `max-width` constant)
   - If trigger is wider than tooltip, arrow centers on tooltip (clamped result)
   - Set via inline style: `arrow.style.left = clampedX + 'px'`

7. **Arrow vertical positioning**: Handled by CSS classes `.above` / `.below`

**Scrolling containers**: Tooltip positioning uses viewport (fixed) coordinates. If Focused View scrolls inside a container, tooltips remain correctly positioned relative to viewport and clamp to viewport edges.

### Timing Behaviour

| Event | Delay | Notes |
|-------|-------|-------|
| Mouse enter label | 400ms | Prevents flicker during normal mouse movement |
| Mouse leave label | 100ms | Grace period before fade begins |
| Mouse re-enter during grace | â€” | Cancels pending hide, tooltip stays visible |
| Opacity fade (show/hide) | 100ms | CSS transition duration (skipped if reduced motion) |
| Control focus (keyboard) | 0ms | Immediate show |
| Control blur (keyboard) | 0ms | Immediate hide |
| Touch tap (show) | 0ms | Immediate on tap |
| Touch tap (dismiss) | 0ms | Tap on label again or tap elsewhere |
| Touch auto-dismiss | 5000ms | Tooltip closes automatically if no interaction |
| Scroll (any device) | 0ms | Any scroll gesture immediately dismisses tooltip |

**Total hide time on mouse leave**: 100ms delay + 100ms fade = **200ms** (or 100ms with reduced motion)

**Fast hover behaviour**: If hover moves from label A to label B before A's 400ms show delay elapses, the pending show for A is cancelled and will not display. Only the most recent hover target is tracked; no queued timers.

**Fade completion detection**: Fade is considered complete on `transitionend` event OR after 120ms timeout (whichever comes first). The timeout fallback handles edge cases where transition events don't fire (tab switch, reduced motion, browser quirks).

### Keyboard Accessibility

**Design decision**: Labels are **not** in keyboard tab order.

Adding `tabindex="0"` to 32 labels would significantly degrade keyboard navigation. Instead:

**Control-focused tooltips**: When a control (`<select>` or `<input>`) receives keyboard focus, the tooltip for its associated label is shown. The **label element in the same `.param-row`** is used as the trigger rect for positioning and arrow tracking.

```html
<div class="param-row">
    <span class="param-label has-tooltip" 
          data-tooltip-key="encoder.type">Type</span>
    <div class="param-input">
        <!-- Control references its label's tooltip key -->
        <select data-tooltip-key="encoder.type" 
                data-option-group="encoderType">...</select>
    </div>
</div>
```

**Behaviour**:
- Tab navigates between controls (selects, inputs) as normal â€” no extra stops
- When a control is focused, tooltip shows for the label in the same row
- Tooltip text comes from the control's `data-tooltip-key` attribute
- Tooltip is positioned relative to the label (not the control)
- When focus leaves the control, tooltip hides

**ARIA handling for control-focused tooltips**:

Minimum required order for screen reader reliability:
1. Tooltip text content must be set **before** `aria-describedby` is applied
2. `aria-hidden="false"` must be set **before** `aria-describedby` is applied

Full show sequence (on control focus):
1. Update tooltip text content from control's `data-tooltip-key`
2. Position tooltip relative to the label element
3. Set `aria-hidden="false"` on tooltip
4. Add `aria-describedby="tooltip"` to the focused **control**
5. Add `.visible` class to tooltip

Hide sequence (on control blur):
1. Remove `.visible` class from tooltip
2. Remove `aria-describedby` from control
3. After fade completes (transitionend or 120ms timeout): set `aria-hidden="true"` on tooltip

### Hover vs Focus Precedence

When both hover and keyboard focus are active (user is hovering a different label while a control is focused):

**Hover takes precedence, reverts on mouse leave**:
- If Type control is focused (showing Type tooltip) and user hovers Chan label, tooltip switches to show Chan
- When mouse leaves Chan label, tooltip reverts to Type (because Type control is still focused)
- If focus then leaves the control, tooltip hides completely

This provides the most natural UX: mouse always shows what you're pointing at, keyboard focus provides a fallback.

### Click/Tap Behaviour by Device

**Desktop (mouse)**:
- Tooltips have `pointer-events: none` â€” clicks pass through to underlying UI
- Clicking anywhere (including the hovered label) dismisses tooltip via implicit mouse-leave
- No special click handling needed

**Touch**:
- Tap on label toggles tooltip visibility
- When tooltip is visible, taps elsewhere are consumed for dismissal and do **not** activate underlying controls (prevents accidental input changes)
- Tapping the currently-showing label again hides the tooltip
- Scroll gesture dismisses tooltip without consuming the scroll

### Touch Support Summary

**Label tooltips on touch devices**:
- First tap on label with `has-tooltip` class â†’ shows tooltip
- Second tap on same label â†’ hides tooltip
- Tap anywhere else â†’ hides tooltip (tap consumed, no click-through)
- Any scroll gesture â†’ immediately hides tooltip (scroll not consumed)
- Auto-dismiss after 5 seconds if no interaction

**Detection**: Touch device detected via `'ontouchstart' in window` or media query `(pointer: coarse)`.

### Overlay Mutual Exclusion

Only one overlay (tooltip or info panel) can be visible at a time:
- Opening an info panel closes any visible tooltip
- Showing a tooltip closes any open info panel
- This prevents stacked overlays and visual clutter

---

## Dropdown Option Help: Info Button Approach

Native `<select><option>` elements cannot display rich tooltips cross-browser. Custom dropdowns would require significant refactoring.

**Solution**: Add an info button next to each dropdown that opens a help panel listing all options with descriptions.

### Info Button Design

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Type [CCAb     â–¾] â“˜                     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Icon**: Use an SVG icon rather than emoji (â„¹ï¸) for consistent cross-platform rendering.

Clicking â“˜ opens a panel:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Encoder Type Options              [Ã—]   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚   CCr1 â€” Relative mode 1. Sends 1 for   â”‚
â”‚          clockwise, 127 for CCW...      â”‚
â”‚                                         â”‚
â”‚   CCr2 â€” Relative mode 2. Sends 65...   â”‚
â”‚                                         â”‚
â”‚ â–º CCAb â€” Absolute 7-bit CC. Standard... â”‚
â”‚                                         â”‚
â”‚   PrGC â€” Program Change. Sends MIDI...  â”‚
â”‚                                         â”‚
â”‚   CCAh â€” Absolute 14-bit high-res...    â”‚
â”‚                                         â”‚
â”‚   Pbnd â€” Pitch Bend. Sends 14-bit...    â”‚
â”‚                                         â”‚
â”‚   AFtt â€” Aftertouch. Sends channel...   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Selection indicator**: Single `â–º` marker shows currently selected option.

### Panel Data Source

**The `<select>` element is the source of truth** for option order and values. The panel:

1. Iterates the actual `<option>` elements in the associated `<select>`
2. For each option, looks up tooltip text via key `<optionGroup>.<optionValue>`
3. Renders in the same order as the select

This prevents drift where the select has options not in `TOOLTIPS` (or vice versa).

**Missing key handling**: If a tooltip key lookup fails, show the option without description and log to console. Dev-only warnings are gated by a build mode or development flag; production builds are silent. Do not show warnings to users.

### Panel Behaviour

**Opening/closing**:
- Opens adjacent to info button, viewport-clamped (same algorithm as tooltips)
- Click Ã— or click outside panel to close
- Opening a new info panel automatically closes any previously open panel (and any visible tooltip)
- Keyboard: Escape closes panel

**Selection**:
- `â–º` marker shows currently selected option (synced with `<select>` value)
- Clicking an option row updates the `<select>` value programmatically (does not open native dropdown UI) but **does not close the panel**
- Selecting the already-selected option is a no-op (no state change)
- If user changes `<select>` value directly while panel is open, `â–º` marker updates to reflect new selection

**Keyboard navigation within panel**:
- Arrow Up/Down moves a **focus ring** (separate from the `â–º` selection marker)
- Focus ring is a visual highlight (e.g., background colour) on the currently focused row
- Enter key selects the focused option (moves `â–º` to it, updates `<select>`)
- Focus ring starts on currently selected option when panel opens

**Rationale for not auto-closing**: Users often want to read multiple option descriptions before deciding. Auto-close on select would force them to reopen the panel repeatedly.

### Dropdown Count

Push buttons and green buttons are **separate control sections** with their own dropdowns, but they share tooltip keys.

**Info buttons required** (12 total):

| Control Section | Dropdowns |
|-----------------|-----------|
| Encoder | Type, Acc, Disp |
| Fader 1-8 | Type, Mode, Disp |
| Fader 9 | Mode |
| Push Button | Type, Mode |
| Green Button | Type, Mode, LED |
| **Total** | **12** |

---

## Tooltip Data Structure

Runtime data structure for tooltip content. **Must match spec tables exactly.**

```javascript
const TOOLTIPS = {
    // Parameter labels - keyed by controlType.paramName
    labels: {
        'encoder.chan': "MIDI channel (1-16). Controls which channel receives encoder messages.",
        'encoder.cc': "Controller number (0-127). The MIDI CC number sent when encoder is turned. For CCAh mode, also occupies CC+32 for the LSB.",
        'encoder.type': "Message type sent when encoder is turned.",
        'encoder.acc': "Acceleration sensitivity. Higher values = faster response to quick turns.",
        'encoder.disp': "Display mode. How values appear on the UC4's 7-segment display.",
        'encoder.min': "Minimum output value (0-127). Encoder won't send values below this.",
        'encoder.max': "Maximum output value (0-127). Encoder won't send values above this.",
        
        'fader.chan': "MIDI channel (1-16). Controls which channel receives fader messages.",
        'fader.cc': "Controller number (0-127). The MIDI CC number sent when fader is moved.",
        'fader.type': "Message type sent when fader is moved.",
        'fader.mode': "Pickup behaviour. How the fader responds when position doesn't match current value.",
        'fader.disp': "Display mode. How values appear on the UC4's 7-segment display.",
        'fader.min': "Value sent at bottom position (0-127). Fader travel maps to Minâ†’Max range.",
        'fader.max': "Value sent at top position (0-127). Set Min > Max to invert fader direction.",
        
        'fader9.chan': "MIDI channel (1-16). Controls which channel receives expression pedal messages.",
        'fader9.cc': "Controller number (0-127). Typically CC 11 (Expression) or CC 7 (Volume).",
        'fader9.mode': "Pickup behaviour. Jump sends immediately; Snap waits to catch current value.",
        'fader9.min': "Value sent at heel position (0-127).",
        'fader9.max': "Value sent at toe position (0-127). Set Min > Max to invert pedal direction.",
        
        'push.chan': "MIDI channel (1-16). Controls which channel receives button messages.",
        'push.note': "Note or CC number (0-127). The MIDI note/CC sent when button is pressed.",
        'push.type': "Message type sent when button is pressed/released.",
        'push.mode': "Button behaviour. Momentary sends on press/release; Toggle latches on/off.",
        'push.lo': "Lower/release value (0-127). Sent on release (Momentary) or when toggled off.",
        'push.hi': "Upper/press value (0-127). Sent on press (Momentary) or when toggled on.",
        
        'green.chan': "MIDI channel (1-16). Controls which channel receives button messages.",
        'green.note': "Note or CC number (0-127). Also used for LED feedback on same number.",
        'green.type': "Message type sent when button is pressed/released.",
        'green.mode': "Button behaviour. Momentary sends on press/release; Toggle latches on/off.",
        'green.led': "LED illumination mode. Controls when the green button lights up.",
        'green.lo': "Lower/release value (0-127). Sent on release (Momentary) or when toggled off.",
        'green.hi': "Upper/press value (0-127). Sent on press (Momentary) or when toggled on."
    },
    
    // Dropdown options - keyed by optionGroup.optionValue
    // Note: acc.* and display.* are shared global groups used by multiple controls
    options: {
        // Encoder types
        'encoderType.CCr1': "Relative mode 1. Sends 1 for clockwise, 127 for counter-clockwise. Best for Ableton Live, Bitwig. Endless rotation.",
        'encoderType.CCr2': "Relative mode 2. Sends 65 for clockwise, 63 for counter-clockwise. Alternative relative format. Endless rotation.",
        'encoderType.CCAb': "Absolute 7-bit CC. Standard MIDI CC (0-127). Value jumps to match encoder position. Factory default.",
        'encoderType.PrGC': "Program Change. Sends MIDI program change (0-127 internally, some instruments display 1-128). Turn to increment/decrement.",
        'encoderType.CCAh': "Absolute 14-bit high-resolution CC. Uses CC N (MSB) and CC N+32 (LSB) for 16384 steps. CC must be 0-31. Best for sensitive parameters.",
        'encoderType.Pbnd': "Pitch Bend. Sends 14-bit Pitch Bend message on the selected MIDI channel (-8192 to +8191). No CC number needed.",
        'encoderType.AFtt': "Aftertouch (Channel Pressure). Sends channel pressure (0-127). Use for expression control.",
        
        // Fader types
        'faderType.CCAb': "Absolute 7-bit CC. Standard MIDI CC (0-127). Factory default. Most common choice.",
        'faderType.PrGC': "Program Change. Sends MIDI program change (0-127). Fader position selects program.",
        'faderType.Pbnd': "Pitch Bend. Sends 14-bit Pitch Bend message on the selected MIDI channel. Full range mapped to fader travel. No CC number needed.",
        'faderType.AFtt': "Aftertouch (Channel Pressure). Sends channel pressure. Fader controls expression amount.",
        
        // Acceleration (shared global group)
        'acc.0': "No acceleration. 1:1 response regardless of turn speed. Most precise, slowest to traverse full range.",
        'acc.1': "Low acceleration. Slight speed boost for faster turns. Good balance of precision and speed.",
        'acc.2': "Medium acceleration. Moderate speed boost. Faster traversal with some precision trade-off.",
        'acc.3': "Maximum acceleration. Fastest response to quick turns. Best for quickly sweeping values. Factory default.",
        
        // Display modes (shared global group)
        'display.OFF': "Display disabled. 7-segment shows nothing when this control is active.",
        'display.Std': "Standard display. Shows current value 0-127 on the 7-segment. Factory default.",
        'display.bPoL': "Bipolar display. Shows value as Â±63, centered at 64. Good for pan or bipolar parameters.",
        
        // Fader modes (shared by faders 1-8 and fader 9)
        'faderMode.Jump': "Immediate response. Sends current position when moved. May cause value jumps.",
        'faderMode.Snap': "Pickup mode. Only sends after position catches current value. Prevents jumps.",
        
        // Button types (shared by push and green buttons)
        'buttonType.OFF': "Button disabled. No MIDI message sent when pressed.",
        'buttonType.Note': "Note On/Off. Sends Note On (Hi velocity) on press, Note Off (Lo) on release. Factory default. Best for triggering clips/samples in DAWs.",
        'buttonType.CC': "Control Change. Sends CC with Hi value on press, Lo on release. Good for momentary effects.",
        'buttonType.PrGC': "Program Change. Sends program change (0-127) on press. Lo value typically ignored.",
        'buttonType.AFtt': "Aftertouch. Sends channel pressure Hi on press, Lo on release.",
        
        // Button modes (shared by push and green buttons)
        'buttonMode.Momentary': "Gate mode. Hi on press, Lo on release. Active only while held.",
        'buttonMode.Toggle': "Latch mode. First press sends Hi, next press sends Lo. Alternates each press.",
        
        // Green LED modes
        'greenLed.OFF': "LED always off. Button never illuminates.",
        'greenLed.Std': "Standard. LED reflects button state AND responds to incoming MIDI feedback. Factory default.",
        'greenLed.EXt': "External only. LED controlled only by incoming MIDI. Button presses don't change LED. Best for DAW sync."
    }
};
```

### Key-to-UI Mapping

When rendering a parameter row, use data attributes to link to tooltip keys:

```html
<div class="param-row">
    <span class="param-label has-tooltip" 
          data-tooltip-key="encoder.type">Type</span>
    <div class="param-input">
        <!-- Control has same tooltip key for keyboard-focus tooltips -->
        <select data-tooltip-key="encoder.type"
                data-option-group="encoderType">...</select>
        <button class="info-btn" 
                data-info-group="encoderType" 
                aria-label="Type options help">
            <svg><!-- info icon --></svg>
        </button>
    </div>
</div>
```

This decouples tooltip content from label display text â€” renaming "Type" to "Msg Type" wouldn't break tooltips.

---

## Implementation Phases

### Phase 1: Label Tooltips

**Scope**: Add tooltips to all `.param-label` elements in Focused View (32 labels total).

**Tasks**:
1. Create tooltip overlay element and append to body
2. Add `has-tooltip` class and `data-tooltip-key` to all param labels
3. Add `data-tooltip-key` to all associated controls (for keyboard focus)
4. Implement mouse hover show/hide with 400ms/100ms delays
5. Implement fast-hover cancellation (only most recent hover target shows)
6. Implement re-enter during grace period (cancels pending hide)
7. Implement control-focused tooltips (keyboard): show tooltip on control focus, position relative to label
8. Implement hover-vs-focus precedence (hover wins, reverts on mouse leave)
9. Implement positioning algorithm with viewport clamping and flip
10. Implement arrow horizontal positioning with post-layout width clamping
11. Add touch support (tap toggle, tap-outside dismiss with click-through prevention, scroll dismiss, 5s auto-dismiss)
12. Add desktop scroll dismiss (mouse wheel)
13. Manage `aria-hidden` and `aria-describedby` dynamically per show/hide sequences (correct ordering)
14. Implement fade completion detection (transitionend + 120ms fallback)
15. Add `prefers-reduced-motion` support

**Validation**:
- [ ] All 7 encoder params have working tooltips
- [ ] All 7 fader 1-8 params have working tooltips
- [ ] All 5 fader 9 params have working tooltips
- [ ] All 6 push button params have working tooltips
- [ ] All 7 green button params have working tooltips
- [ ] Tab navigation between controls shows tooltips (labels not in tab order)
- [ ] Hovering different label while control focused switches tooltip, reverts on mouse leave
- [ ] Touch tap toggles tooltips; tap-outside dismisses with click-through prevention; scroll dismisses
- [ ] Desktop scroll (mouse wheel) dismisses tooltips
- [ ] Fast hover only shows tooltip for final target (pending shows cancelled)
- [ ] Re-entering label during grace period cancels hide

### Phase 2: Dropdown Info Buttons

**Scope**: Add â“˜ buttons next to dropdowns that open option help panels.

**Tasks**:
1. Add info button element next to each of the 12 `<select>` dropdowns
2. Create option panel component with viewport-clamped positioning
3. Populate panel by iterating `<select>` options, looking up tooltips by key
4. Show `â–º` marker on currently selected option
5. Implement click-to-select without opening native dropdown (programmatic value update)
6. Implement live sync: `â–º` updates if `<select>` changes while panel open
7. Implement single-panel rule: opening new panel closes previous
8. Implement overlay mutual exclusion (panel closes tooltip, tooltip closes panel)
9. Implement dismiss on click-outside, Ã— button, or Escape
10. Implement keyboard navigation: arrows move focus ring, Enter selects
11. Handle missing tooltip keys gracefully (dev-only console.warn, silent in prod)

**Validation**:
- [ ] All 12 dropdowns have working info buttons
- [ ] Panel shows options in same order as select
- [ ] `â–º` marker shows currently selected option
- [ ] Clicking option updates select value programmatically (no native dropdown opened), panel stays open
- [ ] `â–º` updates if select value changes externally while panel open
- [ ] Opening new info panel closes previous panel and any tooltip
- [ ] Showing tooltip closes any open info panel
- [ ] Re-selecting current option is no-op
- [ ] Arrow keys move focus ring independently of `â–º` marker
- [ ] Enter selects focused option
- [ ] Missing tooltip keys log to console in dev only, don't crash

### Phase 3: Contextual Section Tips (Optional)

Deferred. Implement only if Phase 1+2 feel incomplete.

---

## Future Enhancements

These are **not** in Phase 1/2 acceptance criteria:

- **Sticky hover**: When moving between adjacent labels within 100ms, reuse the existing show timer rather than restarting from zero. Prevents sluggish feel when scanning across a row of labels.
- **Tooltip pinning**: Option to "pin" a tooltip open for reference while editing.
- **Localisation**: Structure for multi-language tooltip content.
- **Help mode toggle**: Alternative approach where labels become tabbable only when a "?" toggle is enabled.

---

## Success Criteria (Testable)

### Functional

- [ ] All 32 parameter labels in Focused View show tooltip on hover
- [ ] Tooltip text matches spec tables exactly (no placeholders)
- [ ] All 12 dropdowns have info buttons that open option help panels
- [ ] `â–º` marker in panel shows currently selected option
- [ ] Selecting option from panel updates the actual control value (programmatically, no native dropdown)
- [ ] Panel does not auto-close on selection
- [ ] Opening new panel closes previous panel
- [ ] Tooltip and info panel are mutually exclusive (only one visible at a time)

### Timing

- [ ] Tooltip appears after 400ms hover (Â±50ms tolerance)
- [ ] Tooltip begins hiding after 100ms of mouse leave (Â±20ms tolerance)
- [ ] Mouse re-enter during grace period cancels hide
- [ ] Opacity fade completes in 100ms (Â±20ms tolerance) or instant with reduced motion
- [ ] Total hide time â‰ˆ200ms (100ms delay + 100ms fade)
- [ ] Keyboard focus shows tooltip within 50ms
- [ ] Touch auto-dismiss occurs at 5000ms (Â±200ms)
- [ ] Fade completion detected via transitionend or 120ms timeout fallback
- [ ] Fast hover: pending show cancelled if hover changes before 400ms elapses

### Positioning

- [ ] Tooltip never extends beyond viewport at 1024Ã—768 resolution
- [ ] Tooltip never extends beyond viewport at 1920Ã—1080 resolution
- [ ] Tooltip works correctly at browser zoom 80â€“200%
- [ ] Tooltip flips below trigger when insufficient space above (<16px from top)
- [ ] Arrow horizontal position tracks trigger (label) center
- [ ] Arrow is clamped to stay within tooltip content box (16px from edges, using post-layout width)
- [ ] Scrolling container doesn't break tooltip positioning (fixed coords)
- [ ] Scroll while tooltip visible dismisses immediately (no stale positioning)

### Accessibility

- [ ] Labels are **not** in tab order (no `tabindex` on labels)
- [ ] Focusing a control shows tooltip for its associated label
- [ ] Tooltip positioned relative to label element (not control)
- [ ] Focused control has `aria-describedby="tooltip"` when tooltip visible
- [ ] Tooltip has `aria-hidden="false"` when visible, `"true"` when hidden
- [ ] ARIA attributes set in correct order (content set, then aria-hidden=false, then aria-describedby)
- [ ] Screen reader announces tooltip content on control focus
- [ ] Reduced motion preference skips opacity animation (delays preserved)
- [ ] Touch: tap shows tooltip, tap-elsewhere dismisses with click-through prevention, scroll dismisses without prevention
- [ ] Desktop: mouse clicks pass through tooltip (`pointer-events: none`)
- [ ] Info panels are keyboard-navigable (Escape closes, arrows move focus ring, Enter selects)

### Visual

- [ ] Tooltip styling matches existing dark theme (`--bg-panel`, `--accent`, etc.)
- [ ] Dotted underline on labels indicates tooltip availability
- [ ] No visual glitches during show/hide transitions
- [ ] Info button icon renders consistently across browsers (SVG, not emoji)
- [ ] Panel shows single `â–º` marker (no `â—` or other indicators)

---

## References

- UC4_EDITOR_SPEC_FULL.md â€” Complete parameter definitions
- UC4_SYSEX_PROTOCOL_COMPLETE.md â€” Technical encoding details  
- UC4 Manual V03.pdf â€” Official Faderfox documentation (confirms hardware behaviours)
