# 📐 Mathematical Modeling of EEG (Forward Problem)

> **Goal:** Construct an accurate **lead field (gain) matrix** $\mathbf{L}$ mapping neural current sources to scalp potentials under the quasi‑static regime, with quantified error and reproducible numerics.

---

## Contents

- [📐 Mathematical Modeling of EEG (Forward Problem)](#-mathematical-modeling-of-eeg-forward-problem)
  - [Contents](#contents)
  - [Source Models](#source-models)
    - [1) Equivalent Current Dipole (ECD)](#1-equivalent-current-dipole-ecd)
    - [2) Distributed Sources](#2-distributed-sources)
  - [Forward Problem: PDE \& Integral Formulations](#forward-problem-pde--integral-formulations)
  - [Numerical Methods](#numerical-methods)
    - [BEM](#bem)
    - [FEM](#fem)
    - [FDM](#fdm)
  - [Head Modeling](#head-modeling)
    - [Compartments \& Conductivities](#compartments--conductivities)
    - [Anisotropy](#anisotropy)
    - [Electrodes \& Reference](#electrodes--reference)
  - [Lead Fields \& Reciprocity](#lead-fields--reciprocity)
  - [Numerical Accuracy \& Validation](#numerical-accuracy--validation)
    - [Mesh/Discretization](#meshdiscretization)
    - [Conditioning \& Solvers](#conditioning--solvers)
    - [Validation Metrics](#validation-metrics)
  - [Uncertainty \& Sensitivity Analysis](#uncertainty--sensitivity-analysis)
    - [Parameter Sensitivity](#parameter-sensitivity)
    - [Methods](#methods)
    - [Calibration](#calibration)
  - [Practical Defaults, Mesh Specs \& Checklists](#practical-defaults-mesh-specs--checklists)
  - [Suggested Figures](#suggested-figures)
  - [References](#references)
    - [Instructor Notes (optional)](#instructor-notes-optional)

---

## Source Models

### 1) Equivalent Current Dipole (ECD)

Represents compact, synchronized neural ensembles by a point‑like **current dipole** with moment $\mathbf{p} \in \mathbb{R}^3$ at position $\mathbf{r}_0$. Units: $[\mathbf{p}] = \text{A·m}$.

* **Parameters:** location $\mathbf{r}_0$, orientation $\hat{\mathbf{n}}$, strength $p=\|\mathbf{p}\|$.
* **Use cases:** focal/evoked responses (e.g., early sensory ERPs), epileptiform spikes.

### 2) Distributed Sources

Tessellate the cortex or volume into many elementary dipoles.

* **Cortical surface model:** dipoles fixed **normal** to the cortical mantle at vertices $\{\mathbf{r}_i\}$ (1 d.o.f. per vertex).
* **Volumetric grid:** dipoles at voxel centers (3 d.o.f. per voxel or constrained orientation).
* **Advantages:** flexible spatial support; matches priors used by MNE/wMNE/LORETA/eLORETA.

---

## Forward Problem: PDE & Integral Formulations

Under the **quasi‑static** approximation (EEG frequencies $\ll$ MHz), displacement currents are negligible:

* Constitutive: $\mathbf{J} = \boldsymbol{\sigma}\,\mathbf{E} + \mathbf{J}^p$, with conductivity tensor $\boldsymbol{\sigma}(\mathbf{r})$.
* Electrostatics: $\mathbf{E} = -\nabla \phi$.
* Charge conservation: $\nabla \cdot \mathbf{J} = 0$.

**Governing PDE** (Poisson in divergence form):

$$
\nabla \cdot \left( \boldsymbol{\sigma} \nabla \phi \right) = \nabla \cdot \mathbf{J}^p \quad \text{in } \Omega
$$

**Boundary conditions:**

* Tissue interfaces: continuity of potential and normal current:
  $\phi_1=\phi_2$, $(\boldsymbol{\sigma}_1 \nabla \phi_1)\cdot\hat{\mathbf{n}} = (\boldsymbol{\sigma}_2 \nabla \phi_2)\cdot\hat{\mathbf{n}}$.
* Scalp–air (no current): $(\boldsymbol{\sigma}\nabla \phi)\cdot\hat{\mathbf{n}}=0$.
* Potentials defined **up to an additive constant** → fix by a **reference** (see below).

**Analytic check (homogeneous infinite medium):**

$$
\phi(\mathbf{r}) = \frac{1}{4\pi\sigma}\, \frac{\mathbf{p}\cdot(\mathbf{r}-\mathbf{r}_0)}{\|\mathbf{r}-\mathbf{r}_0\|^{3}}
$$

(Use this for unit tests of numerics and dipole implementations.)

**Subtraction approach (for FEM/BEM)**
Split $\phi = \phi^\infty + \phi^c$, where $\phi^\infty$ is the analytic solution in a homogeneous medium; solve for **correction** $\phi^c$ in the **heterogeneous** head with homogeneous RHS. Reduces singularity issues from point dipoles.

---

## Numerical Methods

### BEM

**Idea:** Reformulate PDE as boundary integral equations; mesh **interfaces** only (scalp, skull, CSF, brain).

* **Pros:** Accurate for **piecewise homogeneous isotropic** conductivities; fewer dofs; popular for 3‑/4‑shell and realistic BEM (e.g., OpenMEEG/MNE).
* **Cons:** Dense matrices (memory $O(N^2)$); limited handling of **anisotropy** or inhomogeneous interiors; singular kernel integrals require care.

**Flavors:**

* **Single-/double‑layer** formulations (Geselowitz).
* **Collocation** vs **Galerkin** discretization (Galerkin generally more accurate but costlier).
* **H‑matrix / FMM** accelerations for large meshes.

**CSF modeling note:** Including a CSF shell (4‑shell BEM) significantly improves EEG topographies vs 3‑shell (brain‑skull‑scalp).

---

### FEM

**Idea:** Weak form on a volumetric mesh; supports **arbitrary geometry** and **tensor (anisotropic) σ**.

* **Pros:** Handles **anisotropic white matter**, **layered skull**, surgical defects/implants; sparse SPD systems; scalable iterative solvers.
* **Cons:** Higher model‑building cost (segmentation + meshing), more parameters to validate.

**Weak form:** Find $\phi \in \mathcal{V}$ s.t.

$$
\int_{\Omega} (\nabla v)^\top \boldsymbol{\sigma} \nabla \phi \, d\Omega
= \int_{\Omega} v \, \nabla\cdot \mathbf{J}^p \, d\Omega
\;\; \forall v \in \mathcal{V}_0
$$

Point dipoles are incorporated via **St. Venant** source models (spread over small neighborhood) or the **subtraction** method.

**Electrode modeling:** FEM supports the **Complete Electrode Model (CEM)** with finite electrode area $A_e$ and contact impedance $z_e$, adding boundary equations:

$$
\phi|_{e} + z_e \, I_e/A_e = V_e \quad \text{and} \quad \sum_e I_e = 0
$$

Useful for EIT/tES reciprocity and high‑precision EEG; often overkill for standard EEG but valuable in calibration studies.

---

### FDM

**Idea:** Regular grid voxels, finite differences for Laplacian.

* **Pros:** Straightforward, fast, integrates with voxel segmentations.
* **Cons:** Staircase boundary approximation; reduced accuracy for curved skull/CSF interfaces; anisotropy handling more limited (unless tensor‑aware stencils are used).

---

## Head Modeling

### Compartments & Conductivities

Minimum viable compartments for EEG:

* **Brain (GM+WM)**, **CSF**, **Skull**, **Scalp**.
  Three‑shell (brain, skull, scalp) is common historically; **4‑shell with CSF** is recommended for EEG.

**Canonical isotropic conductivities** (approximate, at low frequency):

* **Scalp**: 0.2–0.5 S/m
* **Skull (effective)**: 0.006–0.02 S/m (highly variable; compact vs spongy bone)
* **CSF**: 1.5–2.0 S/m
* **Gray matter**: 0.2–0.4 S/m
* **White matter (iso equiv.)**: 0.06–0.15 S/m

> **Impact ranking (typical):** Skull conductivity & thickness 〉CSF presence 〉electrode positions 〉white‑matter anisotropy 〉scalp conductivity.

### Anisotropy

* **White matter:** conductivity tensor often derived from DTI: $\boldsymbol{\sigma} = k \, \mathbf{D}$ (scaled diffusion tensor), with volume constraint $\det\boldsymbol{\sigma} = \text{const}$. Parallel vs perpendicular conductivity ratios $\sim$3–10.
* **Skull:** layered anisotropy (diploë spongy bone) tangential > radial conductivity; FEM can model this (BEM cannot).

### Electrodes & Reference

* **Point‑electrode** assumption: sample $\phi$ at electrode centers; acceptable for most EEG.
* **Reference:** Potentials are defined up to a constant. Numerically:

  * Fix one node to zero (e.g., average of all electrodes = 0), or
  * Apply a **reference transform** post‑hoc (average reference, REST).
* **Lead field with reference:** $\mathbf{L}$ must be post‑multiplied by the reference operator $\mathbf{R}$ used in analysis to remain consistent.

---

## Lead Fields & Reciprocity

Let there be $M$ electrodes and $N$ source elements (surface vertices or voxels).

* **Lead field (gain) matrix:** $\mathbf{V} = \mathbf{L}\, \mathbf{j} \;+\; \boldsymbol{\varepsilon}$,
  where $\mathbf{V} \in \mathbb{R}^{M}$ (or $M\times T$), $ \mathbf{j} \in \mathbb{R}^{3N}$ (or $N$ if oriented). Units of $\mathbf{L}$: V/(A·m).

* **Column construction (direct):** For each unit dipole at $\mathbf{r}_i$ and orientation $\hat{\mathbf{n}}$, solve PDE and sample at electrodes → $\mathbf{l}_i$.

* **Reciprocity (efficient):** Drive **unit currents** between electrode pairs and compute the electric field $\mathbf{E}$ inside the head; the potential at electrode $m$ due to a dipole $\mathbf{p}$ equals:

  $$
  V_m = -\, \mathbf{p} \cdot \mathbf{E}_m(\mathbf{r}_0)
  $$

  This yields columns/rows of $\mathbf{L}$ without re‑solving per source location (large $N$ efficiency).

> Reciprocity also underpins **tES/EEG co‑simulation** and **conductivity calibration** via known current injection.

---

## Numerical Accuracy & Validation

### Mesh/Discretization

* **BEM:** ensure triangle quality (aspect ratio < 5), smooth normals, adequate density near skull and CSF.
* **FEM:** tetrahedra with graded refinement at thin skull and electrodes; avoid slivers; check element Jacobians.

**Recommended densities (starting points):**

* BEM: ≥ 10k–20k triangles per surface (scalp/skull/CSF/brain) for 64–128 ch; ≥ 30k–50k per surface for 256 ch.
* FEM: ≥ 1–3 million tets for 4‑shell anisotropic models; refine to keep element size < skull thickness/3.

### Conditioning & Solvers

* **FEM:** SPD → Conjugate Gradient with **algebraic multigrid** (AMG) or incomplete Cholesky.
* **BEM:** dense → iterative GMRES with **H‑matrix/FMM** acceleration; precondition with block‑diagonal or Calderón preconditioners.

### Validation Metrics

Compare against **analytic multi‑shell sphere** or a converged high‑res model; evaluate at electrodes for multiple dipole positions & orientations.

* **RDM (Relative Difference Measure):**

  $$
  \text{RDM} = \left\|\frac{\mathbf{v}_\text{test}}{\|\mathbf{v}_\text{test}\|} - \frac{\mathbf{v}_\text{ref}}{\|\mathbf{v}_\text{ref}\|}\right\|
  $$
* **MAG (Magnitude ratio):** $\text{MAG} = \|\mathbf{v}_\text{test}\| / \|\mathbf{v}_\text{ref}\|$.

Target: RDM < 0.1–0.2 and MAG within ±10% for practical models (task‑dependent).

**Sanity checks**

* **Topographic polarity** vs dipole orientation (gyrus vs sulcus).
* **Reference invariance:** $\mathbf{1}^\top \mathbf{V}=0$ for average reference.
* **Reciprocity consistency** within numerical tolerance.

---

## Uncertainty & Sensitivity Analysis

### Parameter Sensitivity

Perturb key parameters and track impact on $\mathbf{L}$ or topographies:

* **Skull conductivity** ±50% can dramatically alter amplitudes and spatial spread.
* **CSF omission** inflates radial source amplitudes and biases localization.
* **Electrode position** noise (RMS 5–10 mm) degrades inverse accuracy and connectivity.

### Methods

* **One‑at‑a‑time sweeps** with RDM/MAG metrics.
* **Monte Carlo** over conductivity priors; report mean ± SD of inverse localization error.
* **Polynomial chaos / surrogate models** for faster UQ in high‑dimensional parameter spaces.

### Calibration

* **tES reciprocity/EIT:** inject small known currents (safely within standards) and fit skull/CSF conductivities to match measured voltages (FEM+CEM).
* **Data‑driven scaling:** tune skull/CSF σ to minimize mismatch vs EC/EO amplitude ratios or auditory/visual ERPs (inferential—use cautiously).

---

## Practical Defaults, Mesh Specs & Checklists

**Modeling choices (robust defaults)**

* Prefer **4‑shell** models with explicit CSF.
* **BEM** (OpenMEEG/MNE) for isotropic σ and speed; **FEM** when anisotropy/skull detail/implants matter.
* **Cortical‑surface sources** with fixed normal orientation for distributed inverses.

**Conductivity starting values**

* GM 0.33 S/m, WM 0.14 S/m (or tensor from DTI), CSF 1.79 S/m, Skull 0.01 S/m, Scalp 0.43 S/m. Document measurement frequency (≈ DC/low‑Hz).

**Mesh checklist**

* BEM: watertight, correctly nested surfaces, no self‑intersections, normals outward.
* FEM: label integrity for each tissue, minimum element quality > 0.2 (e.g., radius‑ratio), element size adapted to thin skull.

**Electrode & reference**

* Use **digitized positions**; validate RMS < 3 mm to MRI scalp.
* Build $\mathbf{L}$ consistent with **final analysis reference** (e.g., average).

**Archival**

* Store: segmentations, meshes, σ values, solver settings, $\mathbf{L}$, and unit tests (sphere comparisons).

---

## Suggested Figures

1. **Forward modeling pipeline**: MRI → segmentation → meshes → method (BEM/FEM) → lead field.
2. **Three‑ vs Four‑shell** topography comparison (with/without CSF).
3. **Reciprocity diagram**: unit current at electrodes ↔ dipole field.
4. **Anisotropy illustration**: WM tensor ellipsoids and effect on field lines.
5. **Validation metrics**: RDM/MAG histograms across test dipoles.

(Ask and I’ll generate slide‑ready SVGs.)

---

## References

**Reviews / Guidelines**

* Hallez, H., et al. (2007). Review on solving the forward problem in EEG source analysis. *J Neuroengineering and Rehabilitation*, 4:46.
* Vorwerk, J., et al. (2014). Guideline for head volume conductor modeling in EEG/MEG. *NeuroImage*, 100, 590–607.
* Grech, R., et al. (2008). Review on solving the inverse problem of EEG. *Clin Neurophysiol*, 119, 2485–2493. (Contextual inverse references.)

**Theory & Methods**

* Geselowitz, D. B. (1967). On bioelectric potentials in an inhomogeneous volume conductor. *Biophys J*, 7, 1–11.
* de Munck, J. C., & Peters, M. J. (1993). Fast multilayered sphere potential computation. *IEEE TBME*, 40, 1166–1174.
* Mosher, J. C., Leahy, R. M., & Lewis, P. S. (1999). EEG/MEG forward solutions. *IEEE TBME*, 46, 245–259.
* Stenroos, M., & Nummenmaa, A. (2016). Incorporating CSF into forward models improves EEG. *NeuroImage*, 129, 265–273.
* Dannhauer, M., Lanfer, B., Wolters, C. H., & Knösche, T. R. (2011). Modeling the human skull in EEG simulations. *NeuroImage*, 54, 590–593.
* Wolters, C. H., et al. (2006). Tissue anisotropy in realistic FEM. *NeuroImage*, 30, 813–826.

**Conductivity Compendia**

* Gabriel, S., Lau, R. W., & Gabriel, C. (1996). Dielectric properties of biological tissues. *Phys Med Biol*, 41, 2251–2269.
* McCann, H., Pisano, G., & Beltrachini, L. (2019). Variation in reported head tissue conductivity. *Brain Topogr*, 32, 825–858.

**Tooling Tutorials**

* Brainstorm: Head modeling — [https://neuroimage.usc.edu/brainstorm/Tutorials/HeadModel](https://neuroimage.usc.edu/brainstorm/Tutorials/HeadModel)
* FieldTrip: Head model (EEG) — [https://www.fieldtriptoolbox.org/tutorial/headmodel\_eeg/](https://www.fieldtriptoolbox.org/tutorial/headmodel_eeg/)
* MNE‑Python: Forward modeling — [https://mne.tools/stable/auto\_tutorials/forward/](https://mne.tools/stable/auto_tutorials/forward/)

---

### Instructor Notes (optional)

* **Lab 1:** Compare 3‑shell vs 4‑shell BEM on the same subject; compute RDM/MAG across 100 random dipoles.
* **Lab 2:** FEM with/without WM anisotropy (from DTI); quantify topographic differences and inverse localization shifts.
* **Lab 3:** Reciprocity calibration—inject known tES currents in a phantom/sim and fit skull σ to minimize model‑data error.
