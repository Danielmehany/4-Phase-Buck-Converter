# 4-Phase-Buck-Converter
A 480W four-phase application-specific 24V-to-12V synchronous buck converter with 99.0% efficiency

<img width="1100" height="390" alt="image" src="https://github.com/user-attachments/assets/342faa0d-e8ee-453b-a59e-6dc318254f54" />
<img width="1100" height="390" alt="image" src="https://github.com/user-attachments/assets/65f562ca-50a0-4648-a980-8922db322188" />


## Problem statement

A wide input range forces every component to be sized for the worst case (ripple ∝ `Vin`, transition loss ∝ `Vin²`), which means oversized parts and more loss. Fixing the input at 24 V lets the design be tuned for efficiency, not worst-case margin.

---

## Why 300 kHz

Lower `fsw` → less switching loss, but a slower loop and a weaker current-sense signal. Higher `fsw` → faster transient response, but more switching and gate drive loss. 300 kHz sits at the balance point.

---

## Target

| `Vin` | `Vout` | `Iout` | `fsw` | `D` |
| --- | --- | --- | --- | --- |
| 24 V | 12 V | 40 A (4 × 10 A) | 300 kHz | 0.5 |

→ 2× `LTC3855` controller chained, 2 phases each, 90° interleave, DCR sensing

---

## Calculations

*Power stage below is designed per phase (10 A) and replicated across all 4 phases.*

**Inductor**

At `D = 0.5` on 4 phases, output ripple cancels regardless of `ΔI_L` — so `L` isn't sized for ripple. It's sized down until the DCR sense signal gets too small (`ΔVSENSE ≈ 10 mV` floor) or peak current approaches `Isat`.

```
ΔI_L = Vout·(Vin − Vout) / (Vin·L·fsw) = (12·12)/(24·3.3µ·300k) = 6.06 A
```
→ `XGL1010-332` — 3.3 µH · DCR 3.2 mΩ · Isat 24 A · Irms 26 A

✓ `ΔVSENSE` = 19.4 mV (> 10 mV floor) · `I_peak` = 13.0 A (< 24 A Isat)

**Output cap**

Not filtering ripple anymore — sized for transient droop on a 0→40 A step, target <1%.

```
Cout ≥ ΔIstep²·L/(2·(Vin−Vout)·ΔVdroop) = 115 µF  →  141 µF installed = 0.81% droop
```
→ 3× `C5750X7R1E476M230KB` — 47 µF · 25 V · ESR 1.5 mΩ (0.5 mΩ parallel)

**Input cap**

Each phase pulls current only while its switch is on, so the input sees pulses, not a smooth draw. `Cin` supplies those fast edges locally so the supply only sees the average. Assume ΔVin = 1V.

```
Cin ≥ Iout·D(1−D)/(fsw·ΔVin) = 34.7 µF
```
→ 3× `C3225X7R1H226M250AC` — 22 µF · 50 V · ESR 1 mΩ (0.33 mΩ parallel)

**MOSFETs**

```
P_gate = 2·Qg·VINTVCC·fsw = 2·49nC·5V·300k = 150 mW
```
→ 2× `BSC010N04LS6` — 40 V · Rds 1 mΩ · Qg 49 nC · Coss 1900 pF

---

## Efficiency

```
P_loss = I²·DCR + P_extvcc + P_sw + P_coss + P_gate + I²·Rds + I_Cin²·ESR + I_Cout²·ESR
```

| Loss | Formula | mW |
| --- | --- | --- |
| Inductor DCR | `4·I_rms²·DCR` | 1320 |
| Switching | `4·½·Vin·I·(tr+tf)·fsw` — V/I overlap during high-side `Vds` transitions | 1160 |
| Coss | `4·½·Coss·Vin²·fsw` | 660 |
| EXTVCC dropper | `2·(Vin−5)·I_bias` — cost of making the 5 V driver rail from 24 V | 630 |
| Gate drive | `4·2·Qg·VINTVCC·fsw` — `Qg` dumped from the 5 V rail each cycle, 2 FETs/phase | 600 |
| Conduction | `4·I²·Rds` | 520 |
| Cin ESR | cancels at `D = 0.5` (4-phase) | ≈0 |
| Cout ESR | cancels at `D = 0.5` (4-phase) | ≈0 |

```
η = Pout/(Pout + P_loss) = 480/(480 + 4.89) ≈ 99.0 %
```

## Schematic
<img width="600" height="600" alt="image" src="https://github.com/user-attachments/assets/7f7a4061-e2a6-4c26-ac84-79990e92cead" />
<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/e2d9aebd-e8ab-417c-b4fb-8d77f99e3ba4" />
