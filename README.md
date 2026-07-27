# 4-Phase-Buck-Converter
A 480W four-phase application-specific 24V-to-12V synchronous buck converter with 98.8% efficiency
 
## Problem statement
 
A **wide input range** forces every component to be sized for the **worst case**
(ripple ∝ `Vin`, transition loss ∝ `Vin²`), which means oversized parts and more
loss. Fixing the input at **24 V** lets the design be tuned for **efficiency**, not
worst-case margin.
 
---
 
## Target
 
| `Vin` | `Vout` | `Iout` | `fsw` | `D` |
| --- | --- | --- | --- | --- |
| 24 V | 12 V | 40 A (4 × 10 A) | 300 kHz | 0.5 |
 
---
 
## Why 300 kHz
 
Low `fsw` → less switching loss but a bigger inductor. High `fsw` → smaller parts.
There's no middle optimum, just a floor set by magnetics size. **300 kHz** keeps the
inductor a tidy 6.8 µH while holding switching loss low.
 
---
 
## Calculations
 
*Power stage below is designed per phase (10 A) and replicated across all 4 phases.*
 
**Inductor**
```
L = Vout·(Vin − Vout) / (Vin·ΔI_L·fsw) = (12·12)/(24·3·300k) = 6.67 µH → 6.8 µH
```
→ `XGL1010-682` — 6.8 µH · DCR 6.2 mΩ · Isat 22 A · Irms 21.2 A
 
**Input cap**
```
Cin ≥ Iout·D(1−D)/(fsw·ΔVin) = 34.7 µF
```
→ 3× `C3225X7R1H226M250AC` — 22 µF · 50 V · ESR 1 mΩ (0.33 mΩ parallel)
 
**Output cap**
```
Cout ≥ ΔIstep²·L/(2·(Vin−Vout)·ΔVdroop) = 71 µF
```
→ 3× `C5750X7R1E476M230KB` — 47 µF · 25 V · ESR 1.5 mΩ (0.5 mΩ parallel)
 
**MOSFETs**
```
P_gate = 2·Qg·VINTVCC·fsw = 2·49nC·5V·300k = 150 mW
```
→ 2× `BSC010N04LS6` — 40 V · Rds 1 mΩ · Qg 49 nC · Coss 1900 pF
 
**Controller**  
→ 2× `LTC3855` — 4.5–38 V in · 0.6–12.5 V out · DCR or RSENSE · 250–770 kHz ·
each drives 2 phases; chained via `CLKOUT` → `PLLIN` for 4-phase operation
 
---
 
## Efficiency
 
```
P_loss = I²·DCR + P_extvcc + P_sw + P_coss + P_gate + I²·Rds + I_Cin²·ESR + I_Cout²·ESR
```
 
| Loss | Formula | mW |
| --- | --- | --- |
| Inductor DCR | `4·I²·DCR` | 2480 |
| EXTVCC dropper | `2·(Vin−5)·I_bias` | 630 |
| Switching | `4·½·Vin·I·(tr+tf)·fsw` | 1160 |
| Coss | `4·½·Coss·Vin²·fsw` | 660 |
| Gate drive | `4·2·Qg·VINTVCC·fsw` | 600 |
| Conduction | `4·I²·Rds` | 520 |
| Cin ESR | cancels at `D = 0.5` (4-phase) | ≈0 |
| Cout ESR | cancels at `D = 0.5` (4-phase) | ≈0 |
 
```
η = Pout/(Pout + P_loss) = 480/(480 + 6.05) ≈ 98.8 %
```
 
---
 
## Four-phase, zero ripple
 
Four identical phases feed one **12 V / 40 A** rail, each carrying 10 A, interleaved
**90° apart** (0° / 90° / 180° / 270°).
 
At **`D = 0.5`** the four phase currents are spaced exactly one-quarter period apart,
so their ripple sums to **zero** at every instant, each rising phase is cancelled by
a falling one. Net input and output ripple → **0**, which is why the `Cin` / `Cout`
ESR terms drop out of the loss budget above.
 
## Schematic
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/e5d097ea-d9d8-41b6-a8a5-f288badee5a7" />
<img width="500" height="386" alt="image" src="https://github.com/user-attachments/assets/5442ffdd-1da0-4278-83e1-af31fe27ff07" />
