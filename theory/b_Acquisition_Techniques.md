# 🎯 EEG Acquisition Techniques

> **Objective:** Equip you to design, run, and quality‑assure EEG acquisitions that are robust for source localization, ERPs, and spectral/FC analyses — from cap selection and electrode physics to noise mitigation, digitization, and safety.

---

## Table of Contents

1. [Electrode Placement & Standards](#electrode-placement--standards)
   1.1 [International 10–20 System](#international-10–20-system)
   1.2 [10–10 and “5%” High-Resolution Systems](#10–10-and-5-high-resolution-systems)
   1.3 [Labels, Landmarks, and Nomenclature](#labels-landmarks-and-nomenclature)
2. [High-Density EEG Systems](#high-density-eeg-systems)
3. [Electrode Types & Methods](#electrode-types--methods)
   3.1 [Wet (Gel/Paste/Saline)](#wet-gelpastesaline)
   3.2 [Dry & Hybrid](#dry--hybrid)
   3.3 [Active vs. Passive Electrodes](#active-vs-passive-electrodes)
   3.4 [Material Science & Interface Models](#material-science--interface-models)
4. [Acquisition Hardware & Settings](#acquisition-hardware--settings)
   4.1 [Amplifiers, CMRR & DRL](#amplifiers-cmrr--drl)
   4.2 [Sampling, Anti‑aliasing & Quantization](#sampling-anti-aliasing--quantization)
   4.3 [Ground, Reference & Montages](#ground-reference--montages)
5. [Factors Influencing Signal Quality](#factors-influencing-signal-quality)
   5.1 [Electrode–Skin Impedance](#electrode–skin-impedance)
   5.2 [Environmental Noise](#environmental-noise)
   5.3 [Physiological & Movement Artifacts](#physiological--movement-artifacts)
6. [Electrode Localization Methods](#electrode-localization-methods)
   6.1 [Digitization: Polhemus, Photogrammetry, Structured Light](#digitization-polhemus-photogrammetry-structured-light)
   6.2 [Coregistration with MRI & Head Models](#coregistration-with-mri--head-models)
   6.3 [Accuracy, Error Budgets & Best Practices](#accuracy-error-budgets--best-practices)
7. [Checklists & SOPs](#checklists--sops)
8. [Suggested Figures](#suggested-figures)
9. [References & Further Reading](#references--further-reading)

---

## Electrode Placement & Standards

### International 10–20 System

The **10–20 system** places electrodes at percentages (10% or 20%) of nasion–inion and left–right preauricular distances to ensure reproducible coverage. It balances setup time with coverage of major lobes.

**Landmarks**

* **Nasion** (bridge of nose)
* **Inion** (occipital protuberance)
* **LPA/RPA** (left/right preauricular points)

**Key properties**

* Odd numbers = left hemisphere; even = right; **z** = midline.
* Regions: Fp, F, FC, C, CP, P, PO, O, T, FT, TP.

> **Tip:** For source localization, include **fiducials** (nasion, LPA, RPA) and a set of **scalp points** for head-shape capture (improves coregistration stability).

### 10–10 and “5%” High-Resolution Systems

* **10–10** inserts intermediate positions (e.g., AF3, FC1) — roughly doubles coverage.
* **5% system** (Oostenveld & Praamstra, 2001) achieves near-uniform sampling for high-resolution ERP/EEG and better **leadfield conditioning**.

**Why it matters:** Higher spatial sampling reduces interpolation error, improves topography, and can **stabilize distributed inverse solvers** (MNE/wMNE/LORETA variants) when combined with accurate head models.

### Labels, Landmarks, and Nomenclature

* Use **standard names** (e.g., Fp1/2, F7/8, T7/8) and extended sets for HD EEG.
* Maintain a **cap map** JSON/TSV (channel name, x/y/z or θ/φ), version‑controlled with acquisition metadata.

---

## High-Density EEG Systems

**Typical densities:** 64, 128, 256 (geodesic nets common at 128/256).

**Advantages**

* Better spatial sampling → improved **topographic precision** and **source localization**.
* Enhanced **functional connectivity** and **graph metrics** (less spatial aliasing).
* More robust **reference-free** estimators (surface Laplacian, Hjorth).

**Trade-offs**

* **Setup time** and **operator load** increase (especially with hair).
* **Bridging risk** (saline) and **dry-out drift** (gel) for long sessions.
* **Computation & storage** overhead; more stringent artifact handling.

**Rule-of-thumb (spatial sampling):** If skull/CSF smoothing yields an effective spatial correlation length **λ \~ 3–6 cm**, Nyquist suggests inter-electrode spacing **≤ λ/2**. HD arrays approach this in many regions, especially when combined with subject‑specific BEM/FEM.

---

## Electrode Types & Methods

### Wet (Gel/Paste/Saline)

* **Gel/paste Ag/AgCl** (sintered preferred): low polarization, stable half‑cell potentials, **low impedance**.
* **Saline nets** (sponges): rapid setup, good for HD; risk of **salt bridging** and **amplitude drift** (evaporation).

**Pros:** High SNR, stable across long recordings (with maintenance).
**Cons:** Prep time, clean‑up, potential skin irritation; saline nets need frequent rewetting.

### Dry & Hybrid

* **Dry pin/spring electrodes:** rapid, mobile, better for short sessions/BCI.
* **Hybrid** (minimal gel films, hydrogel, microneedle arrays): compromise between comfort and SNR.

**Pros:** Fast setup, field deployable, less mess.
**Cons:** Higher impedance, **motion/triboelectric** susceptibility, variable performance in dense hair.

### Active vs. Passive Electrodes

* **Passive:** electrode → cable → amplifier. Needs excellent shielding; cable motion can microphonic‑couple noise.
* **Active:** preamp at electrode **buffers** high impedance, reducing cable artifacts and common‑mode to differential conversion.

**When to choose active:** mobile/VR, dry electrodes, electrically noisy environments, long leads.

### Material Science & Interface Models

Electrode–skin can be approximated by a **Randles circuit**:

$$
Z_e(\omega) \approx R_s + \frac{1}{j\omega C_{dl}} \parallel R_{ct}
$$

* $R_s$ series resistance (electrolyte/gel), $C_{dl}$ double‑layer capacitance, $R_{ct}$ charge‑transfer resistance.
* **Ag/AgCl** is quasi‑nonpolarizable → smaller $R_{ct}$, better low‑freq fidelity.

**Thermal (Johnson) noise:**

$$
v_{n,\text{rms}} = \sqrt{4k_B T R B}
$$

Example (5 kΩ, 300 K, 100 Hz bandwidth): $v_n \approx 0.09~\mu \text{V}$ (negligible vs EEG 10–100 µV).

---

## Acquisition Hardware & Settings

### Amplifiers, CMRR & DRL

* **Differential amplifiers** reject common-mode noise; **CMRR** > 100 dB is typical.
* **Driven Right Leg (DRL)** (or driven ground): feeds back inverted common‑mode to the participant, reducing line pickup.

**Effective rejection depends on impedance balance**: If electrode impedances at inputs differ by $\Delta Z$, common‑mode leakage to differential grows \~proportionally to $\Delta Z$ relative to input impedance. **Balance impedances** across channels to preserve CMRR.

### Sampling, Anti‑aliasing & Quantization

* **Sampling rate:** ≥ 500 Hz for broadband EEG; ≥ 1 kHz if analyzing high beta/gamma or TMS‑EEG.
* **Analog anti‑aliasing:** low‑pass (e.g., 3rd–5th order) with $f_c < f_s/2$.
* **ADC resolution:** 24‑bit delta‑sigma common; ensure **input‑referred noise** < 1 µV\_rms and **dynamic range** covers ±(200–1000) µV.

**Quantization noise (uniform quantizer):**

$$
q = \frac{\text{FSR}}{2^N}, \quad v_{q,\text{rms}}=\frac{q}{\sqrt{12}}
$$

With FSR = 10 mV and 24‑bit, $v_{q,\text{rms}} \approx 0.00018~\mu\text{V}$ (negligible).

### Ground, Reference & Montages

* **Ground**: stable, low‑impedance contact (often near FPz/AFz or mastoid).
* **Reference options**: single electrode (Cz, FCz), **linked mastoids**, **average reference** (robust with HD), **REST** (infinite reference modeling).

> For source localization, **average reference or REST** is generally preferred **after** bad channels are removed and channels are well distributed.

---

## Factors Influencing Signal Quality

### Electrode–Skin Impedance

* Target per‑channel **< 5 kΩ** for passive wet systems (≤ 20 kΩ often fine with high‑Z front‑ends); for dry/active, higher impedances may be acceptable if SNR is validated.
* Abrasion and conductive prep reduce the capacitive reactance and drift.

**Impedance vs noise (intuition):** Higher $Z_e$ increases susceptibility to **motion artifacts** and reduces **CMRR** effectiveness, even if pure thermal noise remains small.

### Environmental Noise

* **50/60 Hz mains** and harmonics: mitigate via **short leads**, **twisted pairs**, **Faraday enclosure**, **battery power**, **DRL**.
* Avoid **ground loops** (single‑point ground), keep **stimulators** opto‑isolated.
* **Notch filters** help but can distort phase/ERPs; prefer **clean hardware** first and use **zero‑phase FIR** if filtering is required post‑hoc.

### Physiological & Movement Artifacts

* **EOG** (blinks/saccades): 0.1–15 Hz dominant; place vertical & horizontal EOG channels.
* **EMG** (muscle): broad 20–300 Hz; jaw/forehead/neck.
* **ECG**: R‑peaks bleed into nearby electrodes (especially neck/temporal).
* **Motion/triboelectric**: cable rub, connector microphonics → use **strain relief**, **braided/shielded** leads, **active electrodes** where possible.

---

## Electrode Localization Methods

### Digitization: Polhemus, Photogrammetry, Structured Light

* **EM digitizers** (e.g., Polhemus FASTRAK): point‑by‑point; robust, operator‑dependent; typical 3–5 mm RMS error.
* **Photogrammetry** (multi‑camera or smartphone array): fast, dense scalp mesh; accuracy 1–3 mm with good calibration.
* **Structured light/laser scanning**: high fidelity surface meshes; minimal manual clicking.

Best practices:

* Capture **fiducials** (nasion, LPA, RPA) and ≥ 100 **head‑shape points**.
* Validate **cap fit** (sagittal alignment, midline z‑channels) before digitizing.

### Coregistration with MRI & Head Models

* Register digitized points to the **subject’s MRI** (preferred) or a **template (MNI)** via **rigid ICP/Procrustes**.
* Extract **BEM/FEM** surfaces (scalp, skull, CSF, brain).
* Verify **residual error** (< 3 mm ideal) and visually inspect electrode projections on the scalp surface.

### Accuracy, Error Budgets & Best Practices

* **Electrode position** errors (5–10 mm) can bias inverse solutions and functional mapping; HD sampling mitigates but does not eliminate.
* **Skull conductivity** uncertainty is a major source of localization error; consider **calibration** or literature priors.
* Maintain **consistent cap indexing** across sessions; version‑control digitization files.

---

## Checklists & SOPs

### Pre‑Session (15–30 min)

* Confirm **hardware self‑test**, firmware, battery.
* Inspect caps/electrodes; prepare **Ag/AgCl** (sintered preferred).
* Measure **head circumference**, select cap size; mark **fiducials**.
* Configure amplifier: **sampling rate**, **band limits**, **input range**, **reference**.
* Prepare **EOG/ECG** channels if needed.

### Setup (30–60+ min; HD can take longer)

* Seat participant comfortably; minimize cable strain.
* Skin prep (clean, light abrasion if approved).
* Apply gel/paste or saline; target impedance **< 5 kΩ** (passive wet).
* Balance impedances across inputs (reduce $\Delta Z$).
* Verify **drift** and **offsets**; fix bridges/leaks.

### Pre‑Record QC

* Eyes‑open/closed baselines; check **alpha** emergence posteriorly.
* **Line spectrum** check (expect narrow peak at 50/60 Hz only).
* **Impedance map** snapshot; log channel notes.

### During Recording

* Monitor **real‑time topographies** and spectrum; mark **events/artifacts**.
* Re‑wet saline nets periodically; recheck problem channels.

### Post‑Record

* Digitize electrodes + extra head points; export in standard coordinate system.
* Archive **raw + metadata**: cap map, impedance log, digitization file, amplifier config, room notes.

---

## Suggested Figures

1. **10–20 vs 10–10 vs 5% layouts** (overhead schematic with fiducials and spacing percentages).
2. **Electrode–skin interface model** (Randles circuit) with Bode magnitude/phase sketch.
3. **CMRR concept**: common‑mode pickup and effect of impedance mismatch.
4. **Noise sources**: mains, EOG/EMG spectra overlays; hardware vs software mitigation.
5. **Digitization & coregistration pipeline**: fiducials → scalp mesh → ICP alignment → electrodes on MRI scalp → BEM surfaces.

> I can generate vector diagrams (SVG/PNG) for slides on request.

---

## References & Further Reading

**Books**

* Schomer, D. L., & Lopes da Silva, F. H. (2017). *Niedermeyer’s Electroencephalography* (7th ed.). OUP.
* Luck, S. J. (2014). *An Introduction to the ERP Technique* (2nd ed.). MIT Press.

**Standards & Safety**

* IEC 60601‑1 (Medical electrical equipment — General requirements).
* IEC 60601‑2‑26 (Particular requirements for **electroencephalographs**).
* Manufacturer application notes on DRL/CMRR and input‑referred noise (check your amplifier’s datasheet).

**Scientific Articles**

* Oostenveld, R., & Praamstra, P. (2001). The five percent electrode system for high‑resolution EEG/ERP. *Clin Neurophysiol*, 112(4), 713–719.
* Kappenman, E. S., & Luck, S. J. (2010). Electrode impedance effects on ERP data quality. *Psychophysiology*, 47(5), 888–904.
* Lopez‑Gordo, M. A., Sanchez‑Morillo, D., & Valle, F. P. (2014). Dry EEG electrodes. *Sensors*, 14(7), 12847–12870.
* Ferree, T. C., Luu, P., Russell, G. S., & Tucker, D. M. (2001). Scalp electrode positions and their systematic deviations. *Electroencephalogr Clin Neurophysiol*.
* Koessler, L. et al. (2011). Automated cortical projection of EEG sensors: anatomical correlation via MRI. *NeuroImage*, 57(3), 939–948.
* McCann, H., Pisano, G., & Beltrachini, L. (2019). Variation in reported head tissue conductivities. *Brain Topogr*, 32, 825–858.

**Online Resources**

* EEGLAB: Electrode positioning & channel locations — [https://eeglab.org/tutorials/ChannelLocation](https://eeglab.org/tutorials/ChannelLocation)
* FieldTrip: Electrode placement — [https://www.fieldtriptoolbox.org/tutorial/electrode/](https://www.fieldtriptoolbox.org/tutorial/electrode/)
* MNE‑Python: Sensor locations, coregistration, and forward models — [https://mne.tools/stable/auto\_tutorials](https://mne.tools/stable/auto_tutorials)

---

## Instructor Notes (Optional)

* Pair this with a **hands‑on lab**: run impedance optimization on 10 participants; quantify spectral SNR and blink ERP after each optimization step.
* Include a **hardware demo** comparing passive vs active electrodes under induced motion.
* Provide a **digitization practical**: students align electrode sets to an MRI scalp and report RMS residuals and their impact on MNE localization.
