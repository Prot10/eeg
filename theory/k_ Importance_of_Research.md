# ⏱️ Faster & More Accurate EEG Source Localization — What It Takes

> **Objective:** Ship source imaging that is *fast enough for clinical and BCI loops* (≤50–100 ms end‑to‑end) and *accurate enough to be trusted*, with quantified uncertainty and robust behavior under noise, artifacts, and head‑model mismatch.

---

## 1) End‑to‑End Latency Budget (target ≤50–100 ms)

Break the loop into timed stages. Below are **practical budgets** for a 64–128‑ch system streaming at 1 kHz with 250 ms windows and 50 ms hop:

| Stage     | Operation                                                                 | Budget  |
| --------- | ------------------------------------------------------------------------- | ------- |
| Buffering | acquire 50 ms hop; causal windowing                                       | 5–15 ms |
| Preproc   | causal HPF/LPF, line‑noise regression, EOG/ECG regression, bad‑ch masking | 5–15 ms |
| Inverse   | apply precomputed operator ($\mathbf{K}\mathbf{v}$) or beamformer power   | 2–15 ms |
| Features  | parcel projection, envelopes/PSD, leakage‑robust FC (if needed)           | 5–20 ms |
| Decision  | thresholding/decoder + UI feedback                                        | 5–20 ms |

> Tight loops use **precomputation**, **vectorized BLAS**, and **fixed operators** per session. Avoid re‑solving big systems online.

---

## 2) Accuracy Levers vs Speed Levers (what moves the needle)

| Lever                                        |  Accuracy ↑ |       Latency ↑ | Notes                                                             |
| -------------------------------------------- | ----------: | --------------: | ----------------------------------------------------------------- |
| 4‑shell head model (CSF explicit)            |    **High** |    Offline only | Improves topography fidelity; do it offline                       |
| Electrode digitization (RMS <3 mm)           |    **High** |             Low | Big win; reduces inverse bias                                     |
| Whitening with good noise covariance         |    **High** |             Low | Stabilizes inverses & standardizations                            |
| Depth weighting / eLORETA weights            | Medium–High |      Low–Medium | Precompute; online cost is negligible                             |
| Beamformer with well‑estimated covariance    | Medium–High | **Medium–High** | Adaptive; regularize $\mathbf{C}$; consider slower update cadence |
| Dense parcellation (>400)                    |  Low–Medium |          Medium | Don’t out‑resolve sensors/SNR                                     |
| Causal artifact removal (EOG/EMG regression) |      Medium |             Low | Keep regressors short‑lag; no heavy ICA online                    |
| Physics‑informed DL refiner                  | Medium–High |          Medium | Quantize + distill for inference speed                            |

---

## 3) Algorithmic Blueprints (with complexity)

### 3.1 Linear Distributed Inverse (MNE/wMNE/dSPM/sLORETA/eLORETA)

**Offline**

* Build lead field $\mathbf{L}\in\mathbb{R}^{M\times P}$ (e.g., $M=128$, $P=10^4$ cortical vertices).
* Estimate noise covariance $\boldsymbol{\Sigma}_n$; form whitening $\mathbf{W}_n$.
* Choose source prior $\mathbf{W}_s$ (depth, smoothness).
* Precompute inverse operator:

$$
\mathbf{K} = \mathbf{W}_s^{-1}\tilde{\mathbf{L}}^\top\left(\tilde{\mathbf{L}}\mathbf{W}_s^{-1}\tilde{\mathbf{L}}^\top+\lambda\mathbf{I}\right)^{-1},\quad \tilde{\mathbf{L}}=\mathbf{W}_n\mathbf{L}
$$

Using **Woodbury**, this is an $M\times M$ system—cheap offline.

**Online**

* Per hop: $\widehat{\mathbf{j}}=\mathbf{K}\tilde{\mathbf{v}}$; → $O(PM)$ MACs (matrix–vector).
* With $P=10^4, M=128$, that’s \~1.3 M MACs → **well under 10 ms on CPU** with BLAS; sub‑ms on GPU.

**eLORETA**

* Offline: fixed‑point for diagonal weights $\mathbf{T}$; recompute $\mathbf{K}$.
* Online: same cost as MNE.

**Tip:** If memory is tight, compress $\mathbf{K}\approx \mathbf{B}\mathbf{C}$ with rank‑$r$ SVD ($r\!\ll\!M$): cost $O(r(M{+}P))$.

---

### 3.2 Beamformers (LCMV/DICS)

**Offline / slow‑update**

* Compute/regularize covariance $\mathbf{C}$ (or CSD $ \mathbf{S}(f)$).
* For each location/orientation $i$: $\mathbf{w}_i = \frac{\mathbf{C}^{-1}\mathbf{l}_i}{\mathbf{l}_i^\top \mathbf{C}^{-1}\mathbf{l}_i}$.

**Online**

* Source power: $p_i=\mathbf{w}_i^\top \mathbf{C}_\text{cur}\mathbf{w}_i$ or $(\mathbf{w}_i^\top \mathbf{v})^2$.
* Updating $\mathbf{C}^{-1}$ **every hop** is costly; use **EWMA covariance** with **Sherman–Morrison** rank‑1 updates, or update every $K$ hops (e.g., 200–500 ms).

**Trade‑off:** Better at isolating narrowband/un‑correlated sources; sensitive to source correlation and covariance mis‑estimation.

---

### 3.3 Hybrid Physics‑informed DL

* **Init** with $\widehat{\mathbf{j}}_0 = \mathbf{K}\tilde{\mathbf{v}}$.
* **Refine** via a shallow CNN/UNet/GNN constrained by forward‑consistency loss:
  $\|\tilde{\mathbf{L}}\widehat{\mathbf{j}}_\theta - \tilde{\mathbf{v}}\|^2$.
* **Deploy** with **INT8/FP16 quantization** and **knowledge distillation** → $<10$ ms inference on mid‑range GPUs.

---

## 4) Preprocessing for Real‑Time (keep it causal, keep it light)

* **Filters:** causal IIR (Butterworth) or short FIR; document group delay.
* **Line‑noise:** narrow adaptive notch or sinusoid regression; avoid broad notches.
* **Artifact control:** short‑lag EOG/ECG regression (and HEOG/VEOG channels), motion sensor regressors; **no ICA online**—if needed, pre‑learn IC projections offline and apply as fixed SSP.
* **Bad channels:** rolling RMS/line‑ratio detectors; replace with spherical‑spline interpolation.

---

## 5) Head Model Accuracy Without Blowing the Budget

* Build **subject‑specific 4‑shell** BEM/FEM **offline**.
* Sensitivity analysis offline: ±50% skull conductivity; store **two** $\mathbf{K}$ operators (low/high σ) and **pick** at runtime based on EC/EO alpha amplitude scaling (cheap empirical calibration).
* **Electrode jitter compensation:** if cap shifts, apply a **small rigid correction** to digitized positions and re‑compute the **electrode interpolation** (not the whole $\mathbf{K}$).

---

## 6) Automation & Ops (make it push‑button)

* **Preset pipelines** per indication: *Epilepsy spikes*, *Resting stroke*, *Neurofeedback motor*.
* **Autotune** $\lambda$ using **GCV** on a short baseline; cache result.
* **Self‑check**: run sphere‑model unit tests (RDM/MAG) and EC/EO alpha reactivity check before session → abort if out of bounds.
* **QC telemetry** every minute: line‑ratio, channel RMS, PSF width estimate at a sentinel vertex, buffer latency.

---

## 7) Cloud vs Edge

| Where      | What                                                                                                                    | Why                               |
| ---------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **Edge**   | Preproc, inverse apply, simple features, UI                                                                             | Determinism, low latency, privacy |
| **Cloud**  | Heavy analytics (FEM meshing, DL training), longitudinal stats, backups                                                 | Scale & centralization            |
| **Hybrid** | Ship precomputed $\mathbf{K}$, atlas, and light model to edge; keep raw data local; periodically upload derived metrics | Balanced footprint                |

**Security:** if PHI/MRI leaves the site, enforce encryption + BAA; prefer **derived features** for telemetry.

---

## 8) KPIs & Acceptance Tests

* **Latency**: p95 end‑to‑end ≤100 ms; jitter ≤20 ms.
* **Localization** (simulation): median error ≤10–15 mm; PSF FWHM ≤40–60 mm for cortical patches (depends on montage).
* **Stability**: inverse output variance under ±10% electrode jitter ≤X%; under ±50% skull σ change ≤Y% (report).
* **Clinical anchors**: epilepsy—distance to iEEG onset; stroke—r, AUC for classifying affected hemisphere via BSI/DAR.

**Unit tests**

* Analytic dipole in sphere: RDM<0.1, MAG within ±10%.
* Reciprocity check (electrode drive vs dipole).
* Consistency: $\mathbf{1}^\top \mathbf{v}=0$ after average reference; same for $\mathbf{K}\mathbf{L}$.

---

## 9) Concrete Speedups (that actually work)

1. **Precompute everything**: $\mathbf{K}$, parcel projector $\mathbf{P}$, and their product $\mathbf{P}\mathbf{K}$ → parcel time series in one GEMV.
2. **Low‑rank compression** of $\mathbf{K}$ (SVD), and **block‑sparse** layouts (hemisphere blocks) for cache locality.
3. **Pinned memory & batch GEMV** to amortize kernel launch overheads on GPU.
4. **Fixed‑point eLORETA** offline; no iterative ops online.
5. **Sherman–Morrison** covariance updates for beamformers; **slow cadence** re‑weighting (e.g., every 0.5–1 s).
6. **Quantize** DL refiners; **distill** from a big teacher trained offline with physics losses.
7. **Avoid Python loops** in the hot path; keep it BLAS‑level (NumPy MKL/OpenBLAS) or fused kernels.

---

## 10) Common Pitfalls (and how to dodge them)

* **Re‑estimating $\lambda$ every hop** → jitter. *Fix:* set once per session (or per block) via GCV.
* **Double smoothing** (filtering + heavy spatial smoothing + parcellation) → leakage inflation. *Fix:* minimal smoothing; rely on atlas averaging.
* **Online ICA** → latency spikes. *Fix:* use regressors/SSP.
* **Beamformer over‑regularization** → spatial bias. *Fix:* shrinkage with objective selection (e.g., Ledoit–Wolf) and slower update cadence.
* **Ignoring reference consistency** → wrong maps. *Fix:* apply exactly the same reference to $\mathbf{L}$ and data.

---

## 11) Example Real‑Time Recipes

### A) Fast Distributed Inverse (BCI/neurofeedback)

* Offline: 4‑shell BEM; eLORETA $\mathbf{K}$; parcel projector $\mathbf{P}$; compute $\mathbf{Q}=\mathbf{P}\mathbf{K}$.
* Online: $\mathbf{z} = \mathbf{Q}\tilde{\mathbf{v}}$ → Hilbert envelopes → symmetric orthogonalization (optional) → control signal (e.g., occipital alpha).
* Latency: **<30–60 ms** on CPU.

### B) Narrowband Beamforming (oscillatory tasks)

* Offline: $\mathbf{C}^{-1}$ and $\mathbf{w}_i$ at target band; update $\mathbf{C}$ every 0.5 s.
* Online: band‑pass → $(\mathbf{w}_i^\top \mathbf{v})^2$ streams for selected ROIs.
* Latency: **<50–80 ms** with 3–10 ROIs.

### C) Hybrid DL Refiner

* Offline: train UNet/GNN with forward‑consistency; quantize to INT8.
* Online: $\widehat{\mathbf{j}}_0=\mathbf{K}\tilde{\mathbf{v}}$ → DL refine (≤10 ms) → parcels.
* Latency: **<50–70 ms** on modest GPU / fast CPU.

---

## 12) Translational Impact (what faster + more accurate buys you)

* **Acute neurology**: earlier severity stratification; targeted monitoring (e.g., spreading depolarizations proxies) when imaging is unavailable.
* **Epilepsy**: tighter sEEG plans; potential time savings to OR scheduling; better post‑hoc concordance analysis.
* **Rehab & BCI**: smoother control signals → better learning curves and adherence; ability to adapt **within‑session**.
* **Precision psychiatry**: responsive, session‑wise network metrics to guide neuromodulation titration.

---

## 13) References & Further Reading (focused)

* He, B., et al. (2018). Electrophysiological brain imaging: state‑of‑the‑art. *IEEE Reviews in BME*, 11, 142–152.
* Michel, C. M., & Brunet, D. (2019). EEG source imaging: practical steps. *Front Neurol*, 10, 325.
* Luu, P., & Tucker, D. M. (2021). Toward real‑time EEG source localization. *Front Neurosci*, 15, 642702.
* (Tooling) Brainstorm Online Processing; MNE real‑time; OpenBCI docs.

---

### Instructor Options

* **Hands‑on benchmark lab:** students implement Recipe A (eLORETA) and measure p95 latency; compare MKL vs OpenBLAS vs GPU.
* **Ablation:** remove whitening; increase skull σ by ±50%; quantify impact on localization error and PSF width.
* **Ops drill:** simulate a failing session (line‑noise burst, 5 bad channels, cap shift) and recover within latency budget.
