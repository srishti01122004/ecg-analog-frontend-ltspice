# Analog ECG Front-End — Instrumentation Amplifier + 50Hz Notch Filter

Designed and simulated in LTspice. A two-stage analog signal conditioning circuit for ECG acquisition — amplifies millivolt-scale biopotential signals and rejects 50Hz mains interference.

---

## Motivation

ECG signals from skin-surface electrodes are tiny — typically 0.5–5mV — and are buried under common-mode noise and 50Hz interference from mains power. Before any digital processing can happen, the analog front-end must amplify the signal and reject the noise. This project simulates that front-end.

---

## Block Diagram

```
Electrode Input (differential, ~1mV)
        |
        v
+------------------+
|  3 Op-Amp INA    |  Gain ~200x | Rejects common-mode noise
+------------------+
        |
        v
+------------------+
| Twin-T Notch     |  Attenuates 50Hz mains interference
| Filter + Buffer  |
+------------------+
        |
        v
  Conditioned ECG Output
```

---

## Stage 1 — Three Op-Amp Instrumentation Amplifier

**Topology:** Classic 3 op-amp INA  
**Op-amp used:** LT1014  
**Supply:** ±15V

### Key Components
| Component | Value | Purpose |
|-----------|-------|---------|
| Rg | 100Ω | Sets differential gain |
| R1, R2 (feedback) | 10kΩ each | Input buffer gain |
| R3, R4, R5, R6 | 10kΩ each (matched) | Difference amplifier stage |

### Gain Calculation
```
Gain = 1 + (2 × R_feedback / Rg)
     = 1 + (2 × 10,000 / 100)
     = 201 ≈ 200x
```

### Simulation Results
- Input: 1mV differential sine wave at 1Hz (simulating ECG)
- Common-mode noise: 1mV at 50Hz
- Output amplitude: ~190mV → confirms gain of ~190x (close to theoretical 201x, deviation due to op-amp non-idealities)

---
<img width="1197" height="575" alt="ECG_AFE" src="https://github.com/user-attachments/assets/388c3d5c-228d-4551-b1fb-b05a2c46b491" />

<img width="1364" height="590" alt="in vs out" src="https://github.com/user-attachments/assets/91ccada7-922f-4c44-9fac-0927166779d1" />


## Stage 2 — Twin-T Notch Filter + Unity Gain Buffer

**Topology:** Passive Twin-T network followed by op-amp voltage follower  
**Notch frequency:** 50Hz

### Key Components
| Component | Value |
|-----------|-------|
| R9, R10 (series, top path) | 10kΩ each |
| R11 (shunt, bottom path) | 20kΩ |
| C1, C2 (series, bottom path) | 320nF each |
| C3 (shunt, top path) | 160nF |
| Buffer op-amp | LT1014, unity gain |

### Notch Frequency Calculation
```
f_notch = 1 / (2π × R × C)
        = 1 / (2π × 10,000 × 320×10⁻⁹)
        = 49.7Hz ≈ 50Hz ✓
```

### Why the Buffer?
A passive Twin-T filter is highly sensitive to load impedance. Without a unity-gain buffer at the output, the following stage loads the filter and degrades the notch depth. The voltage follower presents near-infinite input impedance, preserving the filter's performance.

### Simulation Results
- Notch center frequency confirmed at **50.93Hz** from AC analysis
- Visible attenuation of 50Hz ripple in transient simulation
- ECG envelope (1Hz) passes through cleanly



<img width="1358" height="624" alt="transient op" src="https://github.com/user-attachments/assets/25d3d75d-16c3-44d2-9062-eb4104c5d3c5" />

---

## AC Frequency Response

The Bode plot (dB scale) shows:
- Flat passband gain of ~46dB across the ECG frequency band (0.5–150Hz)
- Notch dip centered at ~50Hz confirming filter tuning
- Gain recovery above and below the notch frequency

<img width="1358" height="621" alt="Ac analysis" src="https://github.com/user-attachments/assets/5bfc16cd-e3b8-47a3-a4cb-867175cda177" />


---

## Key Specifications

| Parameter | Value |
|-----------|-------|
| Differential gain | ~200x (46dB) |
| Input signal range | 1mV (ECG scale) |
| Supply voltage | ±15V |
| Notch frequency | ~50Hz |
| Op-amp model | LT1014 (LTspice) |
| Simulation tool | LTspice 26.0.1 |

---

## Files

| File | Description |
|------|-------------|
| `Draft6.asc` | LTspice schematic |
| `screenshots/` | Simulation result screenshots |
| `README.md` | This file |

---

## What I Learned

- How resistor mismatch in the difference amplifier stage degrades CMRR and allows common-mode noise to appear at the output
- Why passive filters require output buffering to function correctly under load
- The trade-off between notch depth and bandwidth in Twin-T filters
- Practical gain verification: comparing theoretical vs simulated values

---

## References

- Horowitz & Hill, *The Art of Electronics* — INA design principles
- Texas Instruments Application Note SLOA034 — ECG front-end design
- LT1014 Datasheet — Linear Technology
