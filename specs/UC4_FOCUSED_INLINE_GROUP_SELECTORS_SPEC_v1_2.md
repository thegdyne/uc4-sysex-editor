# UC4 Focused View Inline Group Selectors Specification

**Version**: 1.2  
**Status**: Phase 1 Implemented  
**Date**: 2026-01-15

---

## Implementation Notes

Phase 1 implemented with one deviation from spec:

**Scroll Preservation**: Spec called for section anchoring (`scrollIntoView` to keep clicked section at top). Implemented with viewport anchoring instead (`window.scrollY` capture/restore). This provides a less jarring UX - the screen stays in exactly the same position rather than snapping the section header to the top.

---

## Overview

Add inline group selector controls to each section header in Focused View, allowing users to change the active group without scrolling back to the nav-bar.

**Key insight**: Unlike Overview (which shows all 8 groups simultaneously), Focused View shows ONE group at a time. The inline selector changes **which group is currently displayed** and triggers a section re-render.

---

## Constants

```javascript
/**
 * Which group variable is canonical when linkGroups is toggled ON.
 * When linkGroups transitions false -> true, both groups align to this domain's value.
 * Must match Overview spec for consistency.
 */
const LINK_CANONICAL_DOMAIN = 'encoder';  // or 'fader'

/**
 * Offset for scroll anchor restoration to account for sticky header.
 * Set to your nav-bar height if sticky, or 0 if not.
 */
const STICKY_HEADER_OFFSET_PX = 0;  // Adjust if nav is sticky

/**
 * Tracks pending scroll anchor to implement "last click wins" for rapid clicking.
 * Prevents queued requestAnimationFrame callbacks from causing scroll jumps.
 */
let pendingFocusedAnchorKey = null;

/**
 * Maps section keys to their group domain.
 * More robust than deriving from CSS className.
 */
const SECTION_DOMAIN = {
    faders: 'fader',
    fader9: 'fader',
    green: 'fader',
    encoders: 'encoder',
    push: 'encoder',
};
```

---

## State Mutation Contract

**Rule**: Any code path that mutates `encoderGroup`, `faderGroup`, `linkGroups`, or `currentSetup` MUST call one of:
- `syncGroupUI()` -- preferred, updates nav-bar + inline selectors
- `renderCurrentView()` -- fallback, rebuilds entire view

This prevents desync between nav-bar tabs and inline selectors.

---

## Problem Statement

In Focused View, users edit controls within a single group. To switch groups, they must:

1. Scroll back up to the nav-bar (potentially far if editing Push Buttons at the bottom)
2. Click the group tab (or two tabs if encoder/fader groups are unlinked)
3. Scroll back down to find their control

This breaks workflow when comparing or editing the same control across multiple groups.

---

## Solution

Add compact group selector widgets inline with each section header. These selectors:
- Appear next to section titles (e.g., "Faders [GrP1] [1][2][3]...")
- Change the **currently displayed group** when clicked
- Trigger a **full Focused View re-render** (Phase 1). Optional partial re-render in Phase 2.
- Are context-aware: encoder/push sections control `encoderGroup`, fader/green/fader9 sections control `faderGroup`
- Match the visual style of nav-bar group tabs but smaller

---

## UI Design

### Layout

```
+----------------------------------------------------------------------------+
| Faders [GrP1] [1][2][3][4][5][6][7][8]                                     |
+----------------------------------------------------------------------------+
| +-------------+ +-------------+ +-------------+ +-------------+            |
| |  Fader 1    | |  Fader 2    | |  Fader 3    | |  Fader 4    | ...        |
| +-------------+ +-------------+ +-------------+ +-------------+            |
+----------------------------------------------------------------------------+

+----------------------------------------------------------------------------+
| Fader 9 [GrP1] [1][2][3][4][5][6][7][8]                                    |
+----------------------------------------------------------------------------+
| +-------------+                                                            |
| |  Fader 9    |                                                            |
| +-------------+                                                            |
+----------------------------------------------------------------------------+

+----------------------------------------------------------------------------+
| Green Buttons [GrP1] [1][2][3][4][5][6][7][8]                              |
+----------------------------------------------------------------------------+
| +-------------+ +-------------+ +-------------+ +-------------+            |
| |  Green 1    | |  Green 2    | |  Green 3    | |  Green 4    | ...        |
| +-------------+ +-------------+ +-------------+ +-------------+            |
+----------------------------------------------------------------------------+

+----------------------------------------------------------------------------+
| Encoders [EnC2] [1][2][3][4][5][6][7][8]                                   |
+----------------------------------------------------------------------------+
| +-------------+ +-------------+ +-------------+ +-------------+            |
| |  Encoder 1  | |  Encoder 2  | |  Encoder 3  | |  Encoder 4  | ...        |
| |  E2.1       | |  E2.2       | |  E2.3       | |  E2.4       |            |
| +-------------+ +-------------+ +-------------+ +-------------+            |
+----------------------------------------------------------------------------+

+----------------------------------------------------------------------------+
| Push Buttons [EnC2] [1][2][3][4][5][6][7][8]                               |
+----------------------------------------------------------------------------+
| +-------------+ +-------------+ +-------------+ +-------------+            |
| |  Push 1     | |  Push 2     | |  Push 3     | |  Push 4     | ...        |
| +-------------+ +-------------+ +-------------+ +-------------+            |
+----------------------------------------------------------------------------+
```

### Selector Widget Anatomy

```
Encoders [EnC2] [1][2][3][4][5][6][7][8]
--------  -----  -----------------------
   |        |              |
   |        |              +-- Tabs 1-8, click to change group (2 is active/highlighted)
   |        +-- Badge (group name), click to edit
   +-- Section title
```

All elements left-aligned, flowing naturally left-to-right. The group name badge replaces the current `.section-badge` which already shows the group name.

### Size Specifications

| Element | Value |
|---------|-------|
| Tab element | `<button type="button">` (must specify type to prevent form submission) |
| Tab width | 1.5rem (2rem on mobile) |
| Tab height | 1.5rem (2rem on mobile) |
| Tab font-size | 0.65rem |
| Gap between tabs | 1px |
| Badge font-size | 0.7rem |
| Badge padding | 0.25rem 0.5rem |
| Badge background | var(--bg-control) |

---

## Section Keys

Each section MUST have a stable `data-section` attribute for scroll anchoring and partial re-render targeting:

| Section | `data-section` value | Domain |
|---------|---------------------|--------|
| Faders 1-8 | `faders` | fader |
| Fader 9 | `fader9` | fader |
| Green Buttons | `green` | fader |
| Encoders | `encoders` | encoder |
| Push Buttons | `push` | encoder |

---

## Control Type to Group Mapping

| Section | Controls Group Variable |
|---------|------------------------|
| Faders 1-8 | `faderGroup` |
| Fader 9 | `faderGroup` |
| Green Buttons | `faderGroup` |
| Encoders | `encoderGroup` |
| Push Buttons | `encoderGroup` |

This matches the UC4 hardware's physical selector domains:
- **Fader domain**: Shift + Green 1-8 selects group for faders and green buttons
- **Encoder domain**: Shift + Encoder 1-8 selects group for encoders and push buttons

---

## Behavior

### Core Principle: Section Re-render Required

Unlike Overview (which shows all groups), Focused View shows ONE group. Changing the group changes what controls are displayed.

**Clicking an inline selector MUST:**
- Update the appropriate group variable (`encoderGroup` or `faderGroup`)
- If Link Groups enabled, update BOTH group variables
- Update nav-bar active tab states
- Re-render view (selectors update because they're rebuilt)
- Update nav-bar group name displays

**Clicking an inline selector MUST NOT:**
- Change which section is in view (preserve scroll anchor)
- Clear or reset the global undo stack

### Click Behavior

Focused view uses **single-click only**. No timer-gating or double-click handling is needed (unlike Overview, which uses double-click to jump to Focused).

1. User clicks a group tab in the inline selector
2. System captures the clicked section's `data-section` key
3. System updates the appropriate group variable (`encoderGroup` or `faderGroup`)
4. If "Link Groups" enabled, both variables update together
5. System calls `selectEncoderGroup()` or `selectFaderGroup()` (existing functions)
6. View re-renders showing new group's controls
7. System scrolls the same section back into view (anchor restoration)

### Scroll Anchor Preservation

When changing groups, the user wants to continue viewing the **same section**, not the same pixel offset. Since section content height may change between groups, we anchor to the section element rather than scrollY.

**Implementation approach:**
```javascript
function handleFocusedGroupChange(domain, groupIndex, sectionKey) {
    // Early exit if already on this group (no change needed)
    const currentGroup = domain === 'encoder' ? encoderGroup : faderGroup;
    if (groupIndex === currentGroup && (!linkGroups || encoderGroup === faderGroup)) {
        return;
    }
    
    // Guard against null sectionKey (still do group change, skip anchor restore)
    if (!sectionKey) {
        if (domain === 'encoder') selectEncoderGroup(groupIndex);
        else selectFaderGroup(groupIndex);
        return;
    }
    
    // Store pending anchor for "last click wins" protection
    pendingFocusedAnchorKey = sectionKey;
    
    // Use existing group selection functions (they handle linkGroups and trigger full re-render)
    if (domain === 'encoder') {
        selectEncoderGroup(groupIndex);
    } else {
        selectFaderGroup(groupIndex);
    }
    
    // Restore scroll to same section after render
    requestAnimationFrame(() => {
        if (pendingFocusedAnchorKey !== sectionKey) return;  // Stale callback
        
        const section = document.querySelector(`section[data-section="${sectionKey}"]`);
        if (!section) return;  // Don't clear; allow next render/frame to try again
        
        section.scrollIntoView({ block: 'start' });
        if (STICKY_HEADER_OFFSET_PX > 0) {
            window.scrollBy(0, -STICKY_HEADER_OFFSET_PX);
        }
        
        pendingFocusedAnchorKey = null;  // Clear only after successful anchor
    });
}
```

### Link Groups Interaction

**When `linkGroups` is true:**
- Clicking ANY inline selector updates BOTH `encoderGroup` and `faderGroup`
- ALL inline selectors show the same active group
- ALL sections re-render to show the new group
- Matches existing nav-bar linking behavior

**When `linkGroups` is false:**
- Fader-domain selectors only update `faderGroup` and re-render fader sections
- Encoder-domain selectors only update `encoderGroup` and re-render encoder sections
- Selectors may show different active groups

**When `linkGroups` transitions false -> true:**
- Compute canonical group: `canon = (LINK_CANONICAL_DOMAIN === 'encoder') ? encoderGroup : faderGroup`
- Set both: `encoderGroup = canon; faderGroup = canon`
- Call `syncGroupUI(); renderCurrentView();`

This ensures Focused and Overview behave identically when linking is toggled.

### Undo/Redo Contract

Group switching MUST NOT:
- Clear or reset the global undo stack
- Discard any captured "before" state used by pending undo operations

Undo/redo actions apply to whichever controls are currently visible, but the history stack persists across group switches. Users can:
1. Make an edit in group 1
2. Switch to group 2
3. Switch back to group 1
4. Undo still reverts the group 1 edit

---

## Visual States

### Tab States

| State | Background | Border | Text |
|-------|------------|--------|------|
| Default | var(--bg-control) | var(--border) | var(--text-dim) |
| Hover | var(--bg-control) | var(--accent) | var(--text) |
| Active | var(--accent) | var(--accent) | var(--bg-dark) |

### Badge States

| State | Style |
|-------|-------|
| Default | Background var(--bg-control), text color matches section color |
| Hover | Cursor pointer, border-color: var(--accent), filter: brightness(1.1) |
| Click | Opens group name editor for active domain group |

### Container Style

```css
/* Focused View Inline Group Selectors */

.focused-section-header {
    display: flex;
    align-items: center;
    margin-bottom: 0.75rem;
    flex-wrap: wrap;
    gap: 0.5rem;
}

.focused-section-header .section-title {
    flex-shrink: 0;
}

.focused-inline-group-selector {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.focused-inline-group-selector .group-name-badge {
    font-size: 0.7rem;
    padding: 0.25rem 0.5rem;
    background: var(--bg-control);
    border: 1px solid var(--border);
    cursor: pointer;
    transition: border-color 0.1s, filter 0.1s;
}

.focused-inline-group-selector .group-name-badge:hover {
    border-color: var(--accent);
    filter: brightness(1.1);
}

/* Color-code badge by section type */
.focused-inline-group-selector.encoder-domain .group-name-badge {
    color: var(--encoder);
}

.focused-inline-group-selector.fader-domain .group-name-badge {
    color: var(--fader);
}

.focused-inline-group-selector .selector-tabs {
    display: flex;
    gap: 1px;
}

.focused-inline-group-selector .selector-tab {
    font-family: inherit;
    font-size: 0.65rem;
    width: 1.5rem;
    height: 1.5rem;
    border: 1px solid var(--border);
    background: var(--bg-control);
    color: var(--text-dim);
    cursor: pointer;
    transition: border-color 0.1s, color 0.1s, background-color 0.1s;
    padding: 0;
}

.focused-inline-group-selector .selector-tab:hover {
    border-color: var(--accent);
    color: var(--text);
}

.focused-inline-group-selector .selector-tab.active {
    background: var(--accent);
    border-color: var(--accent);
    color: var(--bg-dark);
    font-weight: 600;
}

/* Mobile: larger touch targets */
@media (max-width: 768px) {
    .focused-inline-group-selector .selector-tab {
        width: 2rem;
        height: 2rem;
        font-size: 0.75rem;
    }
}
```

---

## Implementation

### Phase 1: Core Implementation

#### 1.1 Modify createSection() to Include Inline Selector

**Signature change:** Remove `badge` and `groupIdx` parameters. Derive both from domain and current state.

```javascript
/**
 * Creates a section with inline group selector.
 * @param {string} title - Section title (e.g., "Encoders")
 * @param {string} className - CSS class for styling (e.g., "encoders")
 * @param {string} sectionKey - Stable key for data-section attribute
 */
function createSection(title, className, sectionKey) {
    const section = document.createElement('section');
    section.className = 'control-section';
    section.dataset.section = sectionKey;  // For scroll anchoring + partial re-render
    
    // Derive domain from sectionKey mapping (more robust than className)
    const domain = SECTION_DOMAIN[sectionKey];
    if (!domain) {
        console.error(`Unknown sectionKey: ${sectionKey}`);
        return null;  // Caller must handle null
    }
    section.dataset.domain = domain;  // For Phase 2 partial re-render
    
    // Derive active group from current state (not passed as parameter)
    const activeGroup = domain === 'encoder' ? encoderGroup : faderGroup;
    
    const header = document.createElement('div');
    header.className = 'focused-section-header';
    
    const h2 = document.createElement('h2');
    h2.className = 'section-title ' + className;
    h2.textContent = title;
    header.appendChild(h2);
    
    // Inline group selector
    const selectorContainer = document.createElement('div');
    selectorContainer.className = `focused-inline-group-selector ${domain}-domain`;
    selectorContainer.dataset.domain = domain;
    selectorContainer.setAttribute('aria-label', 
        domain === 'encoder' ? 'Encoder group selector' : 'Fader group selector');
    
    // Group name badge (clickable to edit) - derives group at click time, not creation time
    const nameBadge = document.createElement('span');
    nameBadge.className = 'group-name-badge';
    nameBadge.textContent = getGroupName(currentSetup, activeGroup);
    nameBadge.title = 'Click to edit group name';
    nameBadge.onclick = () => {
        // Derive group at click time to avoid stale closure (future-proofs Phase 2)
        const currentActiveGroup = domain === 'encoder' ? encoderGroup : faderGroup;
        showGroupNameEditor(currentSetup, currentActiveGroup);
    };
    selectorContainer.appendChild(nameBadge);
    
    // Group tabs
    const tabs = document.createElement('div');
    tabs.className = 'selector-tabs';
    tabs.setAttribute('role', 'group');
    tabs.setAttribute('aria-label', 'Select group');
    
    for (let i = 0; i < 8; i++) {
        const tab = document.createElement('button');
        tab.type = 'button';
        tab.className = 'selector-tab' + (i === activeGroup ? ' active' : '');
        tab.dataset.group = i;
        tab.textContent = i + 1;
        tab.title = `Group ${i + 1}: ${getGroupName(currentSetup, i)}`;
        tab.setAttribute('aria-pressed', String(i === activeGroup));
        tabs.appendChild(tab);
    }
    
    // Attach click handler to tabs container (event delegation)
    // Note: Focused view uses single-click only. No dblclick/timer gating required.
    tabs.addEventListener('click', (e) => {
        const tab = e.target.closest('.selector-tab');
        if (!tab) return;
        
        const groupIndex = parseInt(tab.dataset.group, 10);
        // Derive sectionKey from DOM (more robust than passing it)
        const clickedSectionKey = tab.closest('section')?.dataset.section;
        if (!clickedSectionKey) return;  // Guard against null poisoning
        handleFocusedGroupChange(domain, groupIndex, clickedSectionKey);
    });
    
    selectorContainer.appendChild(tabs);
    header.appendChild(selectorContainer);
    
    section.appendChild(header);
    return section;
}
```

#### 1.2 Updated renderFocusedView() Call Sites

```javascript
function renderFocusedView() {
    const main = document.getElementById('mainContent');
    main.innerHTML = '';
    
    // Helper to safely append sections (createSection may return null)
    const appendSection = (section, gridCreator) => {
        if (!section) return;
        if (gridCreator) {
            const grid = gridCreator();
            section.appendChild(grid);
        }
        main.appendChild(section);
    };
    
    // Faders section (uses fader group)
    const faderSection = createSection('Faders', 'faders', 'faders');
    appendSection(faderSection, () => {
        const grid = document.createElement('div');
        grid.className = 'control-grid';
        for (let i = 0; i < 8; i++) grid.appendChild(createFaderCard(i));
        return grid;
    });
    
    // Fader 9 section (uses fader group)
    const fader9Section = createSection('Fader 9', 'fader9', 'fader9');
    if (fader9Section) {
        fader9Section.appendChild(createFader9Card());
        main.appendChild(fader9Section);
    }
    
    // Green Buttons section (uses fader group)
    const greenSection = createSection('Green Buttons', 'green', 'green');
    appendSection(greenSection, () => {
        const grid = document.createElement('div');
        grid.className = 'control-grid';
        for (let i = 0; i < 8; i++) grid.appendChild(createGreenCard(i));
        return grid;
    });
    
    // Encoders section (uses encoder group)
    const encSection = createSection('Encoders', 'encoders', 'encoders');
    appendSection(encSection, () => {
        const grid = document.createElement('div');
        grid.className = 'control-grid';
        for (let i = 0; i < 8; i++) grid.appendChild(createEncoderCard(i));
        return grid;
    });
    
    // Push Buttons section (uses encoder group)
    const pushSection = createSection('Push Buttons', 'push', 'push');
    appendSection(pushSection, () => {
        const grid = document.createElement('div');
        grid.className = 'control-grid';
        for (let i = 0; i < 8; i++) grid.appendChild(createPushCard(i));
        return grid;
    });
}
```

#### 1.3 Group Change Handler with Scroll Anchor Preservation

```javascript
/**
 * Handles group changes from inline selectors in Focused View.
 * Preserves scroll anchor (same section stays in view) across full re-render.
 * Uses "last click wins" guard to prevent rapid clicking from causing scroll jumps.
 */
function handleFocusedGroupChange(domain, groupIndex, sectionKey) {
    // Early exit if already on this group (no change needed)
    const currentGroup = domain === 'encoder' ? encoderGroup : faderGroup;
    if (groupIndex === currentGroup && (!linkGroups || encoderGroup === faderGroup)) {
        return;
    }
    
    // Guard against null sectionKey (still do group change, skip anchor restore)
    if (!sectionKey) {
        if (domain === 'encoder') selectEncoderGroup(groupIndex);
        else selectFaderGroup(groupIndex);
        return;
    }
    
    // Store pending anchor for "last click wins" protection
    pendingFocusedAnchorKey = sectionKey;
    
    // Use existing group selection functions (they handle linkGroups and trigger full re-render)
    if (domain === 'encoder') {
        selectEncoderGroup(groupIndex);
    } else {
        selectFaderGroup(groupIndex);
    }
    
    // Restore scroll to same section after render
    // Guard ensures only the latest click's anchor is applied
    requestAnimationFrame(() => {
        if (pendingFocusedAnchorKey !== sectionKey) return;  // Stale callback, skip
        
        const section = document.querySelector(`section[data-section="${sectionKey}"]`);
        if (!section) return;  // Don't clear; allow next render/frame to try again
        
        section.scrollIntoView({ block: 'start' });
        if (STICKY_HEADER_OFFSET_PX > 0) {
            window.scrollBy(0, -STICKY_HEADER_OFFSET_PX);
        }
        
        pendingFocusedAnchorKey = null;  // Clear only after successful anchor
    });
}
```

### Phase 2: Polish (Optional)

#### 2.1 Keyboard Navigation

Add arrow key support when a selector tab is focused:

```javascript
tabs.addEventListener('keydown', (e) => {
    const tab = e.target.closest('.selector-tab');
    if (!tab) return;
    
    let newIndex = parseInt(tab.dataset.group, 10);
    
    if (e.key === 'ArrowRight' || e.key === 'ArrowDown') {
        newIndex = (newIndex + 1) % 8;
        e.preventDefault();
    } else if (e.key === 'ArrowLeft' || e.key === 'ArrowUp') {
        newIndex = (newIndex + 7) % 8;
        e.preventDefault();
    } else {
        return;
    }
    
    // Move focus to new tab
    const newTab = tabs.querySelector(`[data-group="${newIndex}"]`);
    if (newTab) {
        newTab.focus();
        // Derive sectionKey from DOM (same as click handler)
        const sectionKey = tab.closest('section')?.dataset.section;
        if (sectionKey) {
            handleFocusedGroupChange(domain, newIndex, sectionKey);
        }
    }
});
```

#### 2.2 Partial Section Re-render

Instead of re-rendering the entire Focused View, only re-render affected sections. This requires the `data-section` and `data-domain` attributes added in Phase 1.

```javascript
function handleFocusedGroupChange(domain, groupIndex, sectionKey) {
    const currentGroup = domain === 'encoder' ? encoderGroup : faderGroup;
    if (groupIndex === currentGroup && (!linkGroups || encoderGroup === faderGroup)) {
        return;
    }
    
    if (linkGroups) {
        // Both domains change, full re-render
        encoderGroup = groupIndex;
        faderGroup = groupIndex;
        syncGroupUI();
        renderFocusedView();
    } else if (domain === 'encoder') {
        encoderGroup = groupIndex;
        syncGroupUI();
        rerenderSectionsByDomain('encoder');
    } else {
        faderGroup = groupIndex;
        syncGroupUI();
        rerenderSectionsByDomain('fader');
    }
    
    requestAnimationFrame(() => {
        const section = document.querySelector(`section[data-section="${sectionKey}"]`);
        if (section) section.scrollIntoView({ block: 'start' });
    });
}

/**
 * Re-renders only sections belonging to the specified domain.
 * Encoder domain: encoders, push
 * Fader domain: faders, fader9, green
 */
function rerenderSectionsByDomain(domain) {
    document.querySelectorAll(`section[data-domain="${domain}"]`).forEach(section => {
        const sectionKey = section.dataset.section;
        // Rebuild section content based on sectionKey
        // Implementation depends on your section rendering architecture
    });
}
```

#### 2.3 Manual Selector Sync (if partial re-render used)

Only needed if using partial re-render (Phase 2). With full re-render, selectors are rebuilt automatically.

```javascript
/**
 * Updates all focused view inline selectors to reflect current group state.
 * Only needed for Phase 2 partial re-render optimization.
 */
function updateFocusedInlineSelectors() {
    document.querySelectorAll('.focused-inline-group-selector').forEach(container => {
        const domain = container.dataset.domain;
        const activeGroup = domain === 'encoder' ? encoderGroup : faderGroup;
        
        // Update tabs
        container.querySelectorAll('.selector-tab').forEach((tab, i) => {
            const isActive = i === activeGroup;
            tab.classList.toggle('active', isActive);
            tab.setAttribute('aria-pressed', String(isActive));
            tab.title = `Group ${i + 1}: ${getGroupName(currentSetup, i)}`;
        });
        
        // Update badge text
        const badge = container.querySelector('.group-name-badge');
        if (badge) {
            badge.textContent = getGroupName(currentSetup, activeGroup);
        }
    });
}
```

---

## Test Cases

### Basic Functionality

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | Click group 3 in Faders section selector | `faderGroup` becomes 2, Fader cards show group 3 data, nav-bar updates |
| 2 | Click group 5 in Encoders section selector | `encoderGroup` becomes 4, Encoder cards show group 5 data, nav-bar updates |
| 3 | Enable Link Groups, click group 2 in any section | Both groups become 1, ALL sections show group 2, all selectors show 2 active |

### Cross-Section Sync

| # | Action | Expected Result |
|---|--------|-----------------|
| 4 | Click group 4 in Faders selector | Faders, Fader 9, Green sections all show group 4 |
| 5 | Click group 6 in Push selector | Encoders and Push sections both show group 6 |
| 6 | linkGroups OFF: Click Faders selector | Only fader-domain sections change; encoder sections unchanged |

### Nav-Bar Sync

| # | Action | Expected Result |
|---|--------|-----------------|
| 7 | Change group via inline selector | Nav-bar tabs update to match |
| 8 | Change group via nav-bar tabs | Inline selectors update (via re-render) |

### Scroll Anchor Preservation

| # | Action | Expected Result |
|---|--------|-----------------|
| 9 | Scroll to Push Buttons, click group 3 in Push selector | Push section header remains at top of viewport (+/-5px) |
| 10 | Scroll to middle, change group multiple times rapidly | Final view shows same section anchored |
| 11 | Rapid click: Faders g2, then Encoders g5, then Push g3 | Ends anchored to Push section (last click wins) |

### Badge Functionality

| # | Action | Expected Result |
|---|--------|-----------------|
| 12 | In Encoders section with activeGroup=5, click badge | Group name editor opens for group 5 (not a stale index) |
| 13 | Edit group name and save | Badge text updates after re-render |

### Link Groups Behavior

| # | Scenario | Expected Behavior |
|---|----------|-------------------|
| 14 | linkGroups ON, click encoder-domain selector | ALL sections update to show same group |
| 15 | linkGroups OFF, click encoder-domain selector | Only Encoders/Push update; Faders unchanged |
| 16 | encoderGroup=2, faderGroup=6, toggle linkGroups ON | Both become canonical (encoderGroup=2), all selectors sync |

### Undo/Redo Preservation

| # | Action | Expected Result |
|---|--------|-----------------|
| 17 | Make edit in group 1, switch to group 2, switch back | Undo still reverts group 1 edit |
| 18 | Make edit, switch groups, undo | Edit is reverted (undo stack not cleared) |

### Setup Change

| # | Action | Expected Result |
|---|--------|-----------------|
| 19 | Change `currentSetup` while in Focused View | Badge text and tab tooltips reflect new setup's group names |

### Accessibility

| # | Action | Expected Result |
|---|--------|-----------------|
| 20 | Tab to inline selector, use arrow keys | Focus moves between tabs |
| 21 | Press Enter/Space on focused tab | Group changes to that tab |
| 22 | Screen reader reads selector | Announces "Encoder group selector", "Select group", "Group 3 pressed" |

---

## Accessibility

### Required

- Tabs use `<button type="button">` (focusable, prevents form submission)
- Active state visually distinct (high contrast accent color)
- `title` attributes provide group name context
- Container has `aria-label` describing selector domain
- Tabs container has `role="group"` and `aria-label="Select group"`
- Each tab has `aria-pressed="true"` or `aria-pressed="false"` (string values)
- Badge is clickable with clear hover state

### ARIA Attributes

```html
<div class="focused-inline-group-selector encoder-domain" 
     data-domain="encoder" 
     aria-label="Encoder group selector">
    <span class="group-name-badge" 
          title="Click to edit group name">EnC2</span>
    <div class="selector-tabs" role="group" aria-label="Select group">
        <button type="button" class="selector-tab" 
                data-group="0" 
                aria-pressed="false" 
                title="Group 1: GrP1">1</button>
        <button type="button" class="selector-tab active" 
                data-group="1" 
                aria-pressed="true" 
                title="Group 2: EnC2">2</button>
        <!-- ... -->
    </div>
</div>
```

---

## Differences from Overview Spec

| Aspect | Overview | Focused |
|--------|----------|---------|
| What selector controls | "Target group for Focused view" | "Currently displayed group" |
| Re-render on click | NO (Overview shows all groups) | YES (full Focused view re-render) |
| Scroll preservation | scrollY (no re-render) | Section anchor (re-render changes DOM) |
| Timer-gated double-click | Yes (to switch to Focused) | No (already in Focused) |
| Group name badge | Separate from selector | Integrated into selector |
| Section creation | Uses `createOverviewSection()` | Uses `createSection()` with different signature |
| Use case | "Set where I'll go next" | "Change what I'm editing now" |

---

## Success Criteria

### Visual / Layout
- [ ] Inline selectors visible in all section headers in Focused View
- [ ] Group name badge and tabs form cohesive unit
- [ ] Selector styling matches nav-bar tabs but at smaller scale
- [ ] Mobile breakpoint increases touch target to 2rem
- [ ] Badge color-coded by domain (encoder blue, fader orange)

### Functional
- [ ] Clicking a tab updates the correct group variable
- [ ] Affected sections re-render with new group's data
- [ ] Nav-bar tabs stay in sync with inline selectors
- [ ] Link Groups behavior works correctly (including toggle alignment)
- [ ] Badge click opens group name editor for correct (active) group
- [ ] `aria-pressed` updates on active state change (string values)

### Performance / UX
- [ ] Clicked section remains in view after group change (+/-5px)
- [ ] No visible flicker beyond necessary re-render
- [ ] Rapid clicking settles on final group without errors
- [ ] Undo stack preserved across group switches

### Data Attributes
- [ ] Each section has `data-section` attribute
- [ ] Each section has `data-domain` attribute
- [ ] Selector containers have `data-domain` attribute

---

## Future Enhancements

- **Swipe gestures**: On mobile, swipe left/right on section header to change groups
- **Group comparison mode**: Show two groups side-by-side for easy comparison
- **Keyboard shortcuts**: 1-8 keys to change group when section header is focused
- **Group color coding**: Subtle background tint based on which group is active
- **Sticky header offset**: Account for sticky nav when scrolling section into view
