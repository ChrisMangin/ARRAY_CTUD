<div align="center">

# ARRAY_CTUD
**Studio 5000 Add-On Instruction - Rev 1.0**

[![Studio 5000](https://img.shields.io/badge/Studio_5000-v32.04%2B-CC0000?style=flat-square)](https://github.com/ChrisMangin/ARRAY_CTUD)
[![Controller](https://img.shields.io/badge/Controller-ControlLogix%20%7C%20CompactLogix-0078D4?style=flat-square)](https://github.com/ChrisMangin/ARRAY_CTUD)
[![Channels](https://img.shields.io/badge/Channels-1050-22c55e?style=flat-square)](https://github.com/ChrisMangin/ARRAY_CTUD)
[![Memory Savings](https://img.shields.io/badge/Memory-~9x_smaller-f97316?style=flat-square)](https://github.com/ChrisMangin/ARRAY_CTUD)

</div>

---

Array-based multi-channel up/down counter. Up to **1050 independent channels** sharing a single AOI instance, each with its own INT accumulator and edge-triggered count logic.

## Why This Exists

Standard Studio 5000 `COUNTER` tags are expensive:

| Approach | 1,050 channels | Memory usage |
|----------|---------------|--------------|
| Native `COUNTER` tags | 1,050 x 12 bytes | **12,600 bytes** |
| `ARRAY_CTUD` | Single AOI + arrays | **~3,400 bytes** |

**~9x smaller.** One instance of `ARRAY_CTUD` replaces 1,050 native COUNTER tags with no loss of functionality.

---

## Parameters

| Parameter | Type | Direction | Description |
|-----------|------|-----------|-------------|
| `CM_Inc` | DINT | Input | Channel index (0-1049); used when `AUTO_INC = 0` |
| `AUTO_INC` | BOOL | Input | `0` = manual index via `CM_Inc`; `1` = auto-cycle 0 to 1049 |
| `PRE` | INT | Input | Count preset (0-32767) - `C_DN` fires when `ACC = PRE` |
| `CTU` | BOOL | Input | Count-up trigger (rising edge, one count per edge) |
| `CTD` | BOOL | Input | Count-down trigger (rising edge, one count per edge) |
| `DN_0` | BOOL | Input | Also assert `C_DN` when `ACC = 0` (count-down done) |
| `RES` | BOOL | InOut | Reset - AOI auto-clears this bit after reset executes |
| `RESET_TO` | INT | Input | Value to reset accumulator to (default `0`) |
| `C_DN` | BOOL | Output | Done flag |
| `C_CU` | BOOL | Output | Count-up pulse (one scan wide per CTU edge) |
| `C_CD` | BOOL | Output | Count-down pulse (one scan wide per CTD edge) |
| `C_ACC` | INT | Output | Active channel accumulator value |
| `CM_Fault` | BOOL | Output | Out-of-range index; all logic suspended |
| `EFF_INC` | DINT | Local (RO) | Active channel this scan - index your own arrays from other rungs |

---

## Rung Map

| Rung | Function |
|------|----------|
| 0 | Header / overview NOP |
| 1 | Limitations (channel count, ACC range, memory comparison) |
| 2 | Index resolution |
| 3 | Auto-increment |
| 4 | Fault check |
| 5 | Preset sync |
| 6 | Reset |
| 7 | Count up (edge-triggered via ONS) |
| 8 | Count down (edge-triggered via ONS) |
| 9 | Done flag (`C_DN`) |
| 10 | Accumulator output (`C_ACC`) |

---

## AUTO_INC Mode

Set `AUTO_INC = 1` and the AOI steps `INT_INC` through 0 to 1049, one channel per scan. Read `<instance>.EFF_INC` to know which channel is active and index your own trigger arrays:

```
XIC(MY_TRIGGER_ARRAY[MY_CTR.EFF_INC]) -> CTU
```

> At a 10 ms task period, a full sweep of 1,050 channels takes **~10.5 seconds**.

---

## Expanding Channel Count or Accumulator Range

**More channels:** increase `COUNT_ACC`, `CTU_ONS`, `CTD_ONS` array dimensions; update Rungs 3 and 4.

**Larger ACC (DINT):** change `PRE`, `RESET_TO`, `C_ACC`, `COUNT_PRE`, `COUNT_ACC` from `INT` to `DINT`. Memory increases from ~2 bytes/channel to ~4 bytes/channel.

---

## Requirements

- Studio 5000 / RSLogix 5000 **v32.04+**
- Any **ControlLogix** or **CompactLogix** controller

---

## Files

| File | Description |
|------|-------------|
| `ARRAY_CTUD.L5X` | AOI export - import directly into Studio 5000 |
| `gen_aois_v2.py` | Python generator script (regenerates both this AOI and ARRAY_TIMER) |

---

## Related

- [**ARRAY_TIMER**](https://github.com/ChrisMangin/ARRAY_TIMER) - same pattern applied to timers: 1,050 independent delay channels, ~5x smaller than native TIMER tags
