# 🌐 Brain Mapping & Atlas‑Based Localization

> **Goal:** Map EEG source estimates to anatomically/functional meaningful regions, compute parcel‑level signals and networks, and report results that are reproducible across subjects and studies.

---

## 1) Coordinate Spaces & Registrations (the plumbing)

Before atlases: agree on **where** your data live.

* **Native (subject) space:** individual T1 MRI, FreeSurfer surfaces (`lh.pial`, `rh.pial`, `white`, `inflated`), subject‑specific cortical mesh used for source modeling.
* **Template spaces:**

  * **MNI** (e.g., ICBM152 2009c nonlinear): standard volumetric template.
  * **fsaverage / fsLR** (HCP): spherical surface templates for cross‑subject alignment (fsLR 32k/164k).
* **Talairach:** historical; do not mix with MNI without explicit transforms (e.g., ICBM↔Talairach piecewise affines from Lancaster *icbm2tal*).

**EEG best practice:** Do **all inverse modeling and parcellation in subject space**, then aggregate to template/atlas space via surface‑based registration (spherical registration) or volumetric warps (nonlinear). Avoid resampling sensor data directly to template before inversion.

---

## 2) Atlas Taxonomy

### Anatomical (macroanatomy)

* **Desikan–Killiany (DK, 68 cortical regions)**: gyral‑based, robust across subjects, widely used.
* **Destrieux (148 regions)**: finer gyral/sulcal boundaries than DK.
* **AAL / AAL2**: volumetric parcellation (cortex + subcortex), historic and still popular.

### Cytoarchitectonic / Receptor

* **Brodmann areas (BA)**: classical cytoarchitecture; coarse, variable across individuals.
* **Jülich–Brain probabilistic maps (SPM Anatomy toolbox)**: voxelwise probabilities of cytoarchitectonic areas.

### Functional connectivity–derived

* **Yeo‑7 / Yeo‑17 networks**: large‑scale resting‑state networks (networks > parcels).
* **Schaefer (N=100…1000)**: gradient‑informed functional parcels at multiple resolutions; pairs well with Yeo networks.
* **Gordon, Power, Craddock**: alternative functional schemes.

### Multimodal

* **HCP–MMP1.0 (Glasser; 360 areas)**: myelin, task fMRI, RSFC, and architectonics combined; surface‑based, high resolution.

### Subcortical/Cerebellar

* **Harvard–Oxford**, **ATAG** (basal ganglia), **Cerebellar SUIT** (fine cerebellar lobules).
* EEG sensitivity to **deep structures** is limited—include for completeness, interpret cautiously.

---

## 3) Choosing an Atlas for EEG: Practical Criteria

| Criterion                         | Guidance                                                                                                                                                                                                                    |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Spatial scale vs sensor count** | Don’t out‑resolve your data. As a rule of thumb: #parcels ≤ \~½ × #sensors for stable parcel‑FC. With 64–128 EEG channels, **68–200** parcels is reasonable.                                                                |
| **Registration mode**             | Prefer **surface‑based** atlases (DK/Destrieux/HCP/Schaefer) for cortical sources; volumes for subcortex.                                                                                                                   |
| **Interpretability**              | DK/Destrieux for anatomy; Yeo/Schaefer for networks; HCP‑MMP for fine‑grained studies (but verify SNR supports it).                                                                                                         |
| **Reproducibility**               | Use widely adopted atlases with open definitions (FreeSurfer, HCP, Schaefer). Document template versions.                                                                                                                   |
| **Task/context**                  | Motor/visual tasks: sensorimotor/occipital parcels must be cleanly separable (DK often sufficient). Resting‑state FC: Yeo/Schaefer (200–400). Clinical mapping: combine DK/Destrieux + subcortical (Harvard–Oxford) + SUIT. |

---

## 4) Mapping Sources to Parcels (math you can trust)

Let your source time series be $\mathbf{J}\in\mathbb{R}^{N\times T}$ on a cortical surface with $N$ vertices and $T$ time points. Let parcels $r=1,\dots,R$ be defined by vertex sets $\mathcal{V}_r$.

### 4.1 Basic parcel time series (area/sensitivity weighted)

Weights per vertex $i$ in parcel $r$:

$$
w_{ri} = \frac{a_i\,s_i}{\sum_{k\in \mathcal{V}_r} a_k\,s_k}, \quad i\in \mathcal{V}_r
$$

* $a_i$: vertex area (from surface tessellation).
* $s_i$: sensitivity weight (e.g., column norm of **whitened** lead field, or noise‑normalized dSPM/eLORETA sensitivity).

Parcel series:

$$
z_r(t)=\sum_{i\in \mathcal{V}_r} w_{ri}\,J_i(t)
$$

In matrix form $\mathbf{Z} = \mathbf{P}\mathbf{J}$, with $\mathbf{P}\in \mathbb{R}^{R\times N}$ sparse and row‑stochastic.

### 4.2 Robust alternatives

* **PCA/eigenvariate:** $z_r(t)$ = first PC of $ \{J_i(t)\}_{i\in \mathcal{V}_r}$ (sign fixed by mean direction). Good when within‑parcel signals vary in phase.
* **Median**: robust to outliers/leakage spikes.

### 4.3 Leakage‑aware parcellation (recommended)

Compute **resolution/cross‑talk** matrix $\mathbf{R}=\mathbf{K}\mathbf{L}$ (from your inverse $\mathbf{K}$ and forward $\mathbf{L}$). Build **leakage‑compensated** parcel weights by down‑weighting vertices with high cross‑talk into neighboring parcels, or apply **symmetric orthogonalization** at the parcel level (see §7).

---

## 5) Volume‑ vs Surface‑Based Mapping

* **Surface** (preferred for cortex): spherical registration (FreeSurfer/HCP MSMAll) preserves areal topology; no sulcal mixing.
* **Volume**: nonlinear warps (ANTs/FSL) plus resampling to MNI; watch **partial‑volume** effects and blurring across sulci.

**EEG caution:** Don’t “double smooth.” Inverse methods already spread activity (PSF). Avoid heavy spatial smoothing before parcellation; the atlas averaging is itself a smoother.

---

## 6) Functional Connectivity on Atlases (sensor to network)

Given parcel time series $\mathbf{Z}\in\mathbb{R}^{R\times T}$:

### 6.1 Metrics (choose leakage‑robust)

* **Coherence / PLV**: simple but inflated by zero‑lag leakage.
* **Imaginary coherency / wPLI**: robust to zero‑lag mixing.
* **Amplitude‑Envelope Correlation (AEC)** with **pairwise orthogonalization** (Colclough‑style) reduces leakage for envelope‑based FC.
* **MVAR spectral causality (PDC/DTF)**: powerful but needs stationarity and more data.

### 6.2 Workflow

1. Band‑pass (task‑specific).
2. For oscillatory FC: compute complex analytic signals (Hilbert/wavelet).
3. **Leakage correction** (below).
4. FC estimation + nonparametric **surrogates** for significance.
5. Graph construction (thresholded/weighted); compute network metrics (degree, modularity, efficiency), mindful of **density matching** across subjects.

---

## 7) Spatial Leakage & How to Control It

Zero‑lag mixing from the inverse (PSF width) inflates within‑lobe FC.

**Options (pick one, state it clearly):**

* **Imaginary‑part metrics / wPLI** (frequency domain) → discards zero‑phase edges.
* **Orthogonalization (time domain)**: for parcels $x,y$, regress/orthogonalize one against the other before envelope correlation (symmetric orthogonalization to avoid asymmetry).
* **CTF/PSF‑based deblurring**: use $\mathbf{R}=\mathbf{K}\mathbf{L}$ to correct mixing (regularized inverse of $\mathbf{R}$).
* **Source leakage simulations**: include a *null model* (independent sources) to estimate edge inflation and adjust thresholds.

> Report exactly which leakage control you used (and parameters).

---

## 8) Clinical & Research Interpretation

* **Clinical (epilepsy, tumor, stroke):** use **anatomical atlases** (DK/Destrieux + subcortex) for surgical communication; overlay **statistical maps** and **confidence bounds**; quantify distance to lesion/onset zone.
* **Cognitive neuroscience:** Yeo‑17/Schaefer‑200 or HCP‑MMP for network analyses; combine parcel‑level activations with FC changes; align results to **known networks** (DMN/DAN/VAN/SMN/VIS/LIMB/FP).
* **Aging/degeneration:** probabilistic cytoarchitectonics (Jülich) can contextualize affected regions (e.g., BA44/45 language in PPA).

---

## 9) Comparative Utility (expanded)

| Feature          | Talairach    | MNI (ICBM152) | DK         | Destrieux  | AAL        | Yeo‑7/17        | Schaefer (100–400)         | HCP–MMP (360)      | Jülich                            |
| ---------------- | ------------ | ------------- | ---------- | ---------- | ---------- | --------------- | -------------------------- | ------------------ | --------------------------------- |
| Type             | Coord. space | Coord. space  | Anatomical | Anatomical | Volumetric | Networks        | Functional parcels         | Multimodal parcels | Cytoarchitectonic (probabilistic) |
| Scale            | —            | —             | 68         | 148        | 90–116     | 7/17            | 100–1000                   | 360                | variable                          |
| Surface‑based    | —            | —             | ✓          | ✓          | ✗          | ✓               | ✓                          | ✓                  | ✗ (volumetric probs)              |
| EEG suitability  | —            | —             | **High**   | High       | Medium     | High (networks) | **High** (choose ≤200–400) | Medium (needs SNR) | Medium (probabilistic maps)       |
| Interpretability | —            | —             | High       | High       | Medium     | High (networks) | High                       | High               | High (histology)                  |

---

## 10) End‑to‑End Workflow (atlas‑ready ESI)

1. **Inverse in subject space** → $\mathbf{J}(t)$ on subject cortical surface.
2. **Choose atlas** (surface for cortex; add subcortex if needed).
3. **Register atlas to subject** (surface spherical registration or volumetric warp).
4. **Extract parcel time series** with sensitivity & area weights or PCA.
5. **Leakage control** (imag coherency / wPLI / orthogonalization / PSF‑aware).
6. **Compute FC / stats**, correct for multiple comparisons; build graphs with density control.
7. **Group analysis:** map parcel results to template labels (fsaverage/fsLR); summarize by networks (Yeo).
8. **QC**: show parcel coverage maps, PSF widths per region, test–retest reliability.

---

## 11) Suggested Figures

1. **Registration diagram**: subject surface → spherical map → atlas labels → back‑projection to subject.
2. **Parcel extraction**: vertex weights (area×sensitivity) and PCA eigenvariate.
3. **Leakage illustration**: PSF/CTF over neighboring parcels; effect of orthogonalization and wPLI.
4. **Atlas comparison**: DK vs Schaefer‑200 vs HCP‑MMP (same subject).
5. **Network report**: Yeo‑17 networks with parcel‑level FC edges (leakage‑robust metric).

(Ask and I’ll generate clean SVGs for slides.)

---

## 12) Common Pitfalls & Guardrails

* **Mixing spaces** (MNI vs Talairach vs native) without correct transforms → mislabeling. *Guard:* record transforms; verify with fiducials.
* **Over‑parcellation** (e.g., Schaefer‑1000 with 64‑ch EEG) → unstable/aliased FC. *Guard:* match resolution to sensors/SNR.
* **Double smoothing** (spatial filter + parcel averaging) → inflated local FC. *Guard:* minimal pre‑parcellation smoothing.
* **No leakage control** → spurious zero‑lag networks. *Guard:* wPLI/imag coherency/orthogonalization or PSF correction.
* **Ignoring subcortical/cerebellar labels** when clinically relevant. *Guard:* include SUIT/Harvard–Oxford but interpret with EEG sensitivity limits.
* **Unreported versions** (atlas/template). *Guard:* state atlas name, release, template, hemisphere density (fsLR32k vs fsaverage5/6).

---

## 13) References & Resources

**Atlases & Parcellations**

* Desikan, R. S., et al. (2006). An automated labeling system for subdividing the human cerebral cortex. *NeuroImage*, 31, 968–980.
* Destrieux, C., et al. (2010). Automatic parcellation of human cortical gyri and sulci. *NeuroImage*, 53, 1–15.
* Tzourio‑Mazoyer, N., et al. (2002). AAL parcellation. *NeuroImage*, 15, 273–289.
* Yeo, B. T. T., et al. (2011). 7/17‑network parcellations. *J Neurophysiol*, 106, 1125–1165.
* Schaefer, A., et al. (2018). Local‑global parcellations. *Cereb Cortex*, 28, 3095–3114.
* Glasser, M. F., et al. (2016). HCP‑MMP1.0. *Nature*, 536, 171–178.
* Eickhoff, S. B., et al. (2005+). SPM Anatomy toolbox (Jülich). *NeuroImage*, 25, 1325–1335.

**Spaces & Transforms**

* Mazziotta, J., et al. (2001). Probabilistic atlas & reference system. *Phil Trans R Soc B*, 356, 1293–1322.
* Lancaster, J. L., et al. (2007). Bias between MNI and Talairach; icbm2tal transform. *Hum Brain Mapp*, 28, 1194–1205.

**Tooling**

* **FreeSurfer**: DK/Destrieux, spherical registration — [https://surfer.nmr.mgh.harvard.edu](https://surfer.nmr.mgh.harvard.edu)
* **HCP Workbench** (fsLR, MSMAll) — [https://www.humanconnectome.org/software/connectome-workbench](https://www.humanconnectome.org/software/connectome-workbench)
* **MNE‑Python**: `extract_label_time_course`, `label_connectivity` — [https://mne.tools](https://mne.tools)
* **Brainstorm** atlas/label workflows — [https://neuroimage.usc.edu/brainstorm](https://neuroimage.usc.edu/brainstorm)
* **FieldTrip** source/atlas tutorials — [https://www.fieldtriptoolbox.org](https://www.fieldtriptoolbox.org)
* **Anatomy Toolbox** (SPM) — [https://www.fz-juelich.de/en/inm/inm-7/resources/anatomy-toolbox](https://www.fz-juelich.de/en/inm/inm-7/resources/anatomy-toolbox)
* **Brain Connectivity Toolbox** — [https://sites.google.com/site/bctnet/](https://sites.google.com/site/bctnet/)

---

### Instructor Notes (optional)

* **Lab A (parcellation):** DK vs Schaefer‑200 parcel extraction on the same subject; compute parcel SNR, PSF width, and leakage‑robust FC; discuss trade‑offs.
* **Lab B (leakage control):** Compare coherence, imag coherency, wPLI, and symmetric orthogonalization on identical data; use surrogates to estimate false‑positive rates.
* **Lab C (registration QC):** Visualize label overlays on subject surface; compute Dice/overlap of labels after round‑trip to template and back.
