# Analog ECG Front-End — INA + 50Hz Notch Filter + Right Leg Drive

Designed and simulated in LTspice. A complete analog signal conditioning circuit for ECG acquisition — amplifies millivolt-scale biopotential signals, actively cancels common-mode noise via right leg drive, and rejects residual 50Hz mains interference.

---

## Motivation

ECG signals from skin-surface electrodes are tiny — typically 0.5–5mV — and are buried under common-mode noise and 50Hz interference from mains power. Before any digital processing can happen, the analog front-end must amplify the signal and reject the noise. This project simulates that complete front-end, including active noise cancellation via a Right Leg Drive circuit.

---

## Block Diagram

```
Electrode Input (RA, LA — differential, ~1mV)
        |              |
        |              +-------> Averaging Network
        |                               |
        v                               v
+------------------+         +------------------+
|  3 Op-Amp INA    |         |  RLD Circuit     |
|  Gain ~200x      |         |  (U5, inverting) |
|  Rejects CM noise|         |  Gain ~-47x      |
+------------------+         +------------------+
        |                               |
        v                               v
  Amplified ECG              RL Electrode (feedback
                              to patient's right leg)
        |
        v
+------------------+
| Unity Gain Buffer|  Isolates INA from filter loading
+------------------+
        |
        v
+------------------+
| Twin-T Notch     |  Attenuates residual 50Hz interference
| Filter           |  f_notch = 50Hz
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
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/4e49a70d-b1ba-4ba1-b153-752bfa49b4dd" />

<img width="1364" height="590" alt="in vs out" src="https://github.com/user-attachments/assets/91ccada7-922f-4c44-9fac-0927166779d1" />

---

## Stage 2 — Right Leg Drive (RLD) Circuit

**Topology:** Inverting amplifier driven by averaged electrode voltages, output fed back to patient's right leg electrode  
**Op-amp used:** LT1014  

### Key Components
| Component | Value | Purpose |
|-----------|-------|---------|
| R_rld1, R_rld2 | 10kΩ each | Electrode averaging network (senses common-mode) |
| R_rld_in | 10kΩ | Inverting amplifier input resistor |
| R_rld_fb | 470kΩ | Feedback resistor — sets gain ≈ -47x |
| R_body | 51kΩ | Body impedance model (RL electrode load) |
| U5 | LT1014 | Inverting amplifier |

### How It Works
The two averaging resistors (R_rld1, R_rld2) sense the common-mode voltage present at both RA and LA electrodes and sum them at the inverting input of U5. U5 inverts and amplifies this signal by approximately -47x and feeds it back to the patient's right leg via R_body.

This creates a **negative feedback loop**: any common-mode interference (e.g. mains pickup) is sensed, inverted, and injected back into the body — actively cancelling the interference at its source before it propagates through the INA. This is fundamentally more effective than passive filtering alone.

### RLD vs Passive Notch Filter
| | RLD | Twin-T Notch |
|--|-----|--------------|
| Mechanism | Active cancellation at source | Passive attenuation after amplification |
| Effective against | Broadband common-mode noise | Narrowband (50Hz only) |
| Component count | 5 passives + 1 op-amp | 5 passives + 1 op-amp |
| Used together | ✓ Best practice in real ECG AFE design | ✓ |

In real ECG front-end ICs (e.g. TI ADS1292, Analog Devices AD8232), both RLD and notch filtering are standard.

---

## Stage 3 — Twin-T Notch Filter + Unity Gain Buffer

**Topology:** Unity gain buffer followed by passive Twin-T network  
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
A passive Twin-T filter is highly sensitive to source impedance. Without a unity-gain buffer preceding the filter, the driving stage loads the Twin-T network and shifts the notch frequency away from 50Hz. The voltage follower presents near-zero output impedance to the filter, preserving correct tuning. This was verified during simulation — removing the buffer shifted the notch frequency down to ~30Hz.

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
| RLD feedback gain | ~-47x |
| Body impedance model | 51kΩ |
| Op-amp model | LT1014 (LTspice) |
| Simulation tool | LTspice 26.0.1 |

---

## Files

| File | Description |
|------|-------------|
| `ECG_AFE.asc` | LTspice schematic |
| `screenshots/` | Simulation result screenshots |
| `README.md` | This file |

---

## What I Learned

- How resistor mismatch in the difference amplifier stage degrades CMRR and allows common-mode noise to appear at the output
- Why passive filters require output buffering to function correctly under load — verified by observing notch frequency shift from 50Hz to ~30Hz when buffer was removed
- The trade-off between notch depth and bandwidth in Twin-T filters — a passive Twin-T has moderate Q, so residual 50Hz ripple remains; a DSP stage downstream would handle final rejection in a real system
- How Right Leg Drive works as an active feedback loop to cancel common-mode interference at the source, complementing the passive notch filter
- The difference between active noise cancellation (RLD — broadband, at source) and passive filtering (Twin-T — narrowband, after amplification) and why real ECG ICs use both
- Practical gain verification: comparing theoretical vs simulated values and understanding sources of deviation

---

## References

- Horowitz & Hill, *The Art of Electronics* — INA design principles
- Texas Instruments Application Note SLOA034 — ECG front-end design
- LT1014 Datasheet — Linear Technology
- Prutchi & Norris, *Design and Development of Medical Electronic Instrumentation* — RLD circuit design
