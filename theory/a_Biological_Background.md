# 🧠 Biological Background

> **Goal of this module**: Build a rigorous, mechanistic understanding of how EEG signals arise from neural tissue and propagate to scalp electrodes, with the mathematical tools and assumptions that underpin forward and inverse modeling.

---

## Table of Contents

- [🧠 Biological Background](#-biological-background)
  - [Table of Contents](#table-of-contents)
  - [Neurophysiology of EEG Signals](#neurophysiology-of-eeg-signals)
    - [Neural Basis of EEG Generation](#neural-basis-of-eeg-generation)
    - [Signal Propagation \& Tissue Conductivity](#signal-propagation--tissue-conductivity)
    - [Synaptic Integration \& Summation](#synaptic-integration--summation)
    - [Cortical Layer Contributions](#cortical-layer-contributions)
    - [Synchronization \& Network Oscillations](#synchronization--network-oscillations)
  - [Mathematical Foundations of Volume Conduction](#mathematical-foundations-of-volume-conduction)
    - [Quasi-Static Approximation](#quasi-static-approximation)
    - [Governing Equations](#governing-equations)
    - [Dipole Field in a Homogeneous Medium](#dipole-field-in-a-homogeneous-medium)
    - [Open vs. Closed Fields](#open-vs-closed-fields)
    - [Anisotropy \& Inhomogeneity](#anisotropy--inhomogeneity)
    - [Boundary Conditions](#boundary-conditions)
  - [From Microcircuit to Macro-EEG](#from-microcircuit-to-macro-eeg)
    - [Current Dipole Moment \& Coherence](#current-dipole-moment--coherence)
    - [Columnar Geometry \& Folding](#columnar-geometry--folding)
    - [Cancellation \& Summation](#cancellation--summation)
    - [Scale-Free Activity \& 1/f Structure](#scale-free-activity--1f-structure)
  - [Comparing EEG to Other Modalities](#comparing-eeg-to-other-modalities)
  - [Common Pitfalls \& Myths](#common-pitfalls--myths)
  - [Practical Sidebars, Boxes \& Figures](#practical-sidebars-boxes--figures)
    - [Box A: Validity of the Quasi-Static Approximation](#box-a-validity-of-the-quasi-static-approximation)
    - [Box B: Reciprocity Principle](#box-b-reciprocity-principle)
    - [Box C: Conductivity Values \& Variability](#box-c-conductivity-values--variability)
    - [Box D: Why Action Potentials Rarely Dominate Scalp EEG](#box-d-why-action-potentials-rarely-dominate-scalp-eeg)
    - [Suggested Figures (Diagrams)](#suggested-figures-diagrams)
  - [Mini-Exercises \& Discussion Prompts](#mini-exercises--discussion-prompts)
  - [Glossary](#glossary)
  - [References \& Further Reading](#references--further-reading)
    - [Instructor Notes (optional)](#instructor-notes-optional)

---

## Neurophysiology of EEG Signals

### Neural Basis of EEG Generation

EEG primarily reflects the synchronous postsynaptic activity of cortical **pyramidal neurons**, not isolated action potentials. Pyramidal cells are radially oriented (apical dendrites toward pia, basal toward white matter), forming **open fields** that align transmembrane currents into mesoscopic **current dipoles**. Synaptic activation (EPSPs/IPSPs) drives ionic flux across membrane patches, yielding intracellular return currents and extracellular volume currents. Because PSPs last longer (10–200 ms) than spikes (1–2 ms), they **sum** more effectively across space and time to produce detectable scalp potentials.

**Key contributors**

* **Excitatory synapses (AMPA/NMDA)** on apical dendrites: typically produce current sinks in distal dendrites with source in perisomatic region.
* **Inhibitory synapses (GABA\_A/GABA\_B)** often perisomatic or dendritic: shape timing and amplitude of population PSPs.
* **Glial & neuromodulatory influences**: regulate extracellular ion concentrations and membrane conductances, modulating oscillatory regimes.

> **Takeaway**: EEG ≈ synchronized PSP-driven dipoles from aligned pyramidal populations; spikes contribute minimally at the scalp under normal conditions.

### Signal Propagation & Tissue Conductivity

Neuronal transmembrane currents launch electric fields that propagate through a **volume conductor** comprising cortex, white matter, CSF, skull, and scalp. The **skull** (low conductivity) attenuates and spatially smooths signals; the **CSF** (high conductivity) shunts currents tangentially.

**Typical qualitative ordering of conductivities (σ)**:
CSF > scalp ≈ gray matter > white matter (anisotropic) >> skull.

Consequences:

* **Amplitude attenuation**: skull reduces potential magnitudes by \~10× relative to cortex (order-of-magnitude intuition).
* **Spatial blurring**: high-σ CSF and low-σ skull spread and smear sources, reducing spatial resolution.
* **Orientation sensitivity**: radial vs. tangential sources differ in visibility due to folding and boundary geometry.

### Synaptic Integration & Summation

* **Temporal summation**: PSPs overlapping in time add linearly to first order, increasing net dipole moment.
* **Spatial summation**: co-oriented neurons within \~mm–cm patches sum; coherence decays with distance and heterogeneity.
* **Net effect**: N weak generators can dominate if **phase-locked** (∝ N for coherent amplitudes vs. ∝ √N for incoherent).

### Cortical Layer Contributions

* **Layers II/III & V**: dense pyramidal populations create prominent open-field generators.
* **Thalamocortical inputs**: layer IV and distal apical tufts entrain alpha and sleep rhythms.
* **Inhibitory interneurons**: sculpt spectral content (e.g., gamma via PING/ING motifs).

### Synchronization & Network Oscillations

Canonical sources:

* **Alpha (8–13 Hz)**: thalamocortical loops; posterior dominant rhythm in relaxed wakefulness.
* **Theta (4–8 Hz)**: hippocampal-prefrontal systems, memory and control demands.
* **Beta (13–30 Hz)**: sensorimotor networks; post-movement beta rebound.
* **Gamma (>30 Hz)**: local circuits, often short-range synchrony with limited scalp visibility.

Mechanisms include inhibitory-excitatory feedback (PING), resonance, and long-range coupling via thalamus and cortico-cortical pathways.

---

## Mathematical Foundations of Volume Conduction

### Quasi-Static Approximation

At EEG frequencies (< \~1 kHz), displacement currents are negligible compared to conduction currents in brain tissue. Hence electromagnetic induction is minimal, and electric fields are **irrotational** (∇×E ≈ 0), allowing scalar potential φ.

**Validity heuristic**: $\omega\,\varepsilon / \sigma \ll 1$ for tissue (ω angular frequency, ε permittivity, σ conductivity).

### Governing Equations

Let **J** be current density, **Jᵖ** impressed (primary) current from neural membranes, **σ** conductivity tensor, **E** electric field, **φ** scalar potential.

* Constitutive law: $\mathbf{J} = \sigma\,\mathbf{E} + \mathbf{J}^{p}$
* Electrostatics: $\mathbf{E} = -\nabla \phi$
* Charge conservation (steady state): $\nabla\cdot \mathbf{J} = 0$

Combining yields the **Poisson equation in divergence form**:

$\nabla\cdot(\sigma\,\nabla\phi) = \nabla\cdot\mathbf{J}^{p} \quad (1)$

In homogeneous, isotropic regions (σ constant):

$\sigma\,\nabla^{2}\phi = \nabla\cdot\mathbf{J}^{p} \quad (2)$

### Dipole Field in a Homogeneous Medium

Model a compact neural ensemble as a **current dipole** with moment $\mathbf{p} = \int_{V} \mathbf{J}^{p}(\mathbf{r})\, dV$. In an infinite homogeneous medium with conductivity σ, the potential at $\mathbf{r}$ from a dipole at $\mathbf{r}_{0}$ is

$\phi(\mathbf{r}) = \frac{1}{4\pi\sigma} \frac{\mathbf{p}\cdot(\mathbf{r}-\mathbf{r}_{0})}{\lVert \mathbf{r}-\mathbf{r}_{0} \rVert^{3}} \quad (3)$

**Superposition** applies for multiple dipoles and layered media (with appropriate boundary conditions).

### Open vs. Closed Fields

* **Open fields** (aligned pyramidal dendrites) → strong net dipole, measurable at scalp.
* **Closed fields** (symmetric or misaligned sources) → local cancellation, weak scalp expression but visible in LFP/ECoG.

### Anisotropy & Inhomogeneity

White matter exhibits **anisotropic conductivity** (σ tensor): currents prefer fiber directions. Skull is **layered** (compact vs. spongy bone) with low effective σ. Equation (1) must be solved with spatially varying, tensor-valued σ.

### Boundary Conditions

At an interface between media 1 and 2:

* Potential continuity: $\phi_{1} = \phi_{2}$
* Normal current continuity: $(\sigma_{1}\nabla\phi_{1})\cdot \hat{n} = (\sigma_{2}\nabla\phi_{2})\cdot \hat{n}$

At the outer scalp–air boundary (air ≈ insulator): $(\sigma\nabla\phi)\cdot \hat{n} = 0$.

> These conditions drive the use of **BEM/FEM/FDM** solvers in realistic head models.

---

## From Microcircuit to Macro-EEG

### Current Dipole Moment & Coherence

For N micro-dipoles $\{\mathbf{p}_{i}\}$ within a patch:

$\mathbf{p}_{\text{macro}} = \sum_{i=1}^{N} \mathbf{p}_{i} \quad (4)$

Coherence matters: if phases are aligned, $\lVert \mathbf{p}_{\text{macro}} \rVert \propto N$; if random, $\propto \sqrt{N}$.

### Columnar Geometry & Folding

Gyri vs. sulci flip dipole **orientation** relative to scalp. Tangential sources (sulcal walls) contribute differently than radial sources (gyral crowns). Folding can cause **polarity inversions** across electrodes and **partial cancellations** between adjacent patches.

### Cancellation & Summation

Neighboring patches with opposite orientation reduce the net field (vector sum). CSF sheets promote lateral current spread, increasing **spatial smoothing**.

### Scale-Free Activity & 1/f Structure

EEG power spectra often follow $P(f) \propto 1/f^{\alpha}$ with oscillatory peaks riding on a scale-free background ($\alpha \approx 1\!\text{–}\!2$, state-dependent). Separating **aperiodic** vs. **periodic** components is crucial for mechanistic interpretation.

---

## Comparing EEG to Other Modalities

| Modality | Measured Signal                           | Temporal Resolution | Spatial Resolution (cortex)                | Cost/Portability | Notes                                                           |
| -------- | ----------------------------------------- | ------------------- | ------------------------------------------ | ---------------- | --------------------------------------------------------------- |
| **EEG**  | Scalp potentials from conduction currents | \~1 ms              | \~10–30 mm (state/model-dependent)         | Low/High         | Sensitive to radial & tangential sources; affected by skull/CSF |
| **MEG**  | Magnetic fields from neural currents      | \~1 ms              | \~5–15 mm (better for tangential currents) | High/Low         | Less sensitive to skull; room-temp OPMs emerging                |
| **fMRI** | BOLD (hemodynamic)                        | \~1–2 s (TR)        | \~1–3 mm voxels                            | High/Low         | Indirect neural measure; neurovascular coupling                 |
| **PET**  | Radiotracer metabolism                    | minutes             | \~4–6 mm                                   | Very High/Low    | Molecular specificity                                           |

> **Complementarity**: EEG’s millisecond precision is unmatched; spatial resolution improves with high-density arrays and realistic head models.

---

## Common Pitfalls & Myths

1. **“EEG is just noise.”**
   False. It reflects structured population activity; careful preprocessing and modeling recover interpretable physiology.
2. **“Spikes dominate EEG.”**
   At the scalp, PSPs dominate; spikes contribute locally to LFP/ECoG.
3. **“More electrodes always fix localization.”**
   Gains saturate without accurate **head models**, **electrode positions**, and **conductivity priors**.
4. **“Alpha = idling.”**
   Alpha entails active inhibition and gating in sensory systems, not mere idleness.
5. **“Gamma on scalp equals local cortical gamma.”**
   High-frequency scalp activity mixes muscle and broadband components; interpret cautiously.

---

## Practical Sidebars, Boxes & Figures

### Box A: Validity of the Quasi-Static Approximation

The criterion $\omega\varepsilon/\sigma \ll 1$ holds across EEG bands for brain tissue (σ \~ 0.1–0.6 S/m; ε\_r \~ 10^5–10^7 at low frequencies). Hence equations (1–3) are appropriate. At kHz and in air gaps/electronics, displacement currents can matter.

### Box B: Reciprocity Principle

Driving a known current between two electrodes on the scalp and computing potentials at a putative source location yields the same **lead field** as placing a dipole at that location and measuring at the electrodes. Reciprocity enables efficient lead-field computation and electrode sensitivity analysis.

### Box C: Conductivity Values & Variability

* **Gray matter**: \~0.2–0.4 S/m (isotropic)
* **White matter**: \~0.06–0.15 S/m (anisotropic; parallel > perpendicular)
* **CSF**: \~1.5–2.0 S/m
* **Skull**: \~0.006–0.02 S/m (compact vs. spongy layers)
* **Scalp**: \~0.2–0.5 S/m

> Values vary with frequency, temperature, age, and measurement method. Model uncertainty in σ can dominate localization error.

### Box D: Why Action Potentials Rarely Dominate Scalp EEG

Spikes are brief and spatially localized, with mixed orientations that **cancel** at macroscales. PSPs, by contrast, are slower, widespread, and coherent, producing large-scale dipoles that survive skull filtering.

### Suggested Figures (Diagrams)

> These can be rendered as clean vector schematics for slides/handouts.

1. **Cortical Column as a Dipole**
   *Apical dendrites (sink), soma/basal (source), return currents; field lines and equipotential surfaces.*

```
       pia
       ↑
   [sink •••]
      │││││    ← apical dendrites
      │││││
   ─────────  cortex surface
      │││││
   [source ○○○]  soma/basal
       ↓
     white matter
```

2. **Layered Head Model**
   *Concentric shells with σ labels; arrows showing attenuation through skull and shunting in CSF.*

```
 [ Brain (σ≈0.3) ] — [ CSF (σ≈1.8) ] — [ Skull (σ≈0.01) ] — [ Scalp (σ≈0.4) ] — Air
         ↓ strong field         → lateral spread           ↓ attenuation          → recording
```

3. **Open vs. Closed Fields**
   *Two nearby patches with opposing orientation demonstrating cancellation at the scalp.*

4. **Lead Field Map Example**
   \*Topographic sensitivity of a single electrode to a unit dipole (conceptual heatmap).

---

## Mini-Exercises & Discussion Prompts

1. **Derive Equation (1)** from $\mathbf{J}=\sigma\mathbf{E}+\mathbf{J}^p$, $\mathbf{E}=-\nabla\phi$, and $\nabla\cdot\mathbf{J}=0$. State necessary assumptions.
2. **Orientation Thought Experiment**: A radial dipole on a gyrus vs. a tangential dipole in a sulcus—predict scalp topographies qualitatively. How does CSF thickness alter them?
3. **Conductivity Sensitivity**: Increase skull σ by 2× in a three-shell model. Predict changes in amplitude and spatial spread.
4. **Coherence & Scaling**: If 10,000 micro-dipoles synchronize with 0.8 phase-locking value, estimate scaling of $\lVert\mathbf{p}_{\text{macro}}\rVert$ vs. uncorrelated case.
5. **Quasi-Static Limits**: Identify scenarios in EEG/BCI hardware where displacement currents or inductive effects might become non-negligible.

---

## Glossary

* **Current dipole ($\mathbf{p}$)**: Vector representing strength and orientation of a localized current source.
* **Lead field**: Sensitivity pattern mapping sources to electrode potentials; columns of the **gain matrix**.
* **Volume conductor**: Passive, conductive medium (brain, CSF, skull, scalp) through which currents flow.
* **Open field**: Geometry/alignment that yields nonzero net dipole at macroscales.
* **Quasi-static**: Regime where displacement currents are negligible; electrostatics with conductive media.

---

## References & Further Reading

**Foundational Texts**

* Nunez, P. L., & Srinivasan, R. (2006). *Electric Fields of the Brain* (2nd ed.). OUP.
* Buzsáki, G. (2006). *Rhythms of the Brain*. OUP.
* Plonsey, R., & Heppner, D. B. (1967). Considerations of quasi-stationarity in electrophysiological systems. *Bull. Math. Biophys*.

**Reviews & Tutorials**

* Lopes da Silva, F. (2010). EEG: Origin and measurement. *Clinical Neurophysiology*.
* Buzsáki, G., Anastassiou, C. A., & Koch, C. (2012). The origin of extracellular fields and currents—EEG, ECoG, LFP, and spikes. *Nat. Rev. Neurosci*.
* Jackson, A. F., & Bolger, D. J. (2014). Neurophysiological bases of EEG and signal processing: a tutorial. *Brain Research Reviews*.
* Baillet, S., Mosher, J. C., & Leahy, R. M. (2001). Electromagnetic brain mapping. *IEEE Signal Processing Magazine*.

**Mechanistic/Modeling**

* Hämäläinen, M., Hari, R., Ilmoniemi, R., Knuutila, J., & Lounasmaa, O. V. (1993). MEG—theory, instrumentation, and applications. *Rev Mod Phys*.
* Murakami, S., & Okada, Y. (2006). Contributions of principal neocortical neurons to MEG/EEG. *J Neurophysiol*.
* Nunez, P. L. (1981). A study of traveling waves in the human cortex. *Electroencephalogr Clin Neurophysiol*.
* de Munck, J. C., & Peters, M. J. (1993). A fast method to compute potential fields of dipoles in a multilayered sphere. *IEEE TBME*.
* Dannhauer, M., Lanfer, B., Wolters, C. H., & Knösche, T. R. (2011). Modeling of the human skull. *NeuroImage*.
* Wagner, T. et al. (2004). Noninvasive human brain stimulation and head modeling. *Clinical Neurophysiology*.

**Conductivity & Biophysics**

* Gabriel, S., Lau, R. W., & Gabriel, C. (1996). The dielectric properties of biological tissues. *Phys Med Biol*.
* Geddes, L. A., & Baker, L. E. (1967). The specific resistance of biological material. *Med Biol Eng*.
* McCann, H., Pisano, G., & Beltrachini, L. (2019). Variation in reported human head tissue electrical conductivity. *Brain Topography*.

**Methodological Resources**

* EEGLAB tutorials: [https://eeglab.org/tutorials](https://eeglab.org/tutorials)
* MNE-Python tutorials (forward/inverse modeling): [https://mne.tools/stable/auto\_tutorials/index.html](https://mne.tools/stable/auto_tutorials/index.html)

---

### Instructor Notes (optional)

* Pair this module with a whiteboard derivation of Eq. (1–3), and a live demo in MNE-Python computing a simple three-shell forward model.
* For assessment, include a short quiz on assumptions, and a problem set exploring conductivity perturbations and orientation effects.
