# 🎯 EEG Source Localization (Inverse Problem)

> **Goal:** Given a forward operator $\mathbf{L}$ (lead field), estimate source activity $\mathbf{j}$ from measured voltages $\mathbf{v}$ while quantifying uncertainty and minimizing bias from head‑model and noise mis‑specification.

---

## 1) Problem Statement & Notation

**Linear model**

$$
\mathbf{v}(t) = \mathbf{L}\,\mathbf{j}(t) + \boldsymbol{\varepsilon}(t), \quad 
\boldsymbol{\varepsilon}(t)\sim\mathcal{N}(\mathbf{0},\,\boldsymbol{\Sigma}_n)
$$

* $\mathbf{v}\in\mathbb{R}^{M}$: sensor voltages (referenced).
* $\mathbf{L}\in\mathbb{R}^{M\times P}$: gain matrix (after the same reference transform used in analysis).
* $\mathbf{j}\in\mathbb{R}^{P}$: source coefficients (e.g., cortical vertices; often $P=3N$ for free orientations or $P=N$ for fixed‑normal).
* $\boldsymbol{\Sigma}_n$: noise covariance (from baseline/rest).

**Ill‑posedness & non‑uniqueness.** $M\ll P$, $\mathbf{L}$ is ill‑conditioned, and different $\mathbf{j}$ produce near‑identical $\mathbf{v}$. Regularization (priors) is mandatory.

**Whitening (best practice).** Estimate $\boldsymbol{\Sigma}_n$ and whiten:

$$
\tilde{\mathbf{v}}=\mathbf{W}_n\mathbf{v},\quad \tilde{\mathbf{L}}=\mathbf{W}_n\mathbf{L}, \quad \mathbf{W}_n^\top \mathbf{W}_n = \boldsymbol{\Sigma}_n^{-1}
$$

All formulas below assume whitening when $\boldsymbol{\Sigma}_n$ is not identity.

---

## 2) Classical Dipole Fitting

### 2.1 Single / Multiple ECD (Equivalent Current Dipoles)

Solve for a small set of dipoles $\{\mathbf{r}_k,\mathbf{p}_k\}$ that best explain $\mathbf{v}$.

**Objective (nonlinear in locations):**

$$
\min_{\{\mathbf{r}_k,\mathbf{p}_k\}} \sum_t \left\|\mathbf{v}(t) - \sum_k \mathbf{L}(\mathbf{r}_k)\,\mathbf{p}_k(t)\right\|_2^2
$$

**Optimization**

* Grid search + local optimization (Levenberg–Marquardt), or global stochastic (GA/PSO) for initialization.
* Model order selection via **BIC/AIC**, **residual variance (RV)**, **F‑tests** across time windows.

**Pros/Cons**

* ✅ High interpretability for focal, phase‑locked sources (epileptiform spikes, early sensory ERPs).
* ❌ Fails for distributed/correlated sources; initialization‑sensitive.

---

## 3) Distributed Linear Inverses (Tikhonov family)

### 3.1 Minimum Norm Estimation (MNE)

Solve a **ridge‑regularized least squares**:

$$
\widehat{\mathbf{j}} = \arg\min_{\mathbf{j}} \|\tilde{\mathbf{v}} - \tilde{\mathbf{L}}\mathbf{j}\|_2^2 + \lambda \|\mathbf{W}_s \mathbf{j}\|_2^2
$$

Closed form:

$$
\widehat{\mathbf{j}} = \mathbf{K}\,\tilde{\mathbf{v}}, \quad
\mathbf{K} = \mathbf{W}_s^{-1}\tilde{\mathbf{L}}^\top \left(\tilde{\mathbf{L}} \mathbf{W}_s^{-1}\tilde{\mathbf{L}}^\top + \lambda \mathbf{I}\right)^{-1}
$$

* $\mathbf{W}_s$ encodes **source prior** (e.g., depth weights or smoothness).
* $\lambda$ is the regularization (SNR) parameter.
* **Bayesian view:** MAP with Gaussian prior $\mathbf{j}\sim\mathcal{N}(\mathbf{0},\,\alpha^{-1}\mathbf{W}_s^{-1})$, noise precision $\beta$ ⇒ $\lambda=\alpha/\beta$.

**Depth bias & fix.** Columns of $\mathbf{L}$ decay with depth; use **depth weighting** $w_i \propto \|\tilde{\mathbf{L}}_{\cdot i}\|^{-\gamma}$ (typ. $\gamma\in[0.5,1]$) so $\mathbf{W}_s=\mathrm{diag}(w_i)$.

**Orientation options**

* **Fixed normal** (1 d.o.f./vertex).
* **Free 3‑D** (3 d.o.f./vertex).
* **Loose orientation**: penalize tangential vs normal with factor $0<\mathrm{loose}\le1$ (e.g., 0.2–0.6).

### 3.2 wMNE (depth‑weighted MNE)

As above, with explicit $\mathbf{W}_s=\mathrm{diag}(w_i)$ to reduce depth bias.

### 3.3 LORETA family (smoothness priors)

Impose **spatial smoothness** using a discrete Laplacian $\mathbf{L}_\mathrm{sp}$ (graph Laplacian of cortical mesh).

* **LORETA:** minimize $\|\tilde{\mathbf{v}}-\tilde{\mathbf{L}}\mathbf{j}\|_2^2 + \lambda \|\mathbf{L}_\mathrm{sp}\mathbf{j}\|_2^2$.

  * Promotes smooth distributions; low spatial resolution but robust to noise.

* **sLORETA:** standardize current estimates by their estimated **noise sensitivity** to reduce localization bias. If $\widehat{\mathbf{j}}=\mathbf{K}\tilde{\mathbf{v}}$, then variance at source $i$ is $\sigma_i^2 = [\mathbf{K}\mathbf{K}^\top]_{ii}$. Report standardized maps $\widehat{\mathbf{j}}_i/\sqrt{\sigma_i^2}$ to equalize depth sensitivity.

* **eLORETA:** chooses weight matrix $\mathbf{W}_s$ such that, in the noise‑free single‑source case, localization is **exact** (zero localization error in theory). Implemented by solving for $\mathbf{W}_s$ that equalizes resolution across locations (iterative reweighting).

**When to prefer which?**

* MNE: simple, fast, good for time‑courses/ERPs.
* wMNE: like MNE but fairer to deep sources.
* LORETA/sLORETA/eLORETA: better spatial uniformity and localization fairness; s/eLORETA are popular for statistical mapping.

### 3.4 Statistical normalizations

* **dSPM:** divide MNE estimates by noise std at each source.
* **sLORETA:** equivalent in spirit; slightly different derivations.
* **z‑scoring vs baseline:** for ERPs and task contrasts.

---

## 4) Adaptive Spatial Filters (Beamformers)

Use data covariance $\mathbf{C}=\mathbb{E}[\tilde{\mathbf{v}}\tilde{\mathbf{v}}^\top]$ to **suppress interfering sources** and maximize passband for a location/orientation.

### 4.1 LCMV (time‑domain)

For leadfield vector $\mathbf{l}_i$ (possibly 3‑D), find weights $\mathbf{w}_i$ that minimize output power with unit‑gain constraint:

$$
\min_{\mathbf{w}} \mathbf{w}^\top \mathbf{C} \mathbf{w}\quad \text{s.t.}\quad \mathbf{w}^\top \mathbf{l}_i=1
$$

Closed form (scalar orientation):

$$
\mathbf{w}_i = \frac{\mathbf{C}^{-1}\mathbf{l}_i}{\mathbf{l}_i^\top \mathbf{C}^{-1}\mathbf{l}_i}
$$

For 3‑D, optimize orientation via generalized eigenvalue problem or scan fixed orientations.

### 4.2 DICS (frequency‑domain)

Replace $\mathbf{C}$ with cross‑spectral density $\mathbf{S}(f)$ to map power/coherence at frequency $f$.

**Pros:** Good at isolating narrow‑band, uncorrelated sources.
**Cons:** **Source correlation** (e.g., bilateral homologs) causes signal cancellation; sensitive to $\mathbf{C}$ estimation (needs many samples, proper regularization).

---

## 5) Sparse & Bayesian Methods

### 5.1 Mixed‑norm / Group‑sparse (MxNE, TF‑MxNE)

Promote focality by $\ell_{21}$ penalties across orientations and/or time:

$$
\min_{\mathbf{J}} \frac{1}{2}\|\tilde{\mathbf{V}}-\tilde{\mathbf{L}}\mathbf{J}\|_F^2 + \lambda \sum_{g} \|\mathbf{J}_g\|_2
$$

* Groups $g$ are vertices (with 3‑D orientation), or time‑frequency atoms (TF‑MxNE).
* **Fast coordinate descent** with active sets; strong for focal, transient activity (e.g., spikes).

### 5.2 Hierarchical Bayes / SBL / ARD (e.g., Champagne, ReML)

Model $ \mathbf{j} \sim \mathcal{N}(\mathbf{0},\,\boldsymbol{\Gamma})$, $\boldsymbol{\Gamma}=\mathrm{diag}(\gamma_g)$ (group variances). Maximize evidence w\.r.t. hyperparameters $\{\gamma_g\}$:

* Iteratively update $\gamma_g$ via closed‑form rules; many $\gamma_g\to0$ ⇒ **automatic sparsity**.
* Variants: **Bayesian MNE**, **ReML** priors, **SBL** with temporal smoothness.

**Pros:** Data‑driven regularization; uncertainty quantification (posterior covariance).
**Cons:** Heavier compute; sensitive to initializations for correlated sources.

---

## 6) Hyperparameter & Model Selection

* **$\lambda$ (MNE/LORETA) selection:**

  * **L‑curve**, **GCV**, **Morozov discrepancy** (match residual to noise level), **cross‑validation** over trials/time.
  * **Bayesian evidence maximization** (Type‑II ML) jointly for $\lambda$ and depth/variance weights.

* **Beamformer regularization:** shrinkage for $\mathbf{C}$:
  $\hat{\mathbf{C}} = (1-\rho)\mathbf{C} + \rho \,\frac{\mathrm{tr}(\mathbf{C})}{M}\mathbf{I}$, pick $\rho$ via Ledoit‑Wolf or CV.

* **Dipole number:** BIC/AIC over fits across windows; penalize complexity; require **temporal consistency** of solutions.

---

## 7) Resolution Analysis (How to quantify “blurring”?)

Given an inverse operator $\mathbf{K}$ with $\widehat{\mathbf{j}}=\mathbf{K}\mathbf{v}$:

* **Resolution matrix:** $\mathbf{R}=\mathbf{K}\mathbf{L}$.

  * **Point Spread Function (PSF):** column $i$ of $\mathbf{R}$ = estimated map when true source is unit at $i$.
  * **Cross‑Talk Function (CTF):** row $i$ of $\mathbf{R}$ = contribution of other true sources to estimated $i$.

**Metrics**

* **Spatial dispersion** of PSF (e.g., geodesic FWHM).
* **Localization error**: distance between true location and PSF peak.
* **Leakage index**: off‑diagonal mass of row/column.

Use these to compare MNE vs sLORETA vs eLORETA vs beamformers on your head model.

---

## 8) Practical Implementation Details

### 8.1 Orientation & Constraints

* **Fixed‑normal** on cortical surface: reduces parameters and aligns with pyramidal physiology.
* **Loose/free** only when justified; penalize tangential components.

### 8.2 Depth weighting & scaling

* Depth exponent $\gamma$ \~ 0.5–1.0; tune via CV/evidence.
* Scale $\mathbf{L}$ columns to unit norm before regularization to reduce bias (then undo for physical units).

### 8.3 Referencing & rank

* Ensure the **same reference** applied to $\mathbf{L}$ and $\mathbf{v}$.
* Average reference removes one rank; handle in whitening (project out DC).

### 8.4 Noise covariance estimation

* Use **baseline** (ERP) or **rest** segments; robust estimators (Ledoit–Wolf).
* Remove bad channels/segments before $\boldsymbol{\Sigma}_n$; include reference transform.

### 8.5 Temporal modeling

* For continuous time‑courses, add temporal smoothness ($\|\mathbf{D}_t\mathbf{J}\|^2$) or **Kalman/State‑space** solvers (dSPM‑KF, SSM‑MNE) for better dynamics and SNR.

---

## 9) Validation & Benchmarking

### 9.1 Simulation studies

* Use your **forward model**; place synthetic sources (vary depth/orientation), add realistic noise (from baseline).
* Report: localization error (mm), PSF dispersion, time‑course correlation, robustness to SNR, and **conductivity** perturbations.

### 9.2 Phantom/physical models

* Saline tanks or head phantoms with known dipoles; validate amplitude scaling and topographies (RDM/MAG) and inverse localization.

### 9.3 Clinical/coregistration

* Compare with **iEEG/ECoG** onset zones or stimulation sites; compute distances and ROC for region‑level detection.

**Ablations (what matters most?)**

* Subject‑specific 4‑shell + digitized electrodes vs template;
* Depth weighting choices;
* Whitening accuracy;
* s/eLORETA vs MNE vs beamformer on your task.

---

## 10) Common Failure Modes & Mitigations

* **Depth bias** (superficial hotspots) → depth weighting, s/eLORETA, REST reference, accurate CSF/skull.
* **Source correlation** (beamformer nulls) → use MNE family, or apply vector beamformers with regularization and/or source leakage correction.
* **Leakage in connectivity** (zero‑lag inflation) → operate in **source space** with orthogonalization or imaginary/wPLI metrics.
* **Head‑model mismatch** (skull σ, electrode errors) → sensitivity analysis; calibrate; report uncertainty.
* **Over‑filtering/high‑pass** (temporal distortion) → verify ERP integrity; use baseline regressions.
* **Hyperparameter over‑fitting** → CV/evidence; lock parameters before group stats.

---

## 11) Reporting Checklist (put in your Methods)

* Forward model: BEM/FEM/FDM, shells, conductivities (values & frequency).
* Electrode positions: digitization method & RMS error to MRI scalp.
* Reference & whitening: reference type, $\boldsymbol{\Sigma}_n$ estimation, whitening status.
* Inverse: algorithm (MNE/wMNE/s/eLORETA/LCMV/MxNE/ARD), orientation constraint, depth weight $\gamma$, $\lambda$ selection method.
* Validation: PSF/CTF metrics, localization error on simulations or phantoms, SNR levels.
* Statistical mapping: baseline windows, multiple‑comparison control.

---

## 12) Suggested Figures

1. **Taxonomy map** of inverse methods (dipoles → linear → adaptive → sparse/Bayes) with typical use‑cases.
2. **Depth bias demo**: MNE vs wMNE vs sLORETA/eLORETA for a deep source.
3. **Beamformer schematic**: covariance ellipsoid and unit‑gain constraint.
4. **Resolution matrix** heatmaps with PSF/CTF illustrations.
5. **Hyperparameter curves**: L‑curve and CV error vs $\lambda$.
6. **Effect of whitening**: topography normalization and dSPM maps.

(If useful, I can generate these as vector‑quality SVGs.)

---

## 13) References (core set)

* Hämäläinen, M. S., & Ilmoniemi, R. J. (1994). Interpreting magnetic fields of the brain: minimum norm estimates. *Med Biol Eng Comput*, 32, 35–42.
* Dale, A. M., Liu, A. K., et al. (2000). dSPM. *Neuron*, 26, 55–67.
* Pascual‑Marqui, R. D., Michel, C. M., & Lehmann, D. (1994). LORETA. *Int J Psychophysiol*, 18, 49–65.
* Pascual‑Marqui, R. D. (2002). sLORETA technical details. *Methods Find Exp Clin Pharmacol*, 24(Suppl D), 5–12.
* Pascual‑Marqui, R. D. (2007). Exact (eLORETA). arXiv:0710.3341.
* Van Veen, B. D., et al. (1997). LCMV beamforming. *IEEE Trans Biomed Eng*, 44, 867–880.
* Gross, J., et al. (2001). DICS beamforming. *J Neurosci Methods*, 108, 121–134.
* Gramfort, A., Kowalski, M., & Hämäläinen, M. (2012). Mixed‑norm estimates. *NeuroImage*, 70, 410–421.
* Wipf, D., & Nagarajan, S. (2009). SBL for neuromagnetic inverse. *NeuroImage*, 44, 947–966.
* Acar, Z. A., & Makeig, S. (2013). Forward model errors impact. *Brain Topogr*, 26, 378–396.
* Grech, R., et al. (2008). Review on inverse problem. *Clin Neurophysiol*, 119, 2485–2493.
* Mosher, J. C., Leahy, R. M., & Lewis, P. S. (1999). Forward solutions for inverse methods. *IEEE TBME*, 46, 245–259.

**Tutorials & tools**

* Brainstorm / FieldTrip / MNE‑Python inverse tutorials (same links as Part 5).

---

### Instructor options

* **Lab:** Compare MNE, sLORETA, eLORETA, LCMV, and MxNE on identical simulated deep vs superficial sources; compute PSF dispersion and localization error distributions.
* **Lab:** Evaluate hyperparameter selection (L‑curve vs CV vs evidence) and show over‑/under‑regularization effects on time‑courses and spatial leakage.
* **Lab (advanced):** Implement ARD/SBL with evidence updates; compare sparsity patterns and robustness to correlated sources.
