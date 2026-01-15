# Faderfox UC4 SysEx Protocol Specification

## Overview

- **Full dump size**: 100,640 bytes (18 setups)
- **v1 targets "Send All Setups" format** (100,640 bytes); single-setup format exists but not implemented
- **Encoding**: All values stored as 3-byte triplets: `4D [high] [low]`
- **Decoded value**: `((high & 0x0F) << 4) | (low & 0x0F)`
- **Section markers**: `49 [secId_hi] [secId_lo] 4A [bank_hi] [bank_lo]`
- **Checksums**: 16-bit sum at end of each bank: `4B [hi1] [hi0] 4C [lo1] [lo0]`

---

## Complete Section Map (per setup)

Each (sectionId, bank) pair appears 18 times in the full dump (once per setup).

| Control | Data | Section | Bank | Values | Encoding |
|---------|------|---------|------|--------|----------|
| **Encoders** | Channel+Type | 0x1C | 0x00 | 64 | (Type<<4)\|Chan |
| | CC Number | 0x1C | 0x40 | 64 | Direct 0-127 |
| | Min | 0x1C | 0x80 | 64 | Direct 0-127 |
| | Max | 0x1C | 0xC0 | 64 | Direct 0-127 |
| | Acc+Display | 0x1D | 0x00 | 64 | (Acc<<4)\|Disp |
| **Push Buttons** | Channel+Type | 0x1D | 0x40 | 64 | TypeNibble\|Chan |
| | CC/Note | 0x1D | 0x80 | 64 | Direct 0-127 |
| | Lower Value | 0x1D | 0xC0 | 64 | Direct 0-127 |
| | Upper Value | 0x1E | 0x00 | 64 | Direct 0-127 |
| | Mode+Display | 0x1E | 0x40 | 64 | (Mode<<4)\|Disp |
| **Green Buttons** | Channel+Type | 0x1E | 0x80 | 64 | TypeNibble\|Chan |
| | CC/Note | 0x1E | 0xC0 | 64 | Direct 0-127 |
| | Lower Value | 0x1F | 0x00 | 64 | Direct 0-127 |
| | Upper Value | 0x1F | 0x40 | 64 | Direct 0-127 |
| | Mode+Display | 0x1F | 0x80 | 64 | (Mode<<4)\|Disp |
| **Faders 1-8** | Channel+Type | 0x1F | 0xC0 | 64 | (Type<<4)\|Chan |
| | CC Number | 0x20 | 0x00 | 64 | Direct 0-127 |
| | Min | 0x20 | 0x40 | 64 | Direct 0-127 |
| | Max | 0x20 | 0x80 | 64 | Direct 0-127 |
| | Mode+Display | 0x20 | 0xC0 | 64 | (Mode<<4)\|Disp |
| **Fader 9** | All params | 0x17 | 0x00 | 40 | 5 values Ã— 8 groups |
| **Group Names** | Characters | 0x14 | 0x80 | 32 | 4 chars Ã— 8 groups |

---

## Value Encoding Details

### Encoder Channel+Type Byte (Section 0x1C Bank 0x00)
```
Value = (Type << 4) | Channel

Type values (0-6):
  0 = CCr1 (relative mode 1 - sends 1/127)
  1 = CCr2 (relative mode 2 - sends 63/65)
  2 = CCAb (absolute 7-bit) â€” FACTORY DEFAULT
  3 = PrGC (program change)
  4 = CCAh (14-bit high-res)
  5 = Pbnd (pitch bend)
  6 = AFtt (aftertouch)

Channel: 0-15 (0 = MIDI Ch 1)

Example: 0x20 = CCAb on Channel 1
Example: 0x25 = CCAb on Channel 6
Example: 0x00 = CCr1 on Channel 1 (RELATIVE MODE!)
```

### Encoder Acc+Display Byte (Section 0x1D Bank 0x00)
```
Value = (Acceleration << 4) | DisplayMode

Acceleration (0-3):
  0 = Acc0 (no acceleration)
  1 = Acc1 (low)
  2 = Acc2 (medium)
  3 = Acc3 (max) â€” FACTORY DEFAULT

DisplayMode (0-2):
  0 = OFF
  1 = Std â€” FACTORY DEFAULT
  2 = bPoL (bipolar)

Example: 0x31 = Acc3 + Std display (factory default)
Example: 0x00 = Acc0 + OFF display
```

### Fader 1-8 Channel+Type Byte (Section 0x1F Bank 0xC0)
```
Value = (Type << 4) | Channel

Type values (0-3):
  0 = CCAb (absolute 7-bit) â€” FACTORY DEFAULT
  1 = PrGC (program change)
  2 = Pbnd (pitch bend)
  3 = AFtt (aftertouch)

Channel: 0-15

Example: 0x00 = CCAb on Channel 1
Example: 0x20 = Pbnd on Channel 1
```

### Fader 1-8 Mode+Display Byte (Section 0x20 Bank 0xC0)
```
Value = (Mode << 4) | DisplayMode

Mode:
  0x00 = Jump â€” FACTORY DEFAULT
  0x10 = Snap (bit 4 set)

DisplayMode (0-2):
  0 = OFF
  1 = Std â€” FACTORY DEFAULT
  2 = bPoL

Example: 0x01 = Jump + Std
Example: 0x11 = Snap + Std
```

### Button Channel+Type Byte (Push: 0x1D/0x40, Green: 0x1E/0x80)
```
Value = TypeNibble | Channel

TypeNibble (pre-shifted, DO NOT shift again):
  0x00 = OFF (unverified)
  0x10 = Note â€” FACTORY DEFAULT
  0x20 = CC
  0x30 = PrGC (unverified)
  0x40 = AFtt (unverified)

Channel: 0-15

Example: 0x10 = Note on Channel 1
Example: 0x20 = CC on Channel 1
Example: 0x27 = CC on Channel 8

CRITICAL: Button types are stored as pre-shifted nibbles.
Do NOT treat as 0,1,2 and shift â€” that will corrupt the data.
```

### Button Mode+Display Byte (Push: 0x1E/0x40, Green: 0x1F/0x80)
```
Value = (Mode << 4) | DisplayMode

Mode:
  0x00 = Momentary â€” FACTORY DEFAULT
  0x10 = Toggle (bit 4 set)

DisplayMode:
  Push buttons: 0=OFF, 1=Std
  Green buttons: 0=OFF, 1=Std, 2=EXt

Example: 0x00 = Momentary
Example: 0x10 = Toggle
Example: 0x11 = Toggle + Std display
```

### Fader 9 Structure (Section 0x17 Bank 0x00)
```
5 values per group (40 total for 8 groups):
  [0] Channel (0-15)
  [1] CC Number (0-127)
  [2] Min value (0-127)
  [3] Max value (0-127)
  [4] Mode: 1=Jump, 17(0x11)=Snap (bit 4 = Snap flag)

Group N data starts at: dataOffset + (N Ã— 5 Ã— 3) bytes
(accounting for triplet encoding: each value = 3 bytes)
```

### Group Names (Section 0x14 Bank 0x80)
```
32 values = 4 characters Ã— 8 groups
Character encoding: 7-segment display codes (mapping TBD)

From dump analysis:
  'G' at position 0 decoded as 16
  Changed character decoded as 25
  
Character map needs further analysis.
```

---

## Checksum Algorithm

```javascript
function recalcBankChecksum(buf, dataStart) {
    let sum = 0;
    let pos = dataStart;
    
    // Sum all decoded triplet values
    while (buf[pos] === 0x4D) {
        sum += ((buf[pos+1] & 0x0F) << 4) | (buf[pos+2] & 0x0F);
        pos += 3;
    }
    
    // 16-bit wrap
    sum = sum & 0xFFFF;
    
    // Write checksum to 4B/4C markers
    if (buf[pos] === 0x4B && buf[pos+3] === 0x4C) {
        buf[pos + 1] = 0x20 | ((sum >> 12) & 0x0F);
        buf[pos + 2] = 0x10 | ((sum >> 8) & 0x0F);
        buf[pos + 4] = 0x20 | ((sum >> 4) & 0x0F);
        buf[pos + 5] = 0x10 | (sum & 0x0F);
    }
}
```

---

## Section Finding Algorithm

```javascript
function findSectionBank(buf, secId, bank) {
    for (let i = 0; i < buf.length - 200; i++) {
        if (buf[i] === 0x49 && (i === 0 || buf[i-1] !== 0x4D)) {
            const sec = ((buf[i+1] & 0x0F) << 4) | (buf[i+2] & 0x0F);
            if (sec === secId && buf[i+3] === 0x4A) {
                const bnk = ((buf[i+4] & 0x0F) << 4) | (buf[i+5] & 0x0F);
                if (bnk === bank) return i + 6; // Return data start position
            }
        }
    }
    return -1;
}
```

---

## Index Building (for all 18 setups)

```javascript
// Build index at import time
function buildIndex(buf) {
    const occ = {};  // occ[secId][bank] = [ {dataOffset, valueCount, checksumOffset}, ... ]
    
    for (let i = 0; i < buf.length - 10; i++) {
        if (buf[i] === 0x49 && (i === 0 || buf[i-1] !== 0x4D) && buf[i+3] === 0x4A) {
            const secId = ((buf[i+1] & 0x0F) << 4) | (buf[i+2] & 0x0F);
            const bank = ((buf[i+4] & 0x0F) << 4) | (buf[i+5] & 0x0F);
            const dataOffset = i + 6;
            
            // Count values
            let pos = dataOffset;
            while (buf[pos] === 0x4D) pos += 3;
            const valueCount = (pos - dataOffset) / 3;
            const checksumOffset = pos;  // where 4B starts
            
            // Store occurrence
            if (!occ[secId]) occ[secId] = {};
            if (!occ[secId][bank]) occ[secId][bank] = [];
            occ[secId][bank].push({ dataOffset, valueCount, checksumOffset });
        }
    }
    
    // Convert to per-setup index
    const index = [];
    for (let setupIdx = 0; setupIdx < 18; setupIdx++) {
        index[setupIdx] = {};
        for (const secId in occ) {
            index[setupIdx][secId] = {};
            for (const bank in occ[secId]) {
                if (occ[secId][bank][setupIdx]) {
                    index[setupIdx][secId][bank] = occ[secId][bank][setupIdx];
                }
            }
        }
    }
    
    return index;
}
```

---

## Value Read/Write

```javascript
function decodeValue(buf, offset, idx) {
    const pos = offset + idx * 3;
    if (buf[pos] !== 0x4D) return null;
    return ((buf[pos+1] & 0x0F) << 4) | (buf[pos+2] & 0x0F);
}

function encodeValue(buf, offset, idx, value) {
    const pos = offset + idx * 3;
    buf[pos] = 0x4D;
    buf[pos + 1] = 0x20 | ((value >> 4) & 0x0F);
    buf[pos + 2] = 0x10 | (value & 0x0F);
}
```

---

## Factory Default Values (Setup 1)

### Encoders
- Channel+Type byte: 0x20 (CCAb + Ch1) for all
- Acc+Display byte: 0x31 (Acc3 + Std) for all
- Min: 0, Max: 127
- CCs by group: G1=8-15, G2=16-23, G3=24-31, G4=32-39, G5=72-79, G6=80-87, G7=88-95, G8=96-103

### Faders 1-8
- Channel+Type byte: 0x00 (CCAb + Ch1) for all
- Mode+Display byte: 0x01 (Jump + Std) for all
- Min: 0, Max: 127
- CCs vary by group (see manual)

### Buttons (Push and Green)
- Channel+Type byte: 0x10 (Note + Ch1) for all
- Mode+Display byte: 0x00 (Momentary) or 0x01 (Momentary + Std)
- Lower: 0, Upper: 127
- Notes: 64-127 across groups

### Fader 9
- All groups: CC 112, Ch1, Min 0, Max 127, Mode 1 (Jump)

### Group Names
- GrP1, GrP2, GrP3, GrP4, GrP5, GrP6, GrP7, GrP8

---

## Remaining Unknowns

- Button type nibbles for OFF (0x00?), PrGC (0x30?), AFtt (0x40?) - unverified
- Group name character encoding map (7-segment codes)

---

## Critical Implementation Notes

1. **Encoder types shift, button types don't** â€” Encoder type is 0-6 shifted left; button type is pre-shifted nibble (0x10, 0x20)
2. **Fader Channel+Type is packed** â€” Section 0x1F Bank 0xC0 contains both, not channel alone
3. **Mode uses bit 4** â€” 0x00 = Momentary/Jump, 0x10 = Toggle/Snap
4. **Each (section, bank) appears 18 times** â€” Build index at import, access by setup index
5. **Recalculate checksums per bank** â€” After any edit, recalc that bank's checksum before export
6. **16-bit checksum wrap** â€” `sum & 0xFFFF` before writing
