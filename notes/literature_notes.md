# Literature notes

**Thesis:** *Clinical Note Intelligence: An NLP Pipeline for Structured Information Extraction and Prescription Anomaly Detection in Electronic Health Records*

**Bibliography:** [`../references.bib`](../references.bib) — 30 sources (key in bold).

## How to read this file

- Each entry says **what is relevant** in the source and **what it is used for** (thesis section or research question).
- ★ = core source, ○ = supporting source. `[ ]` = tick when the PDF is in `../papers/` (as `<key>.pdf`) and you have read it.

## Research questions and the sources behind them

| Research question | Main sources |
|---|---|
| RQ1 – extraction of medications, doses, allergies | devlin2019bert, lee2020biobert, alsentzer2019clinicalbert, henry2020n2c2, wei2020n2c2deep, kormilitzin2021med7, xu2010medex, goss2014allergy, chapman2001negex, nelson2011rxnorm |
| RQ2 – cross-checking against prescriptions, reliable flags | vandersijs2006overriding, topaz2016allergyalerts, shenoy2019penicillin, schiff2017outlier, hogue2021atypical |
| RQ3 – effect of PDF/OCR ingestion | smith2007tesseract, hsu2022scanned, hamdi2023ocrner |
| RQ4 – doctor-controlled platform, human in the loop | goddard2012automationbias, kazzazi2021adam, delbanco2012opennotes, steitz2023immediate, zaretsky2024discharge |

## Design decisions the literature supports

1. **Fine-tune a BERT-class model instead of prompting a general LLM** (`lehman2023clinicallms`, `hu2024promptner`).
2. **Use n2c2 2018 data plus a separate allergy component**, since n2c2 has no allergy entity (`henry2020n2c2`, `goss2014allergy`).
3. **Handle negation and normalise drug names before any check** (`chapman2001negex`, `nelson2011rxnorm`).
4. **Keep flags few, graded and non-repeating** (`vandersijs2006overriding`, `topaz2016allergyalerts`, `shenoy2019penicillin`).
5. **Show evidence, not verdicts** (`goddard2012automationbias`, `kazzazi2021adam`).
6. **Keep raw flags away from patients; review patient summaries** (`steitz2023immediate`, `zaretsky2024discharge`).
7. **Test PDF ingestion with controlled noise** (`hamdi2023ocrner`, `hsu2022scanned`).

---
## Problem motivation (Chapter 1)

- [ ] ★ **sinsky2016allocation** — Sinsky et al., 2016, *Annals of Internal Medicine*.
  - **Relevant:** time-and-motion study of 57 physicians in 4 specialties. 27.0% of office time went to direct clinical face time and 49.2% to EHR and desk work, plus 1–2 hours of EHR work each night.
  - **Use:** the first sentence of Sec. 1.1 (documentation burden). It covers physicians only, so do not use it to support the claim about pharmacists.

## 2.1 Language models for clinical text (RQ1)

- [ ] ★ **devlin2019bert** — Devlin et al., 2019, NAACL.
  - **Relevant:** BERT, a bidirectional transformer pre-trained on unlabeled text and fine-tuned for a task with one extra output layer.
  - **Use:** background for Sec. 2.1; the basis of BioBERT and ClinicalBERT.
- [ ] ★ **lee2020biobert** — Lee et al., 2020, *Bioinformatics*.
  - **Relevant:** continues BERT pre-training on PubMed abstracts and PMC full texts; beats BERT on biomedical NER, relation extraction and question answering.
  - **Use:** candidate model for the extraction step.
- [ ] ★ **alsentzer2019clinicalbert** — Alsentzer et al., 2019, Clinical NLP Workshop.
  - **Relevant:** Bio+ClinicalBERT = BioBERT further pre-trained on about 2M MIMIC-III notes. Improves clinical NER and MedNLI; no gain on de-identification tasks.
  - **Use:** the main candidate model. Name the exact checkpoint in the thesis (`emilyalsentzer/Bio_ClinicalBERT`), because "ClinicalBERT" also names a different model (Huang et al., 2019, a readmission-prediction model).
- [ ] ★ **gu2022pubmedbert** — Gu et al., 2022, *ACM Trans. Comput. Healthcare*.
  - **Relevant:** PubMedBERT and the BLURB benchmark. Pre-training from scratch on in-domain text beats continued pre-training of a general BERT.
  - **Use:** possible third model in the comparison.
- [ ] ★ **lehman2023clinicallms** — Lehman et al., 2023, CHIL (PMLR 209).
  - **Relevant:** 12 models from 220M to 175B parameters on 3 clinical tasks. Small specialised clinical models beat all in-context-learning approaches, even with little fine-tuning data.
  - **Use:** the main argument for a fine-tuned BERT-class model over an LLM.
- [ ] ★ **johnson2016mimic** — Johnson et al., 2016, *Scientific Data*.
  - **Relevant:** MIMIC-III: de-identified data of 53,423 adult admissions to critical care units (2001–2012), with structured tables (including `PRESCRIPTIONS`) and free-text `NOTEEVENTS`. Access needs credentialing and a data use agreement.
  - **Use:** the dataset (Chapter 3); source of notes and structured prescriptions.

## 2.2 Medication and allergy extraction (RQ1)

### Shared tasks and benchmarks

- [ ] ★ **uzuner2010i2b2** — Uzuner, Solti, Cadag, 2010, *JAMIA*.
  - **Relevant:** 2009 i2b2 medication challenge: 1,243 de-identified discharge summaries, 20 teams. Rule-based systems dominated the top ten and the best system was a hybrid. Medication names F > 0.75 for the top ten; duration (best F 0.525) and reason (0.459) were hardest.
  - **Use:** benchmark history; shows which fields are hard.
- [ ] ★ **henry2020n2c2** — Henry et al., 2020, *JAMIA*.
  - **Relevant:** 2018 n2c2 Track 2: 505 MIMIC-III discharge summaries (303 train / 202 test); entities Drug, Strength, Dosage, Duration, Frequency, Form, Route, Reason, ADE. Best concept-extraction F1 0.9418, end-to-end 0.8905. There is **no allergy entity**.
  - **Use:** gold annotations for fine-tuning and evaluation (needs the n2c2 data use agreement); entity schema.
- [ ] ★ **mahajan2023n2c2cmed** — Mahajan et al., 2023, *J. Biomed. Inform.*
  - **Relevant:** 2022 n2c2 contextualized medication event task: 32 teams, 211 systems, mostly transformers. NER is "relatively mature", but medication-change context (started, stopped, continued; negation; temporality) is still open.
  - **Use:** "Continue metoprolol" versus "amoxicillin added" in the running example is exactly this context problem; it limits naive duplicate detection.

### Rule-based and hybrid systems

- [ ] ★ **xu2010medex** — Xu et al., 2010, *JAMIA* (MedEx).
  - **Relevant:** on 50 discharge summaries: drug names F 93.2%; strength, route, frequency F 94.5%, 93.9%, 96.0%.
  - **Use:** a strong rule-based baseline; rules do well on "signature" fields.

### Neural, transformer and LLM systems

- [ ] ★ **wei2020n2c2deep** — Wei et al., 2020, *JAMIA*.
  - **Relevant:** n2c2 2018 participant: NER F1 93.45% (rank 2), relation classification 96.30% (rank 1), end-to-end 89.05% (rank 1). Deep learning beat CRF and SVM. Describes the 303/202 split.
  - **Use:** the strongest pre-transformer baseline on the same data.
- [ ] ★ **kormilitzin2021med7** — Kormilitzin et al., 2021, *Artif. Intell. Med.* (Med7).
  - **Relevant:** NER with 7 classes (drug, route, frequency, dosage, strength, form, duration) built on MIMIC-III, micro-F1 0.957. On UK mental-health records it fell to 0.762; fine-tuning on a small sample restored 0.944.
  - **Use:** an entity schema close to the thesis one, so a natural baseline; evidence that fine-tuning on the target domain matters (RQ3).
- [ ] ★ **hu2024promptner** — Hu et al., 2024, *JAMIA*.
  - **Relevant:** with full prompt engineering GPT-4 reached relaxed F1 0.861 on MTSamples against 0.901 for BioClinicalBERT.
  - **Use:** quantitative support for fine-tuning a BERT-class model.

### Negation, normalisation and allergies

- [ ] ★ **chapman2001negex** — Chapman et al., 2001, *J. Biomed. Inform.* (NegEx).
  - **Relevant:** simple regex negation algorithm: specificity 94.5%, PPV 84.5%, sensitivity 77.8%.
  - **Use:** needed so that "NKDA" or "no allergy to penicillin" does not raise a false conflict.
- [ ] ★ **nelson2011rxnorm** — Nelson et al., 2011, *JAMIA* (RxNorm).
  - **Relevant:** normalised drug names with relations across 11 source vocabularies.
  - **Use:** brand-to-ingredient mapping (e.g. Lopressor = metoprolol) for duplicates and allergy classes.
- [ ] ★ **goss2014allergy** — Goss et al., 2014, AMIA Annual Symposium.
  - **Relevant:** MTERMS allergy module on 400 emergency-department notes: F 87.6% for allergen names and "no known allergies", 90% for reactions, but only 69% for linking a reaction to its allergen.
  - **Use:** baseline expectations for the allergy entity; relation linking is the hard part.

## 2.3 Prescription checking and medication safety (RQ2, RQ4)

### Decision support and alert fatigue

- [ ] ★ **vandersijs2006overriding** — van der Sijs et al., 2006, *JAMIA*.
  - **Relevant:** literature review: clinicians override drug safety alerts in 49–96% of cases; low specificity and unclear information are error-producing conditions.
  - **Use:** the "few and precise flags" design choice.
- [ ] ★ **topaz2016allergyalerts** — Topaz et al., 2016, *JAMIA*.
  - **Relevant:** 611,192 drug-allergy alerts at 2 hospitals; overrides rose from 83.3% to 87.6% (2004–2013). More than half of override reasons called the alert irrelevant, most often because the patient had tolerated the drug before. Repeat alerts were overridden more (89.7%).
  - **Use:** the allergy flag should consider tolerance documented in the notes and should not repeat itself.
- [ ] ★ **shenoy2019penicillin** — Shenoy et al., 2019, *JAMA*.
  - **Relevant:** about 10% of the US population reports a penicillin allergy, but fewer than 5% are clinically significant; penicillin–cephalosporin cross-reactivity is about 2%.
  - **Use:** the drug-class logic of the running example, and graded flags (same class = high, cross-class = low).

### Data-driven anomaly and outlier detection

- [ ] ★ **schiff2017outlier** — Schiff et al., 2017, *JAMIA*.
  - **Relevant:** outlier-detection system on 747,985 outpatients; about 75% of alerts valid and about 75% of those useful. The MedAware studies are vendor-affiliated; say so when citing.
  - **Use:** closest prior work to the atypical-dose check.
- [ ] ★ **hogue2021atypical** — Hogue et al., 2021, *JAMIA*.
  - **Relevant:** unsupervised autoencoder on 12,471 orders labelled by 25 pharmacists: per-order P 35%, R 26%, F1 0.30; whole profiles F1 0.59. Pharmacists found it useful for screening.
  - **Use:** realistic expectations: per-order anomaly detection is hard.

### Human factors: automation bias (RQ4)

- [ ] ★ **goddard2012automationbias** — Goddard, Roudsari, Wyatt, 2012, *JAMIA*.
  - **Relevant:** systematic review of 74 studies. Automation bias depends on trust, workload, task complexity and time pressure. Mitigations: training, accountability, placement of advice, confidence levels, giving information rather than recommendations.
  - **Use:** design rationale: flags as evidence-linked suggestions beside the source text, with the doctor as accountable approver.
- [ ] ★ **kazzazi2021adam** — Kazzazi, 2021, *Future Healthc. J.* (ADAM framework).
  - **Relevant:** five-level classification of AI tools in medicine; defines augmented AI as an adjunct to human intelligence that does not replace it; lists barriers such as algorithmic bias and automation bias.
  - **Use:** positions the system at the lowest levels (adjunctive AI); quote for the introduction; risks discussion.

## 2.4 PDF ingestion and OCR (RQ3)

- [ ] ★ **smith2007tesseract** — Smith, 2007, ICDAR.
  - **Relevant:** overview of the Tesseract open-source OCR engine, originally developed at HP.
  - **Use:** OCR fallback for scanned PDFs.
- [ ] ★ **hamdi2023ocrner** — Hamdi et al., 2023, *Nat. Lang. Eng.*
  - **Relevant:** controlled OCR-noise experiments on NER and entity linking. NER F falls from about 90% to 60% as the error rate rises into roughly the 8–20% range.
  - **Use:** direct support for the RQ3 hypothesis and a template for the experiment (inject noise, measure F1 against character error rate).
- [ ] ★ **hsu2022scanned** — Hsu et al., 2022, *JAMIA Open*.
  - **Relevant:** 955 scanned sleep-study reports processed with OCR and ClinicalBERT: document accuracy 94.76% (AHI) and 91.61% (SaO2). Image preprocessing and layout information both mattered.
  - **Use:** the closest clinical analogue of the PDF pipeline.

## 2.5 Patient access to records and patient-facing summaries (RQ4)

- [ ] ★ **delbanco2012opennotes** — Delbanco et al., 2012, *Ann. Intern. Med.*
  - **Relevant:** OpenNotes quasi-experiment: patients reading their doctors' notes reported better understanding and recall of their care plan; only 1–8% reported worry, confusion or offense; many wanted to add comments.
  - **Use:** supports a shared space where patients can see approved information and contribute documents.
- [ ] ★ **steitz2023immediate** — Steitz et al., 2023, *JAMA Netw. Open*.
  - **Relevant:** survey of 8,139 patients after immediate release of results: 95.7% preferred immediate access; increased worry for 16.5% with abnormal results versus 5.0% with normal ones.
  - **Use:** justifies keeping raw flags visible to professionals only.
- [ ] ★ **zaretsky2024discharge** — Zaretsky et al., 2024, *JAMA Netw. Open*.
  - **Relevant:** GPT-4 rewrote 50 discharge summaries; reading grade fell from 11.0 to 6.2 and understandability (PEMAT) rose from 13% to 81%, but some outputs were inaccurate or omitted details.
  - **Use:** plain-language summaries, with physician review required.

---

## Open gap (for the end of Chapter 2)

Medication NER on clinical notes is mature (F1 about 0.93–0.98 on n2c2 benchmarks). Allergy extraction is rarely studied and weaker at linking reaction to allergen. Prescription-error detection, rule-based or ML-based, works mostly on **structured orders** and suffers from alert fatigue. The effect of PDF/OCR ingestion on clinical medication NER is largely unmeasured. Based on the sources reviewed, few systems connect note-level extraction to prescription checking in a clinician-controlled workflow; this thesis addresses that intersection.
