# ARRAY_CTUD
**Studio 5000 Add-On Instruction (AOI) — Rev 1.0**

Array-based multi-channel up/down counter.  
Up to **1050 independent channels** sharing a single AOI instance, each with its own INT accumulator and edge-triggered count logic.

## Why this exists
Standard Studio 5000 `COUNTER` tags cost **12 bytes each**.  
1050 COUNTER tags = 12,600 bytes.  
`ARRAY_CTUD` stores all 1050 accumulators in arrays: **~3,400 bytes** — roughly **9× smaller**.

## Key parameters
| Parameter | Type | Direction | Description |
|-----------|------|-----------|-------------|
| `CM_Inc` | DINT | Input | Channel index (0–1049), used when `AUTO_INC = 0` |
| `AUTO_INC` | BOOL | Input | 0 = manual index via `CM_Inc`; 1 = auto-cycle 0→1049 |
| `PRE` | INT | Input | Count preset (0–32767). `C_DN` fires when `ACC = PRE` |
| `CTU` | BOOL | Input | Count-up trigger (rising edge, one count per edge) |
| `CTD` | BOOL | Input | Count-down trigger (rising edge, one count per edge) |
| `DN_0` | BOOL | Input | Also assert `C_DN` when `ACC = 0` (count-down done) |
| `RES` | BOOL | InOut | Reset — AOI auto-clears this bit after reset executes |
| `RESET_TO` | INT | Input | Value to reset accumulator to (default 0) |
| `C_DN` | BOOL | Output | Done flag |
| `C_CU` | BOOL | Output | Count-up pulse (one scan wide per CTU edge) |
| `C_CD` | BOOL | Output | Count-down pulse (one scan wide per CTD edge) |
| `C_ACC` | INT | Output | Active channel accumulator value |
| `CM_Fault` | BOOL | Output | Out-of-range index; all logic suspended |
| `EFF_INC` | DINT | Local (RO) | Active channel this scan — read from other rungs to index your own arrays |

## Rung map
| Rung | Function |
|------|----------|
| 0 | Header / overview NOP |
| 1 | **Limitations** (channel count, ACC range, memory comparison) |
| 2 | Index resolution |
| 3 | Auto-increment |
| 4 | Fault check |
| 5 | Preset sync |
| 6 | Reset |
| 7 | Count up (edge-triggered via ONS) |
| 8 | Count down (edge-triggered via ONS) |
| 9 | Done flag (`C_DN`) |
| 10 | Accumulator output (`C_ACC`) |

## AUTO_INC mode
Set `AUTO_INC = 1` and the AOI steps `INT_INC` through 0→1049 one channel per scan.  
Read `<instance>.EFF_INC` to know which channel is active and index your own trigger arrays:
```
XIC(MY_TRIGGER_ARRAY[MY_CTR.EFF_INC]) -> CTU
```
Full sweep at 10 ms task period ≈ **10.5 seconds**.

## Expanding channel count or accumulator range
**More channels:** increase `COUNT_ACC`, `CTU_ONS`, `CTD_ONS` dimensions; update Rungs 3 and 4.  
**Larger ACC (DINT):** change `PRE`, `RESET_TO`, `C_ACC`, `COUNT_PRE`, `COUNT_ACC` from INT → DINT.  
Memory increases from ~2 bytes/channel to ~4 bytes/channel.

## Requirements
- Studio 5000 / RSLogix 5000 v32.04+
- Controller: any ControlLogix / CompactLogix

## File
| File | Description |
|------|-------------|
| `ARRAY_CTUD.L5X` | AOI export — import directly into Studio 5000 |
| `gen_aois_v2.py` | Python generator script (regenerates both this AOI and ARRAY_TIMER) |
