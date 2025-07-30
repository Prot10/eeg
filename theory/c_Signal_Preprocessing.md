# ⚙️ EEG Signal Preprocessing

> **Goal:** Convert raw EEG into analysis‑ready data while preserving neural signal, minimizing distortion, and documenting every transformation for reproducibility and downstream source localization.

---

## Contents

- [⚙️ EEG Signal Preprocessing](#️-eeg-signal-preprocessing)
  - [Contents](#contents)
  - [Design Principles \& Pipeline Overview](#design-principles--pipeline-overview)
  - [Artifact Removal Techniques](#artifact-removal-techniques)
    - [Independent Component Analysis (ICA)](#independent-component-analysis-ica)
    - [Artifact Subspace Reconstruction (ASR)](#artifact-subspace-reconstruction-asr)
    - [Regression‑Based Correction (EOG/ECG/EMG)](#regressionbased-correction-eogecgemg)
    - [SSP / DSS / SSD (SVD‑based methods)](#ssp--dss--ssd-svdbased-methods)
  - [Signal Filtering \& Enhancement](#signal-filtering--enhancement)
    - [High‑Pass \& Low‑Pass](#highpass--lowpass)
    - [Notch / Comb Notch](#notch--comb-notch)
    - [Re‑referencing Strategies](#rereferencing-strategies)
    - [Laplacian / CSD Transform](#laplacian--csd-transform)
  - [Epoching, Baseline, and Averaging](#epoching-baseline-and-averaging)
    - [Epoching](#epoching)
    - [Baseline](#baseline)
    - [Averaging (ERP)](#averaging-erp)
  - [QC Metrics, Thresholds, and Logs](#qc-metrics-thresholds-and-logs)
  - [Suggested Figures](#suggested-figures)
  - [Parameter Defaults / Quick Table](#parameter-defaults--quick-table)
  - [References \& Resources](#references--resources)
    - [Instructor Notes (optional)](#instructor-notes-optional)

---

## Design Principles & Pipeline Overview

**Principles**

* Prefer **linear, time‑invariant** operations before localization (filtering, referencing). Clearly document **nonlinear** steps (e.g., ASR).
* Separate **pre‑ICA** and **post‑ICA** filter settings: a slightly higher high‑pass (e.g., 0.5–1 Hz) stabilizes ICA; final export can be 0.1–40 Hz for ERP if needed.
* Keep a **transform log**: each step is a matrix/operator $\mathbf{L}_i$; cumulative operator $\mathbf{L} = \prod_i \mathbf{L}_i$ applied to raw $\mathbf{X}$.

**Canonical pipeline (production)**

1. Channel audit (dead/bridged) → 2) Robust re‑reference seed (e.g., PREP) → 3) High‑pass (0.5–1 Hz for ICA stability) → 4) Line‑noise mitigation → 5) **ICA** (optionally with bad‑segment masking) → 6) IC classification & removal → 7) Interpolate bad channels → 8) Final reference (average/REST) → 9) Task‑specific filters → 10) Epoch/baseline/average or TFR/FC.

---

## Artifact Removal Techniques

### Independent Component Analysis (ICA)

**Model**
Observed data $\mathbf{X} \in \mathbb{R}^{C \times T}$ is a linear mixture:

$$
\mathbf{X} = \mathbf{A}\mathbf{S},\quad \mathbf{S}=\mathbf{W}\mathbf{X},\; \mathbf{W}=\mathbf{A}^{-1}
$$

Columns of $\mathbf{A}$ are scalp maps; rows of $\mathbf{S}$ are independent time series.

**Algorithms**

* **Infomax / Extended‑Infomax**, **FastICA**, **SOBI** (second‑order), **AMICA** (mixtures; best separation, heavier compute).
* **Dimensionality**: run ICA on rank‑reduced data (PCA to numerical rank after removing flat/zero‑var channels).

**Pre‑ICA prep (robust practice)**

* High‑pass at **0.5–1 Hz** (zero‑phase FIR) to reduce slow drifts that destabilize ICA.
* Mark large segments of gross motion as **bad** (omit from ICA fit).
* Use **line‑noise clean** (multi‑taper or adaptive notch) before ICA.

**IC Identification**

* Eye (EOG): frontal scalp maps, strong low‑freq, blink‑locked;
* Muscle (EMG): lateral/temporal maps, 20–200 Hz power;
* Cardiac (ECG): \~1–2 Hz periodic, polarity around mastoids/neck.
* Tools: **ICLabel** (probabilistic), spectrum/topography templates, correlation with EOG/ECG.

**Remove vs retain**
Zero out artifact ICs: $\mathbf{X}_{\text{clean}} = \mathbf{A}\mathbf{S}_{\text{kept}}$. Keep a YAML/JSON manifest of removed IC indices and labels.

**Failure modes**

* Too few samples (rule of thumb: **> 20× number of channels** in seconds).
* High‑pass too aggressive → **ERP distortions** post‑ICA when projecting back to 0.1 Hz.
* Mislabeling neural ICs as artifact (guard with review + ICLabel thresholding).

---

### Artifact Subspace Reconstruction (ASR)

**Idea**
Estimate a “clean” covariance $\mathbf{C}_0$ from calibration data; detect windows where covariance along any eigenvector exceeds a cutoff; project out excess, reconstruct with subspace limited to $k$-σ deviations.

**Sketch**

1. Compute $\mathbf{C}_0$ on clean baseline.
2. For each sliding window $\mathbf{X}_w$, get covariance $\mathbf{C}_w$.
3. In eigenspace of $\mathbf{C}_0$, cap eigen‑amplitudes exceeding threshold $\tau$ (e.g., **k = 20–30**).
4. Reconstruct $\mathbf{X}_w'$ by shrinking/removing outlier subspace.

**Pros**: Works online; rescues continuous data with bursts.
**Cons**: Nonlinear; can attenuate genuine transients if calibration is too “clean” or threshold too strict. Log affected samples.

---

### Regression‑Based Correction (EOG/ECG/EMG)

**Model (per channel)**

$$
\mathbf{y} = \mathbf{X}\boldsymbol{\beta} + \boldsymbol{\varepsilon},\quad 
\hat{\boldsymbol{\beta}} = (\mathbf{X}^\top \mathbf{X})^{-1}\mathbf{X}^\top \mathbf{y}
$$

* $\mathbf{y}$: EEG channel; $\mathbf{X}$: regressors (VEOG, HEOG, ECG, EMG envelopes, motion).
* Cleaned: $\mathbf{y}_{\text{clean}} = \mathbf{y} - \mathbf{X}\hat{\boldsymbol{\beta}} = (\mathbf{I}-\mathbf{P})\mathbf{y}$, with $\mathbf{P}=\mathbf{X}(\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top$.

**Tips**

* Use **time‑lagged** regressors (±200 ms) to capture blink dynamics.
* Prefer **robust regression** (Huber) when outliers exist.
* Always validate residual spectra and ERP morphology after regression.

---

### SSP / DSS / SSD (SVD‑based methods)

* **SSP** (Signal Space Projection): compute artifact subspace via SVD from blink epochs; project it out.
* **DSS** (Denoising Source Separation): maximize reproducibility across trials (ERP enhancement).
* **SSD** (Spatio‑Spectral Decomposition): optimize SNR in a frequency band (e.g., alpha), useful pre‑connectivity.

**Pros**: Transparent linear operators; good for targeted artifacts or oscillations.
**Cons**: Requires clean examples; risk of removing neural signal if artifact covariates overlap with brain sources.

---

## Signal Filtering & Enhancement

### High‑Pass & Low‑Pass

**Convolutional FIR (zero‑phase)**
Apply forward–reverse filtering (e.g., `filtfilt`), achieving zero phase:

$$
\mathbf{y} = \mathbf{h} * \mathbf{x},\quad \text{then reversed pass}
$$

* Order $N$ for FIR with transition width $\Delta f$ roughly:

$$
N \approx \alpha \frac{f_s}{\Delta f} \quad (\alpha \sim 3\!-\!5\; \text{for Hamming})
$$

* **Group delay** zeroed by forward–reverse, but note **edge transients**: pad with reflection ≥ $3N$.

**Practical defaults**

* **Pre‑ICA HPF**: 0.5–1.0 Hz (FIR, zero‑phase).
* **ERP final**: 0.1–30/40 Hz (task‑dependent).
* **Oscillations**: choose passbands per hypothesis (e.g., theta 3–8 Hz, beta 13–30 Hz), always report exact cutoffs.

**IIR (Butterworth, Biquads)**

* Lower order, sharper transitions, but **nonlinear phase** unless used in forward–reverse mode. For online BCI, causal IIR is acceptable—document **group delay**.

**Filter‑induced distortions**

* High‑pass > 0.3–0.5 Hz can **distort slow ERPs** and baselines (filter ringing). Prefer **baseline regression** plus gentle HPF.

### Notch / Comb Notch

* Single notch at 50/60 Hz (and possibly 100/120 Hz harmonics).
* Better: **multi‑taper regression** of narrowband line components to avoid passband ripples.
* **Do not over‑notch**; validate with spectra pre/post.

### Re‑referencing Strategies

Reference is a **linear transform** $\mathbf{X}' = \mathbf{R}\mathbf{X}$.

* **Average reference (AR)**: $\mathbf{R} = \mathbf{I} - \frac{1}{C}\mathbf{1}\mathbf{1}^\top$ (after removing bads & ensuring decent coverage).
* **Linked mastoids**: common clinically; can inject mastoid noise.
* **REST**: infinite‑reference estimate from head model; good for source analyses when model is available.

**Best practice**

* Do initial robust reference (e.g., **PREP** pipeline: remove bads → robust average reference).
* For localization, use **AR** or **REST** post‑cleaning.

### Laplacian / CSD Transform

Enhances spatial resolution by referencing each electrode to its neighbors (surface second spatial derivative). Spherical spline CSD (Perrin):

$$
\text{CSD}(\theta,\phi) = \mathcal{L}_\text{sph} \big[ V(\theta,\phi) \big]
$$

**Pros**: Reduces volume conduction, sharpens local sources.
**Cons**: Not a potential anymore; interpret with care; sensitive at edges; rely on dense arrays and accurate positions.

---

## Epoching, Baseline, and Averaging

### Epoching

* Define windows around events (e.g., $[-200, 800]$ ms).
* Reject or mark **bad epochs** via thresholds (peak‑to‑peak, z‑score on high‑freq RMS), or use **autoreject**/RANSAC.

### Baseline

* **Mean subtraction** over pre‑stim window:

$$
\tilde{x}(t) = x(t) - \frac{1}{T_b}\sum_{t \in \text{baseline}} x(t)
$$

* Prefer **baseline regression** when drifts exist: include a polynomial trend regressor over each epoch and remove fitted trend.

### Averaging (ERP)

* Simple average improves SNR ≈ $\sqrt{N}$ if noise is i.i.d.
* **Robust averaging**: weights $w_i \propto 1/\hat{\sigma}_i^2$ (per‑epoch noise), or trimmed mean to reduce outlier influence.
* Keep **trial counts** per condition; report **effective N** after rejection.

**Time‑frequency / FC**

* Use **multi‑taper** or **wavelets**; report time‑bandwidth product and number of tapers.
* For connectivity, compute **orthogonalized**/imaginary coherence to reduce volume conduction bias.

---

## QC Metrics, Thresholds, and Logs

**Channel‑level**

* Flatline > 5 s → bad.
* High variance RMS z‑score > 5 → inspect.
* Line‑noise ratio (50/60 Hz band / neighboring bands) > 3 → investigate.

**Epoch‑level**

* Peak‑to‑peak > 150–200 µV (adult) → reject (task‑dependent).
* High‑freq RMS (30–100 Hz) z‑score > 3 → suspect EMG.
* Blink detector correlation > 0.7 → flag (for ERP).

**ICA/ASR logs**

* Document: algorithm, version, rank, #ICs removed, labels, % variance removed, ASR cutoff $k$, % samples reconstructed.

**Reproducibility**

* Save raw, cleaned, and **operators** (filter coeffs, projection matrices, ICA unmixing $\mathbf{W}$, montages).
* Export **HTML/markdown QC report** with spectra, topographies, and thresholds.

---

## Suggested Figures

1. **Pipeline flowchart**: raw → channel audit → HPF → line clean → ICA → IC reject → interpolation → AR/REST → task filters → epoch/average.
2. **Filter effects**: step response comparison for HPF 0.1 vs 1 Hz (zero‑phase).
3. **IC maps**: canonical EOG/EMG/ECG scalp maps + spectra.
4. **ASR geometry**: covariance ellipsoid with outlier directions capped.
5. **References as linear transforms**: illustration of AR vs linked mastoids vs REST.
6. **CSD (Laplacian)**: topography sharpening before/after.

(If you want, I can generate these as vector diagrams for your slides.)

---

## Parameter Defaults / Quick Table

| Step                | Default                                    | Rationale / Notes                 |
| ------------------- | ------------------------------------------ | --------------------------------- |
| Initial HPF         | 0.5–1.0 Hz FIR (zero‑phase)                | Stabilize ICA; reduce drifts      |
| Line noise          | Adaptive notch or multi‑taper regression   | Minimize passband ripple          |
| Pre‑ICA data length | ≥ 20×C seconds                             | Sample sufficiency for stable ICA |
| ICA algo            | Extended‑Infomax or AMICA (if time)        | AMICA best separation, slower     |
| IC selection        | ICLabel p(artifact) ≥ 0.7 + manual         | Reduce false positives            |
| Bad channels        | Flatline/variance/line‑ratio rules; RANSAC | Document and interpolate later    |
| Interpolation       | Spherical spline                           | Preserve neighborhood structure   |
| Final reference     | Average or REST                            | For inverse modeling              |
| ERP final band      | 0.1–30/40 Hz                               | Preserve slow components          |
| Epoch reject        | PTP > 150–200 µV; HF RMS z>3               | Task/population dependent         |
| Averaging           | Robust (weighted/trimmed)                  | Guard against outliers            |

---

## References & Resources

**Textbooks**

* Luck, S. J. (2014). *An Introduction to the Event‑Related Potential Technique* (2nd ed.). MIT Press.
* Cohen, M. X. (2014). *Analyzing Neural Time Series Data*. MIT Press.

**Core papers**

* Jung, T.‑P., et al. (2000). Removing EEG artifacts by blind source separation. *Psychophysiology*, 37, 163–178.
* Delorme, A., et al. (2007). Enhanced artifact detection with higher‑order stats and ICA. *NeuroImage*, 34, 1443–1449.
* Mullen, T. R., et al. (2015). Real‑time monitoring using wearable dry EEG (ASR framework). *IEEE TBME*, 62, 2553–2567.
* Bigdely‑Shamlo, N., et al. (2015). The PREP pipeline. *Frontiers in Neuroinformatics*, 9:16.
* Hyvärinen, A., & Oja, E. (2000). Independent Component Analysis: algorithms and applications. *Neural Networks*, 13, 411–430.
* Perrin, F., et al. (1989/1990). Spherical splines for scalp potentials and CSD. *Electroencephalogr. Clin. Neurophysiol.*

**Online tutorials**

* EEGLAB: Artifact rejection & ICA — [https://eeglab.org/tutorials/](https://eeglab.org/tutorials/)
* FieldTrip: Preprocessing — [https://www.fieldtriptoolbox.org/tutorial/preprocessing/](https://www.fieldtriptoolbox.org/tutorial/preprocessing/)
* MNE‑Python: Preprocessing, ICA, SSP, Maxwell filtering (MEG) — [https://mne.tools/stable/auto\_tutorials/](https://mne.tools/stable/auto_tutorials/)

---

### Instructor Notes (optional)

* Assign a **replicable preprocessing lab**: students compare (a) HPF 0.1 vs 1 Hz pre‑ICA; (b) Infomax vs AMICA; (c) AR vs REST. Evaluate ERP amplitude/latency shifts and spectral SNR.
* Provide a **QC rubric** with numeric pass/fail gates and require a **preprocessing manifest** (operators + parameters) per dataset.
