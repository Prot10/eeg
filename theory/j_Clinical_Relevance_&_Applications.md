# 🩺 Clinical Relevance & Applications of EEG Source Localization

> **Goal:** Use source imaging (ESI) to inform diagnosis, prognosis, targeting, and monitoring—while quantifying uncertainty and aligning with clinical endpoints.

---

## 1) Clinical Use‑Cases & Decision Pathways

### 1.1 Ischemic/Hemorrhagic Stroke

**Questions**

* Where is functional disruption? How asymmetric is activity? What predicts recovery?

**Quantitative markers (source‑space)**

* **Relative band power maps:** ↑delta/θ, ↓alpha over peri‑lesional cortex.
* **DAR / DTABR:** $\mathrm{DAR} = P_\delta / P_\alpha$, $\mathrm{DTABR} = (P_\delta+P_\theta)/(P_\alpha+P_\beta)$.
* **Brain Symmetry Index (BSI)** (hemispheric asymmetry) — compute on parcel pairs in source space:

  $$
  \mathrm{BSI}(f) = \frac{\sum_{r\in \text{LH}}|S_r(f)-S_{\pi(r)}(f)|}{\sum_{r\in \text{LH}}(S_r(f)+S_{\pi(r)}(f))}
  $$

  where $\pi(r)$ maps LH parcel to its RH counterpart.

**Workflow**

1. Resting EEG (EC/EO), robust preprocessing.
2. Distributed inverse (sLORETA/eLORETA or wMNE) on subject‑specific 4‑shell models.
3. Parcel time series (Desikan/Destrieux), compute DAR/DTABR & BSI; compare to normative database (z‑scores).
4. Prognosis models: regress NIHSS/modified Rankin at 3 months vs early source‑space metrics; report CIs.

**Impact**

* Early **severity/prognosis**, guiding intensity of rehab; mapping **functional diaschisis** beyond structural lesion.

---

### 1.2 Epilepsy (Presurgical Evaluation)

**Questions**

* Where is the irritative zone (interictal spikes)? Does it concord with MRI/PET/SPECT/iEEG? Can ESI guide electrodes and resection?

**Signals & methods**

* **Interictal spikes/fast ripples**: time‑locked averaging; beamformers for narrowband oscillations; distributed inverses for spikes.
* **Dipole fitting** for early spike peak (focal cases); **MxNE/sparse Bayesian** for multifocal patterns.
* Source localization error typically reported as distance (mm) to iEEG onset or resection margin.

**Workflow**

1. High‑density EEG (128–256), spike detection → average spikes near 50% upstroke.
2. Build 4‑shell BEM/FEM; digitized electrodes; average reference or REST.
3. Inverse (eLORETA/wMNE for maps; ECD for focal peak; MxNE for sparse multiple foci).
4. Concordance analysis with MRI lesion, PET hypometabolism, SISCOM/ESI overlays.
5. **Decision aid**: recommend stereo‑EEG (sEEG) contacts targeting ESI maxima; compute distances post‑hoc.

**Impact**

* Higher **localization concordance** → improved selection of sEEG targets; potential **Engel I** seizure‑freedom predictors when resection overlaps ESI maxima.

---

### 1.3 Neurodegeneration (Alzheimer’s & related disorders)

**Questions**

* Are there early functional changes? Can we track progression/therapy?

**Markers**

* **Alpha slowing** (reduced IAF), ↑theta/delta in posterior networks, default‑mode connectivity disruptions.
* Source‑space **network metrics** (e.g., wPLI within DMN; posterior alpha power z‑scores vs age‑matched norms).

**Workflow**

* Eyes‑closed rest → eLORETA/wMNE → Schaefer‑200 parcellation → wPLI (8–12 Hz) and aperiodic‑adjusted alpha power → z‑scored summary.

**Impact**

* **Early detection/monitoring** adjunct; not standalone diagnostic—combine with cognitive tests and imaging.

---

### 1.4 Psychiatric Disorders (Depression, Anxiety)

**Questions**

* Biomarkers for stratification and monitoring? Neuromodulation targets?

**Markers (caveats)**

* **Frontal alpha asymmetry** (controversial as a diagnostic marker; treat as *monitoring* candidate).
* Task‑ or symptom‑relevant **network changes** (e.g., fronto‑limbic coupling).

**Workflow**

* Task or rest; leakage‑robust FC (wPLI/orthogonalized AEC).
* For TMS targeting, identify **hypo/hyper‑connected parcels** (e.g., left DLPFC network coupling) and co‑register to neuronavigation space.

**Impact**

* **Treatment monitoring** (e.g., rTMS response trajectories) more defensible than diagnosis.

---

### 1.5 ADHD

**Notes**

* Theta/beta ratio (TBR) is **non‑specific**; prefer task‑relevant source maps (e.g., midline theta during cognitive control) and **connectivity** within attention networks; use as **supportive** evidence only.

---

### 1.6 Schizophrenia & Autism Spectrum Disorders

**Signals**

* **Gamma deficits** (E/I imbalance hypotheses), **MMN/ERN** ERP source alterations, long‑range dysconnectivity.

**Workflow**

* Task‑locked ERPs (MMN/oddball) → source time‑courses in auditory/ACC; resting gamma with muscular controls (CSD + stringent artifact rejection).
* Interpret gamma cautiously at scalp; prefer task‑locked narrowband patterns.

---

## 2) Real‑Time Monitoring & Neurofeedback/BCI

**Latency budget (target ≤100 ms end‑to‑end; ideal ≤50 ms)**

* Acquisition (buffering): 20–40 ms (e.g., 64–128 Hz hop on 256–512 Hz stream).
* Preprocessing: 5–15 ms (causal filters, artifact regressors).
* Inverse: 2–10 ms (precomputed $\mathbf{K}$×vector; or fast beamformer).
* Decoding/feedback rendering: 5–20 ms.

**Design**

* Fix inverse operator ($\mathbf{K}$) for the session; **precompute** parcel projection; operate on **envelopes** for robustness; apply **leakage‑robust** features (orthogonalized envelopes / wPLI on sliding windows).
* For neurofeedback: choose **one parcel** (or network score) and a **clear shaping rule** (e.g., increase alpha ERS in occipital parcels).

**Use‑cases**

* Stroke rehab (motor imagery → ipsilesional SM parcels).
* Anxiety modulation (posterior alpha up‑training).
* Epilepsy closed‑loop warning (risk indices from source‑space features; caution: clinical‑grade devices require regulatory clearance).

---

## 3) Which Inverse Method When?

| Indication                  | Signal          | Recommended inverse                 | Rationale                                                |
| --------------------------- | --------------- | ----------------------------------- | -------------------------------------------------------- |
| Interictal spikes (focal)   | Transient spike | **ECD** ± eLORETA                   | Peak localization; simple interpretability               |
| Multifocal/uncertain spikes | Transient       | **eLORETA / MxNE**                  | Balanced localization + focality                         |
| Resting stroke mapping      | Band power      | **eLORETA / wMNE**                  | Stable distributed maps for hemispheric indices          |
| Task ERPs                   | Evoked          | **MNE/dSPM/eLORETA**                | Good time‑course fidelity                                |
| Narrowband oscillations     | Ongoing         | **LCMV/DICS**                       | Adaptive suppression of interferers (beware correlation) |
| FC/connectivity             | Ongoing/ERP     | **eLORETA/MNE + leakage‑robust FC** | Stable source signals first, then robust FC              |

---

## 4) Reporting & Interpretation (Clinical‑grade)

**Minimum report items**

* Acquisition (density, montage), preprocessing (filters, artifact removal), **head model** (shells, conductivities), electrode digitization method, **reference**, noise covariance (whitening).
* Inverse method + hyperparameters (depth/loose/orientation, $\lambda$).
* Atlas & space; leakage control in FC.
* **Uncertainty**: PSF width at peak parcel; distance to confidence isosurface; for epilepsy, **distance to resection/onset zone**.
* **Concordance** with MRI/PET/SPECT/iEEG; discrepancies explained.

**Visualization**

* Source maxima on the individual MRI; parcel summaries with z‑scores; FC matrices with density‑matched graphs.

---

## 5) Validation Against Clinical Endpoints

* **Epilepsy**: distance (mm) between ESI maxima and iEEG onset / resection; association with Engel I outcomes.
* **Stroke**: source‑space DAR/DTABR/BSI vs NIHSS/Rankin at discharge and 3‑month follow‑up.
* **Neurodegeneration**: longitudinal effect sizes in posterior alpha power/IAF & DMN wPLI vs cognitive decline scales (MMSE/MoCA).
* **Psychiatry**: pre‑ to post‑treatment changes in network metrics vs clinical scales (HAM‑D, BDI); emphasize *monitoring*, not diagnostic claims.

**Statistical hygiene**

* Multiple‑comparisons control (cluster/TFCE, FDR).
* Test–retest reliability (ICC) for resting measures.
* External validation across sites/hardware when possible.

---

## 6) Safety, Ethics, and Regulatory Notes

* **Device class** & claims: diagnostic vs adjunctive vs research—match documentation accordingly.
* **Bias & generalization**: site/montage differences; use harmonization (e.g., ComBat) for group analyses.
* **Data privacy**: de‑identify MRIs and EEG; manage PHI in DICOMs.
* **Clinical responsibility**: ESI **augments** but does not replace clinical judgment or imaging.

---

## 7) Practical Pipelines (by Indication)

### 7.1 Epilepsy (interictal)

1. Detect spikes → average; prune artifacts.
2. 4‑shell head model + digitized electrodes.
3. eLORETA map at spike peak ±10 ms; **ECD** fit for peak vertex; optional **MxNE** over spike window.
4. Concordance table with MRI/PET/iEEG; distances; suggestion for sEEG targets.

### 7.2 Stroke (rest)

1. 5–10 min EO/EC rest; robust preprocessing.
2. eLORETA/wMNE; parcel extraction (Desikan).
3. Compute DAR, DTABR, IAF; **BSI** on matched parcels.
4. Normative z‑scores; lateralization indices; prognosis model with CI.

### 7.3 Neurofeedback/BCI

1. Precompute $\mathbf{K}$ and parcel projector.
2. Causal filters; envelope extraction; leakage‑robust feature.
3. Latency monitor; feedback display; session‑wise QA.

---

## 8) Common Pitfalls & Guardrails

* **Assuming diagnostic specificity** (e.g., TBR for ADHD) → treat as *supportive*, not definitive.
* **Ignoring head‑model uncertainty** → run sensitivity analysis (±50% skull σ) and acknowledge in the report.
* **Sensor‑space FC** reported as source‑space → always compute in source space with leakage control.
* **Unwhitened inverses** → inflated false positives in standardized maps.
* **Over‑smoothing & over‑parcellation** → unstable metrics; match resolution to SNR/sensors.

---

## 9) Suggested Figures (for slides/reports)

1. **Epilepsy case**: spike topography → eLORETA map → ECD dot → sEEG plan → resection overlap.
2. **Stroke**: source‑space DAR and BSI hemispheric maps with NIHSS scatter.
3. **Neurodegeneration**: posterior alpha power z‑map + IAF shift over time.
4. **Neurofeedback**: latency breakdown bar + online parcel trace.

---

## 10) References & Resources (selected)

**Stroke / qEEG**

* Finnigan, S., & van Putten, M. (2013). qEEG in ischemic stroke. *Clin Neurophysiol*, 124, 10–19.

**Epilepsy / ESI**

* Michel, C. M., & Murray, M. M. (2012). EEG as a brain imaging tool. *NeuroImage*, 61, 371–385.
* Plummer, C., et al. (2019). EEG source imaging in presurgical evaluation: a review. *Clin Neurophysiol*.
* solutions using HD‑EEG and combined modalities (PET/SPECT/iEEG) — various clinical series.

**Neurodegeneration**

* Babiloni, C., et al. (2020). EEG biomarkers in AD. *Neurobiol Aging*, 85, 1–10.

**General / Methods**

* Niedermeyer & Lopes da Silva (2017). *Electroencephalography* (7th ed.). OUP.
* Michel, C. & Brunet, D. (2019). EEG source imaging: a practical review. *Handbook of Clinical Neurology*.

**Societies / Guidance**

* ACNS — [https://www.acns.org/](https://www.acns.org/)
* IFCN — [https://www.ifcn.info/](https://www.ifcn.info/)

---

### Instructor Notes (optional)

* **Case‑based workshop:** provide de‑identified epilepsy and stroke datasets; students produce ESI reports with uncertainty, concordance analysis, and clinical decision suggestions.
* **Validation lab:** compute localization error to sEEG/radiology ground truth and evaluate ROC for region‑level detection.
* **Real‑time lab:** implement a 50 ms‑latency neurofeedback loop using precomputed $\mathbf{K}$ and parcel envelopes; measure end‑to‑end latency.
