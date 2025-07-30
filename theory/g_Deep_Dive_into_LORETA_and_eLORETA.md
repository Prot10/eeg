# 🔎 Deep Dive into LORETA, sLORETA, and eLORETA

> **Goal:** Build, validate, and interpret LORETA‑family estimators with correct noise handling, depth/orientation constraints, and resolution analysis — so you can trust the spatial maps and report them defensibly.

---

## 1) Notation & Setup

We work with the standard linear forward model (after **the same reference** used during analysis and after **whitening** by noise covariance $\boldsymbol{\Sigma}_n$):

$$
\tilde{\mathbf{v}}(t) = \tilde{\mathbf{L}}\mathbf{j}(t) + \tilde{\boldsymbol{\varepsilon}}(t),\quad 
\tilde{\boldsymbol{\varepsilon}}\sim\mathcal{N}(\mathbf{0},\mathbf{I})
$$

* $\tilde{\mathbf{L}}=\mathbf{W}_n\mathbf{L}$, $\tilde{\mathbf{v}}=\mathbf{W}_n\mathbf{v}$, with $\mathbf{W}_n^\top\mathbf{W}_n=\boldsymbol{\Sigma}_n^{-1}$.
* Sources may be **fixed‑normal** (1 d.o.f./vertex) or **3‑D/free** (3 d.o.f./vertex, often with “loose” penalties).

We estimate a **linear inverse operator** $\mathbf{K}$ (units A·m/V) with $\widehat{\mathbf{j}}=\mathbf{K}\tilde{\mathbf{v}}$.

---

## 2) LORETA (Smoothness‑Regularized MNE)

### 2.1 Objective & Solution

LORETA is Tikhonov (ridge) with a **spatial smoothness** operator (discrete Laplace–Beltrami on the cortical mesh):

$$
\widehat{\mathbf{j}}
= \arg\min_{\mathbf{j}}
\|\tilde{\mathbf{v}}-\tilde{\mathbf{L}}\mathbf{j}\|_2^2
+ \lambda\,\|\mathbf{L}_\text{sp}\mathbf{j}\|_2^2
$$

Closed form (MAP with Gaussian prior):

$$
\mathbf{K}_\text{LORETA}
= (\mathbf{L}_\text{sp}^\top \mathbf{L}_\text{sp})^{-1}\tilde{\mathbf{L}}^\top
\Big(\tilde{\mathbf{L}}(\mathbf{L}_\text{sp}^\top \mathbf{L}_\text{sp})^{-1}\tilde{\mathbf{L}}^\top + \lambda\mathbf{I}\Big)^{-1}
$$

**Practical construction**

* Build $\mathbf{L}_\text{sp}$ from the mesh graph Laplacian (cotangent weights preferred).
* Normalize rows/cols to avoid scale pathologies; ensure the null space corresponds only to global constants (handle with boundary conditions or small ridge).

### 2.2 Properties & Tuning

* Promotes **spatially smooth** $\mathbf{j}$ → **broad PSFs**, robust to noise.
* **$\lambda$** controls resolution vs stability (L‑curve, GCV, CV).
* Works well for widespread or low‑SNR effects; localizes **centroids** more than focal peaks.

---

## 3) sLORETA (Variance‑Standardized LORETA/MNE)

### 3.1 Rationale

Linear inverse estimates have **location‑dependent variance** due to field spread and depth. sLORETA corrects by **standardizing** each source by its estimated (noise‑only) standard deviation.

Let $\mathbf{K}$ be any linear inverse (e.g., MNE or LORETA). The estimator noise covariance is:

$$
\boldsymbol{\Sigma}_{\hat{\mathbf{j}}} = \mathbf{K}\mathbf{K}^\top \quad (\text{since we whitened})
$$

Then **standardized current density** is:

$$
\widehat{j}_i^{\,\text{s}} = \frac{\widehat{j}_i}{\sqrt{[\boldsymbol{\Sigma}_{\hat{\mathbf{j}}}]_{ii}}}
$$

This is conceptually similar to **dSPM**; differences arise from the choice of $\mathbf{K}$ and normalization details.

### 3.2 Property

Under idealized conditions (white noise, correct $\tilde{\mathbf{L}}$, single source), the **localization bias is reduced** (peak occurs at the true generator more often), though spatial extent remains broad.

---

## 4) eLORETA (Exact LORETA)

### 4.1 Idea

Choose **diagonal weights** so that the **resolution matrix** becomes **identity for single‑source cases**, yielding **zero localization error** in theory (noise‑free, correctly modeled head).

Let $\mathbf{T}=\mathrm{diag}(t_i)\succ0$ be source weights. Define:

$$
\mathbf{K}_\text{eL} = \mathbf{T}\tilde{\mathbf{L}}^\top\big(\tilde{\mathbf{L}}\mathbf{T}\tilde{\mathbf{L}}^\top + \lambda \mathbf{I}\big)^{+}
$$

(If data are unwhitened, replace $\mathbf{I}$ by $\boldsymbol{\Sigma}_n$. The $+$ denotes (pseudo)inverse for rank‑deficient cases, e.g., average reference.)

The **resolution matrix** is $\mathbf{R}=\mathbf{K}_\text{eL}\tilde{\mathbf{L}}$. eLORETA chooses $\mathbf{T}$ so that each **column PSF** peaks at its own index (zero localization error).

### 4.2 Fixed‑Point Update (practical)

A robust implementation uses an iterative reweighting:

1. Initialize $t_i^{(0)} = 1 / \|\tilde{\mathbf{L}}_{\cdot i}\|_2^2$ (or all ones).

2. At iteration $k$, compute

   $$
   \mathbf{G}^{(k)} = \tilde{\mathbf{L}}\,\mathbf{T}^{(k)} \tilde{\mathbf{L}}^\top + \lambda \mathbf{I}
   $$

   and the **diagonal sensitivity** (leverage) for each $i$:

   $$
   h_i^{(k)} = \tilde{\mathbf{l}}_i^\top \big(\mathbf{G}^{(k)}\big)^{+}\tilde{\mathbf{l}}_i,
   \quad \tilde{\mathbf{l}}_i=\tilde{\mathbf{L}}_{\cdot i}
   $$

3. Update weights to equalize sensitivity:

   $$
   t_i^{(k+1)} = \frac{1}{h_i^{(k)} + \epsilon}
   $$

   (small $\epsilon$ for numerical stability; optionally rescale $\mathbf{T}$ by a positive scalar — it cancels out.)

4. Repeat until $\max_i |t_i^{(k+1)}-t_i^{(k)}|/t_i^{(k)}<\tau$ (e.g., $10^{-3}$).

Under typical conditions, this converges quickly and yields PSFs with peaks at the correct vertices (the defining eLORETA property). After convergence, compute $\mathbf{K}_\text{eL}$ and estimates $\widehat{\mathbf{j}}=\mathbf{K}_\text{eL}\tilde{\mathbf{v}}$.

> **Note:** If the data are properly **whitened** and a realistic head model is used, $\lambda$ can be small; with limited trials, keep a modest $\lambda$ (e.g., 3–10% of the largest eigenvalue of $\tilde{\mathbf{L}}\mathbf{T}\tilde{\mathbf{L}}^\top$) to guard against numerical issues.

### 4.3 Relationship to sLORETA/dSPM

* **sLORETA/dSPM**: **post‑hoc standardization** of any linear inverse by noise sensitivity.
* **eLORETA**: **pre‑hoc reweighting** of the inverse operator so that **localization is theoretically exact** (for single point sources, noise‑free).

---

## 5) Depth, Orientation & Constraints

* **Orientation**: Prefer **fixed‑normal** on the cortical surface unless a scientific reason demands free/loose 3‑D. For 3‑D, apply **loose** (e.g., 0.2–0.6) to penalize tangential > normal.
* **Depth weighting**: LORETA/sLORETA may still benefit from depth weighting of $\tilde{\mathbf{L}}$ columns; eLORETA’s $\mathbf{T}$ compensates much of depth bias inherently.
* **Rank/referencing**: Ensure that $\tilde{\mathbf{L}}$ and $\tilde{\mathbf{v}}$ share the same **average‑reference** (or other) transform; project out the constant vector to avoid singularities.

---

## 6) Hyperparameter Selection ($\lambda$) & Noise

* Estimate **noise covariance** from baseline/rest; **whiten** before inversion.
* Select $\lambda$ via **L‑curve**, **GCV**, or **cross‑validation** across trials/time windows.
* In sLORETA/dSPM maps, because you standardize by variance, $\lambda$ mainly affects **smoothness/extent**, less the **localization peak**; in eLORETA, modest $\lambda$ doesn’t destroy the exactness much but helps numerical stability.

---

## 7) Resolution Analysis & What “Zero Localization Error” Means

Given $\mathbf{K}$, the **resolution matrix** $\mathbf{R}=\mathbf{K}\tilde{\mathbf{L}}$ maps true sources to their estimates.

* **PSF (Point Spread Function)** for index $i$: column $ \mathbf{r}_{\cdot i}$. Peak location = **localization**; width = **spatial dispersion** (geodesic FWHM on cortex).
* **CTF (Cross‑Talk Function)** for index $i$: row $\mathbf{r}_{i\cdot}$. Large off‑diagonal mass = leakage from others into $i$.

**Expectations**

* **LORETA**: broad PSFs, modest CTF leakage.
* **sLORETA**: peaks closer to true location; PSF still broad.
* **eLORETA**: **peak at true location (theoretical)**; PSF width narrower than LORETA/sLORETA, but still non‑zero; CTF reduced but not eliminated (especially with multiple correlated sources and noise).

> Zero localization error ≠ delta function PSF. It means the **maximum** of PSF aligns with the true source under the stated assumptions.

---

## 8) Workflows: Before Applying eLORETA

1. **Preprocessing**: bad‑channel repair, robust average ref/REST, high‑pass (≤ 0.5–1 Hz for ICA), line‑noise control, ICA/ICLabel to remove EOG/EMG/ECG, interpolate, final reference.
2. **Electrodes**: 3‑D digitization (fiducials + head points), RMS to MRI scalp < 3 mm.
3. **Head model**: Subject‑specific 4‑shell (with CSF); BEM for isotropic, FEM for anisotropy or implants; document conductivities.
4. **Whitening**: baseline segments → $\boldsymbol{\Sigma}_n$ → $\mathbf{W}_n$.
5. **Orientation**: fixed‑normal cortical model unless motivated otherwise.
6. **eLORETA weights**: fixed‑point updates (above), pick $\lambda$, compute $\mathbf{K}_\text{eL}$.
7. **QC**: Check $\mathbf{R}$ PSFs at random vertices; verify peaks align; report PSF width and CTF leakage.
8. **Stats**: sLORETA‑like standardization or baseline z‑scoring for group maps; multiple‑comparison control (cluster/TFCE).

---

## 9) Comparative Analysis Cheat‑Sheet

| Criterion                 | LORETA                              | sLORETA                         | eLORETA                                                    |
| ------------------------- | ----------------------------------- | ------------------------------- | ---------------------------------------------------------- |
| Regularizer               | Smoothness ($\mathbf{L}_\text{sp}$) | Any (then variance standardize) | LORETA/MNE‑like with **data‑driven weights** $\mathbf{T}$  |
| Localization bias         | Moderate                            | Low                             | **Zero (ideal, single source, noise‑free)**                |
| Spatial resolution        | Low (broad PSF)                     | Moderate (broad but centered)   | Higher (narrower PSF)                                      |
| Robustness to noise       | High                                | Moderate‑High                   | Moderate (benefits from whitening/$\lambda$)               |
| Compute                   | Low‑moderate                        | Low‑moderate + variance map     | **Higher** (iterative weight updates)                      |
| Sensitivity to head model | Moderate                            | Moderate                        | **Higher** (weights reflect $\tilde{\mathbf{L}}$ fidelity) |

---

## 10) Failure Modes & Mitigations

* **Head‑model mismatch (skull σ, CSF)** → mislocalized peaks even in eLORETA. *Mitigate:* 4‑shell models, sensitivity analysis, possible conductivity calibration.
* **Electrode mis‑registration** → peak shifts; ensure digitization accuracy and visual QC.
* **Noise mis‑specification / no whitening** → inflated false positives in standardized maps; always estimate $\boldsymbol{\Sigma}_n$.
* **Multiple correlated sources** → beamformers struggle; LORETA family returns blended maps; interpret with PSF/CTF and task priors.
* **Orientation mis‑modeling** → free‑3D without loose constraints yields spurious tangential activity; prefer fixed‑normal or loose ≤ 0.6.

---

## 11) Mini‑Exercises & Derivations

1. **Posterior variance & sLORETA**: Show that for whitened data the estimator noise covariance is $\mathbf{K}\mathbf{K}^\top$. Derive sLORETA as per‑vertex z‑scores.
2. **PSF peak condition**: For eLORETA, prove that with the fixed‑point choice of $\mathbf{T}$, the PSF column achieves its maximum at the true index (use Cauchy–Schwarz in the metric induced by $\mathbf{G}^{+}$).
3. **L‑curve**: Implement an L‑curve for LORETA; identify the corner and show PSF dispersion vs residual norm trade‑off.
4. **Depth bias demo**: Simulate a deep vs superficial source; compare MNE, wMNE, sLORETA, eLORETA localization error and PSF width.

---

## 12) Suggested Figures (slide‑ready)

1. **Taxonomy**: LORETA → sLORETA (post‑hoc standardize) → eLORETA (pre‑hoc reweight).
2. **PSF/CTF comparison**: same head model, focal source; show top‑row PSFs and bottom‑row CTFs for LORETA, sLORETA, eLORETA.
3. **eLORETA fixed‑point**: flow diagram of $t_i$ updates via leverage $h_i$.
4. **Effect of whitening**: standardized maps with/without whitening.
5. **Sensitivity to skull σ**: peak shift vs σ perturbation (+/−50%).

(Ask and I’ll generate SVGs.)

---

## 13) References & Resources

**Core**

* Pascual‑Marqui, R. D., Michel, C. M., & Lehmann, D. (1994). LORETA. *Int J Psychophysiol*, 18, 49–65.
* Pascual‑Marqui, R. D. (2002). sLORETA technical details. *Methods Find Exp Clin Pharmacol*, 24(Suppl D), 5–12.
* Pascual‑Marqui, R. D. (2007). Exact (eLORETA). arXiv:0710.3341.

**Related / Context**

* Dale, A. M., Liu, A. K., et al. (2000). dSPM. *Neuron*, 26, 55–67.
* Hauk, O. (2004). Keep a lid on inverse methods. *Clin Neurophysiol*, 115, 795–803.
* Acar, Z. A., & Makeig, S. (2013). Forward‑model errors and localization. *Brain Topogr*, 26, 378–396.

**Software tutorials**

* LORETA‑KEY: [https://www.uzh.ch/keyinst/loreta](https://www.uzh.ch/keyinst/loreta)
* Brainstorm (Source Estimation): [https://neuroimage.usc.edu/brainstorm/Tutorials/SourceEstimation](https://neuroimage.usc.edu/brainstorm/Tutorials/SourceEstimation)
* FieldTrip (Source Analysis): [https://www.fieldtriptoolbox.org/tutorial/sourceanalysis/](https://www.fieldtriptoolbox.org/tutorial/sourceanalysis/)
* MNE‑Python (Inverse): [https://mne.tools/stable/auto\_tutorials/inverse/](https://mne.tools/stable/auto_tutorials/inverse/)

---

### Instructor Notes (optional)

* Provide a **resolution‑analysis notebook** that computes PSF/CTF for LORETA, sLORETA, eLORETA on your class head model; have students quantify localization error and FWHM.
* Include a **robustness lab**: perturb skull conductivity, electrode positions, and noise covariance; measure peak shift and PSF broadening.
* For assessment, ask students to **justify $\lambda$** and **orientation constraints** in a methods paragraph given a specific task and SNR.
