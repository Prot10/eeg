# 📊 EEG Frequency Bands & Neurophysiological Meaning

> **Goal:** Interpret EEG rhythms without folklore. Learn defensible measurement (PSD, connectivity, CFC), individualized banding, and common confounds (muscle, aperiodic 1/f).

---

## Contents

- [📊 EEG Frequency Bands \& Neurophysiological Meaning](#-eeg-frequency-bands--neurophysiological-meaning)
  - [Contents](#contents)
  - [Foundations: Spectral Estimation \& Aperiodic Signal](#foundations-spectral-estimation--aperiodic-signal)
    - [Power Spectral Density (PSD)](#power-spectral-density-psd)
    - [The Aperiodic (1/f) Component](#the-aperiodic-1f-component)
  - [Individualized Banding (IAF) \& Developmental Effects](#individualized-banding-iaf--developmental-effects)
    - [Individual Alpha Frequency (IAF)](#individual-alpha-frequency-iaf)
    - [Age \& State](#age--state)
  - [Frequency Bands: Physiology, Cognition, Pitfalls](#frequency-bands-physiology-cognition-pitfalls)
    - [🔹 Delta (0.5–4 Hz)](#-delta-054hz)
    - [🔹 Theta (4–8 Hz)](#-theta-48hz)
    - [🔹 Alpha (8–13 Hz)](#-alpha-813hz)
    - [🔹 Beta (13–30 Hz)](#-beta-1330hz)
    - [🔹 Gamma (\>30 Hz; often low 30–55, high 65–150 Hz)](#-gamma-30hz-often-low-3055-high-65150hz)
  - [Band Ratios: When (Not) to Use](#band-ratios-when-not-to-use)
  - [Connectivity in Frequency Domain](#connectivity-in-frequency-domain)
    - [Coherence (magnitude‑squared)](#coherence-magnitudesquared)
    - [Imaginary Coherency / wPLI](#imaginary-coherency--wpli)
    - [Amplitude‑Envelope Correlation (AEC)](#amplitudeenvelope-correlation-aec)
    - [Multivariate (MVAR) Spectral Causality](#multivariate-mvar-spectral-causality)
  - [Cross‑Frequency Coupling (CFC)](#crossfrequency-coupling-cfc)
    - [Phase–Amplitude Coupling (PAC)](#phaseamplitude-coupling-pac)
    - [Phase–Phase / Amplitude–Amplitude](#phasephase--amplitudeamplitude)
  - [Normalization, Reactivity \& QC](#normalization-reactivity--qc)
    - [Power Normalization](#power-normalization)
    - [Alpha Reactivity (QC)](#alpha-reactivity-qc)
    - [Minimal QC Panel](#minimal-qc-panel)
  - [Suggested Figures](#suggested-figures)
  - [References \& Resources](#references--resources)
    - [Instructor Notes (optional)](#instructor-notes-optional)

---

## Foundations: Spectral Estimation & Aperiodic Signal

### Power Spectral Density (PSD)

Given channels $x_c[t]$, estimate PSD to quantify band power.

* **Welch’s method** (variance‑reduced):

  * Segment length $N$, overlap $O$, window $w[n]$, sampling $f_s$.
  * FFT each segment, average magnitude squares:

  $$
  \hat{S}_{xx}(f) = \frac{1}{K}\sum_{k=1}^{K} \frac{1}{U}\left|\sum_{n=0}^{N-1} x_k[n]\, w[n]\, e^{-j 2\pi fn/f_s}\right|^2
  $$

  where $U$ normalizes window power.

* **Multitaper (DPSS)**

  * $K$ Slepian tapers, time‑bandwidth $NW$:

  $$
  \hat{S}_{xx}(f) = \frac{1}{K} \sum_{k=1}^{K} \left| \sum_{n} x[n]\, v_k[n]\, e^{-j2\pi fn/f_s} \right|^2
  $$

  Better bias/variance control for short data and narrow peaks.

**Resolution:** $\Delta f \approx \tfrac{f_s}{N_{\text{eff}}}$. Choose $N_{\text{eff}}$ to resolve your band of interest (e.g., ≥1 s for alpha peaks).

### The Aperiodic (1/f) Component

EEG PSD ≈ **aperiodic** $1/f^\chi$ background + **periodic** peaks:

$$
\log_{10} S(f) \approx b - \chi \log_{10} f \;+\; \sum_i \mathcal{N}(f;\,\mu_i,\sigma_i^2)
$$

Changes in slope $\chi$ (excitation/inhibition proxies) can masquerade as band “power changes.” Parameterize/“FOOOF” the spectrum before comparing band powers; report both **aperiodic‑adjusted** and **raw** metrics.

---

## Individualized Banding (IAF) & Developmental Effects

### Individual Alpha Frequency (IAF)

Alpha center varies (∼8–12+ Hz) across people and tasks. Anchor bands to **IAF** (peak in eyes‑closed occipital PSD):

* **Theta** ≈ IAF − 6 to IAF − 4
* **Alpha** ≈ IAF − 2 to IAF + 2 (or split α1/α2 around IAF)
* **Beta** ≈ IAF + 2 to IAF + 20

This improves sensitivity when comparing subjects or longitudinal data.

### Age & State

* **Infancy/childhood:** high delta/theta, slower alpha; alpha frequency increases with maturation.
* **Aging/dementia:** global slowing (↑delta/theta, ↓alpha peak).
* **Arousal/eyes state:** alpha reactivity (EC > EO).

---

## Frequency Bands: Physiology, Cognition, Pitfalls

> Band boundaries are conventional; physiology is continuous. Always interpret with task, state, and aperiodic slope in mind.

### 🔹 Delta (0.5–4 Hz)

**Generators/Mechanisms**

* Cortico‑thalamic slow oscillations; cortical UP/DOWN state dynamics in sleep.
* Large‑scale synchronization; high amplitude, broad topography.

**Cognition/State**

* NREM (SWS) hallmark; in wakefulness, prominent delta suggests reduced vigilance or pathology.

**Clinical**

* **Focal/hemispheric lesions** (stroke/TBI/tumors): focal or diffuse delta.
* **Anoxia/encephalopathies:** diffuse slowing.

**Pitfalls**

* **Sweat/skin potentials** and slow drifts bleed into delta—ensure appropriate HPF (e.g., 0.1 Hz for ERP; ≥0.3–0.5 Hz for resting‑state) and artifact control.

---

### 🔹 Theta (4–8 Hz)

**Generators/Mechanisms**

* Fronto‑midline theta in cognitive control networks; hippocampal‑cortical interactions (source contributions to scalp are indirect).

**Cognition**

* Encoding/retrieval, working memory load, conflict monitoring; increases over midline frontal sites in demanding tasks.

**Clinical**

* **Frontal theta** may increase in mild cognitive impairment/Alzheimer’s; **TBR in ADHD** is debated and not specific.

**Pitfalls**

* Eye movements and **saccadic spike potentials** can inflate low‑theta; include EOG regressors and inspect topographies.

---

### 🔹 Alpha (8–13 Hz)

**Generators/Mechanisms**

* Posterior dominant rhythm from occipito‑parietal networks modulated by thalamo‑cortical loops; sensorimotor alpha (mu) over central sites.

**Cognition**

* **Inhibition/gating**: alpha increases with suppression of task‑irrelevant regions; decreases (desynchronizes) with active processing.
* **Alpha reactivity**: Eyes‑closed > eyes‑open amplitude drop is a robust QC marker.

**Clinical**

* In **depression/anxiety**, alpha patterns can be dysregulated (topography/hemispheric asymmetry literature mixed).
* **Stroke rehab**: alpha normalization and connectivity changes may track recovery.

**Pitfalls**

* Reference choice alters alpha maps; use **AR/REST** post‑cleaning.
* Distinguish occipital alpha from central **mu** (8–13 Hz but sensorimotor and task‑locked to movement).

---

### 🔹 Beta (13–30 Hz)

**Generators/Mechanisms**

* Sensorimotor networks; corticospinal loops; GABAergic modulation.

**Cognition/Behavior**

* **ERD/ERS** with movement: beta **desynchronization** during movement/imagery, **rebound** post‑movement.
* Sustained beta in attention/maintenance.

**Clinical**

* **Parkinson’s disease:** excessive beta synchronization in basal ganglia‑cortical circuits (invasive recordings); scalp metrics vary with task/medication.
* **Anxiety/stress:** some studies report elevated diffuse beta at rest—interpret cautiously.

**Pitfalls**

* Line‑harmonic contamination (50/60 Hz multiples) and **EMG** above \~20 Hz; verify with Laplacian/CSD and peripheral EMG channels.

---

### 🔹 Gamma (>30 Hz; often low 30–55, high 65–150 Hz)

**Generators/Mechanisms**

* Local cortical circuits (fast‑spiking interneurons + pyramidal cells, PING/ING).
* Short‑range synchronization; limited spatial spread to scalp; many findings come from ECoG/invasive.

**Cognition**

* Sensory binding, attention, working memory; task‑locked narrowband gamma in visual cortex (often 40–70 Hz).

**Clinical**

* Altered gamma synchrony in **schizophrenia** and **ASD** (interneuron dysfunction hypotheses).
* **Epilepsy:** high‑frequency oscillations (80–250 Hz) intra‑cranially; scalp access is limited and artifact‑prone.

**Pitfalls (critical)**

* **EMG contamination** dominates >30 Hz at the scalp (forehead/jaw/neck).
* **Microsaccades** produce broadband “false gamma.” Always combine: artifact channels, CSD, and stringent preprocessing; prefer **task‑locked, narrowband** gamma with clear topography.

---

## Band Ratios: When (Not) to Use

**Common ratios:** theta/alpha, alpha/beta, theta/beta (TBR), delta/alpha.

**Problems**

* Sensitive to **aperiodic slope**, reference, and noise; non‑Gaussian distributions.
* TBR for ADHD is **non‑specific** and population‑dependent.

**Better practice**

* Use **aperiodic‑adjusted band power** (subtract 1/f fit), log‑transform, and report **effect sizes**.
* Prefer **task‑specific markers** (e.g., alpha ERD in a defined ROI) over global ratios.

---

## Connectivity in Frequency Domain

Let $X(f), Y(f)$ be Fourier transforms of two channels (or sources).

### Coherence (magnitude‑squared)

$$
C_{xy}(f)=\frac{|\mathbb{E}[X(f)Y^*(f)]|^2}{\mathbb{E}[|X(f)|^2]\;\mathbb{E}[|Y(f)|^2]}
$$

**Pros:** Simple, interpretable.
**Cons:** Inflated by **volume conduction** and common reference.

### Imaginary Coherency / wPLI

* **Imaginary part of coherency** focuses on non‑zero phase‑lag interactions (Nolte 2004).
* **Weighted Phase‑Lag Index (wPLI)** (Vinck 2011):

$$
\text{wPLI}(f)=\frac{|\mathbb{E}[\text{sign}(\Im\{S_{xy}(f)\}) \cdot |\Im\{S_{xy}(f)\}|]|}{\mathbb{E}[|\Im\{S_{xy}(f)\}|]}
$$

**Pros:** Reduces zero‑lag leakage.
**Cons:** Less sensitive to true near‑zero‑lag physiology.

### Amplitude‑Envelope Correlation (AEC)

Band‑pass → Hilbert amplitude → correlate envelopes (often orthogonalized to reduce leakage).

### Multivariate (MVAR) Spectral Causality

* Fit MVAR: $\mathbf{x}_t = \sum_{k=1}^{p} \mathbf{A}_k \mathbf{x}_{t-k} + \mathbf{e}_t$.
* Derive **Directed Transfer Function (DTF)**, **Partial Directed Coherence (PDC)**.
  **Caveats:** Requires stationarity, sufficient samples; sensitive to model order and preprocessing.

**Best practice**

* Compute on **source space** (after inverse) with **Laplacian/AR/REST**, use leakage‑robust metrics (imaginary/wPLI/orthogonalization), and apply **nonparametric surrogates** for significance.

---

## Cross‑Frequency Coupling (CFC)

### Phase–Amplitude Coupling (PAC)

Does low‑frequency phase $\phi_L(t)$ modulate high‑frequency amplitude $A_H(t)$?

* **Modulation Index (MI; Tort)**: compute amplitude distribution across phase bins, quantify deviation from uniform (K–L divergence).
* **Canolty PAC (MI\_complex):** $ \text{MI} = | \mathbb{E}[A_H(t)\,e^{j\phi_L(t)}] |$.

**Controls**

* Use **narrow bands** with minimal leakage, appropriate filter orders, and **surrogates** (time‑shift, trial shuffle).
* Remove **EMG** and microsaccades; PAC is extremely artifact‑sensitive—especially gamma amplitude.

### Phase–Phase / Amplitude–Amplitude

* **n\:m phase locking** (PLV on phase difference $n\phi_L - m\phi_H$).
* **AAC**: correlate band‑limited amplitudes; orthogonalize to reduce leakage.

---

## Normalization, Reactivity & QC

### Power Normalization

* **Relative power**: band/total power (but confounded by aperiodic slope).
* **dB change from baseline**: $10\log_{10}\frac{P_{\text{task}}}{P_{\text{baseline}}}$.
* **Aperiodic‑adjusted**: subtract fitted 1/f before ratios or dB.

### Alpha Reactivity (QC)

Compute EC vs EO alpha power:

$$
\Delta_\alpha = 10\log_{10}\frac{P_\alpha^{\text{EC}}}{P_\alpha^{\text{EO}}}
$$

Expect positive occipital values; flat reactivity flags issues (setup, vigilance, processing).

### Minimal QC Panel

* PSD (0.5–70 Hz), highlight 50/60 Hz and harmonics.
* Topographies of band power (delta–beta), and **gamma with CSD**.
* Aperiodic slope/intercept.
* Eyes‑closed alpha peak & reactivity.

---

## Suggested Figures

1. **PSD decomposition**: raw spectrum with 1/f fit + Gaussian peaks (alpha, beta).
2. **IAF anchoring**: subject spectra showing individualized bands, vs fixed 8–13 Hz.
3. **Band topographies**: delta/theta frontal vs posterior alpha vs central mu.
4. **Connectivity maps**: coherence vs wPLI (source‑space), illustrating volume conduction bias.
5. **PAC diagram**: low‑freq phase histogram modulating high‑freq amplitude.
6. **Gamma pitfalls**: EMG spectrum & topography vs true visual gamma patch.

(Ask if you want me to generate these as clean SVGs.)

---

## References & Resources

**Books & Overviews**

* Buzsáki, G. (2006). *Rhythms of the Brain*. OUP.
* Cohen, M. X. (2014). *Analyzing Neural Time Series Data*. MIT Press.
* Mitra, P., & Bokil, H. (2008). *Observed Brain Dynamics*. Oxford.

**Alpha/Theta Cognition**

* Klimesch, W. (1999). Alpha/theta oscillations and memory. *Brain Res Rev*, 29, 169–195.
* Palva, S., & Palva, J. M. (2007). New vistas for alpha. *Trends Neurosci*, 30, 150–158.

**Connectivity Metrics**

* Nolte, G. et al. (2004). Imaginary part of coherency. *Clin Neurophysiol*, 115, 2292–2307.
* Vinck, M. et al. (2011). wPLI. *NeuroImage*, 55, 1548–1565.
* Bastos, A. M., & Schoffelen, J.‑M. (2016). Tutorial on FC measures. *eNeuro*.

**Aperiodic Spectra**

* Donoghue, T. et al. (2020). Parameterizing neural power spectra (FOOOF). *Nat Neurosci*, 23, 1655–1665.

**Gamma & CFC**

* Hyafil, A. et al. (2015). CFC in cognition. *Trends Neurosci*, 38, 725–740.
* Ray, S., & Maunsell, J. (2015). Do gamma oscillations carry information? *Trends Cogn Sci*, 19, 78–85.
* Yuval‑Greenberg, S. et al. (2008). Microsaccades and induced gamma. *Neuron*, 58, 429–441.

**Clinical**

* Finnigan, S., & van Putten, M. (2013). qEEG in stroke. *Clin Neurophysiol*, 124, 10–19.
* Uhlhaas, P. & Singer, W. (2010). Abnormal gamma in schizophrenia. *Nat Rev Neurosci*, 11, 100–113.

**Tool Tutorials**

* EEGLAB: Spectral Analysis — [https://eeglab.org/tutorials/05\_Spectral\_analysis.html](https://eeglab.org/tutorials/05_Spectral_analysis.html)
* FieldTrip: Frequency Analysis — [https://www.fieldtriptoolbox.org/tutorial/frequencyanalysis/](https://www.fieldtriptoolbox.org/tutorial/frequencyanalysis/)
* MNE‑Python: Spectral/Connectivity — [https://mne.tools/stable/auto\_tutorials/time-freq/](https://mne.tools/stable/auto_tutorials/time-freq/)

---

### Instructor Notes (optional)

* Lab: Compute PSD with **Welch vs multitaper**, then **FOOOF**; compare raw vs aperiodic‑adjusted alpha/beta power.
* Lab: EC/EO alpha reactivity + IAF estimation per subject; anchor individualized bands.
* Lab: Connectivity on **sensor vs source space**; compare coherence vs wPLI vs AEC‑orth.
* Advanced: PAC with rigorous surrogates and EMG control; discuss false positives.
