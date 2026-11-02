# Cybersecurity Deep Research - Ten Problem Statements

**Prepared:** 2026-11-02 | **Repository:** `cybersec_research` | **Companion files:** `gap_analysis_matrix.md`, `industry_telemetry/`, `academic_gaps/`, `extracted_evidence/evidence_ledger.md`

## How to read this document

- Each problem statement has the eight required sections. Bracketed IDs such as **[E01]** or **[A10]** point to rows in `extracted_evidence/evidence_ledger.md`.
- **Evidence honesty.** Only the Mandiant M-Trends 2026 report was read at primary level (**P**). Every other industry figure comes from secondary coverage (**S**), and every academic claim from abstracts or snippets (**A**). Several of the hosts that publish primary documents were blocked in the research environment (see `extracted_evidence/research_log.md`). Re-verify any number before it enters a paper.
- **Dataset honesty.** Datasets are named from prior knowledge or from search results; their current availability, licensing and download paths were **not** verified live. Items marked *(verify)* are ones where I am less certain of the exact release.
- **Safety.** All offensive emulation in the methodologies is intended for isolated lab ranges with benign payloads and synthetic data. No weaponised content is specified here.
- **Targets are hypotheses.** Numeric thresholds like "X%" are to be fixed after a pilot, not claimed results.

## Index

| # | Primary domain | Secondary domains | Short title |
|---|---|---|---|
| PS-01 | Threat Intelligence | SIEM, Malware | Actor-conditioned IOC survival modelling |
| PS-02 | Incident Response | Threat Intelligence, Malware | Sub-minute hand-off prediction and containment policy |
| PS-03 | Digital Forensics | XDR, IR, Ransomware | Minimal tamper-evident artifacts for hypervisor/edge compromise |
| PS-04 | Vulnerability Management | Threat Intelligence, CTEM | Prioritisation when time-to-exploit is negative |
| PS-05 | Malware / Ransomware Analysis | XDR, AI | Intermittent encryption + EDR-killer adversarial robustness |
| PS-06 | AI in Cybersecurity | Threat Intelligence, XDR | Telemetry signatures of LLM-orchestrated intrusions |
| PS-07 | XDR | SIEM, Forensics, Identity | Partial-observability incident graphs across identity/endpoint/edge |
| PS-08 | CTEM | Vulnerability Mgmt, Cloud IAM | Scalable multi-cloud attack-path prioritisation with safe validation |
| PS-09 | SIEM | AI, Detection Engineering | Execution-validated detection coverage and rule liveness |
| PS-10 | Defender AI | SIEM, AI, IR | Prompt-injection-resilient SOC agents via taint and capability control |

---

## PS-01 - Threat Intelligence

### 1. Title
**Actor-Conditioned Indicator Survival Modelling: Estimating Per-IOC Time-to-Live and Cross-Feed Marginal Utility to Convert Shared CTI into Time-Bounded, Auditable Detections**

### 2. Domain Mapping
- **Primary:** Threat Intelligence
- **Secondary:** SIEM / detection engineering (consumption of indicators), Malware analysis (infrastructure of commodity loaders)

### 3. Current State of the Art
- Operational CTI is shared as STIX/TAXII and MISP objects; indicators carry an expiry set by type or by feed policy (fixed TTLs), and platforms apply reputation/aggregation scores.
- Academic work on feeds evaluates overlap, novelty and aging (e.g., an OUSPG analysis of 16 free feeds) and recently scores feeds on uniqueness and maintenance. A SANS.edu paper argues decay should be actor-specific and analysed hundreds of IOCs across three APTs **[A17]**.
- LLM pipelines extract TTPs and indicators from reports (IntelEX and others) **[A18]**.

### 4. The Live Industry Pain Point
- Mandiant states that "relying solely on static indicators of compromise (IOCs) is no longer sufficient," while the median hand-off from initial access to a secondary actor fell from more than 8 hours (2022) to 22 seconds (2025) **[E01, P]**.
- Roughly 82% of 2025 detections were reported as malware-free in CrowdStrike's 2026 report **[E02, S, unverified]**, and 56% of Sophos IR/MDR cases involved adversaries "logging in" **[E06, S]**, so indicator matching addresses a shrinking slice of intrusions.
- Malware families now call LLMs at runtime to regenerate code and commands (PROMPTFLUX, PROMPTSTEAL) **[E01 P, E14 S]**, further weakening file-hash and string indicators.
- Providers overlap by no more than ~4.5% on similar intelligence types **[A16]**, so buyers pay for multiple feeds without knowing each indicator's remaining useful life.

### 5. The Academic/Technical Gap
- No standard, peer-reviewed estimator of IOC time-to-live conditioned on actor behaviour and infrastructure features was found; the closest (SANS.edu) is small-sample and not peer-reviewed **[A17]**. One cited IoT-botnet study (90% of C2 lived under 5 days) is old and domain-specific **[secondary, Netresec]**.
- Feed quality is judged by proxies (overlap/novelty/aging), not by downstream detection outcome.
- **Why non-trivial:** (a) *right-censoring* - an IOC is only observed while some party probes it; (b) *observation bias* - feeds record what their vendor sees; (c) *death ambiguity* - an indicator may disappear from a feed due to takedown/sinkholing, adversary rotation, or vendor curation; (d) *reflexivity* - publishing an indicator accelerates adversary rotation, so the data-generating process shifts once the model is deployed; (e) *label leakage* - feed membership correlates with other features at labelling time.

### 6. Core Research Question
Can a censoring-aware survival model conditioned on actor/campaign attribution, indicator type and infrastructure features (ASN churn, registrar, certificate reuse, hosting pattern) predict per-IOC time-to-live well enough that expiry policies derived from it reduce stale-indicator matches by a pre-registered margin (hypothesis: >=30%) at fixed true-positive coverage, versus fixed-TTL baselines, and does this hold for **unseen actors** and under an **adaptive adversary** who accelerates rotation once expiry is predicted?

### 7. Concrete Evaluation Methodology
1. **Longitudinal corpus (>=12 months).** Snapshot open feeds daily (ThreatFox, URLhaus, MalwareBazaar, Feodo Tracker, OTX pulses, CIRCL OSINT/MISP). Store first-seen, last-seen, status changes, and tags. Enrich with passive DNS-like resolution history from your own periodic resolution, certificate transparency (crt.sh) and RDAP/WHOIS.
2. **Event definition and censoring.** Define IOC "death" operationally and in three variants (stops resolving/serving; removed from source feed; reclassified benign). Record censoring explicitly; run all analyses on each variant to measure label sensitivity.
3. **Attribution labels.** Map pulses/families to ATT&CK groups/software using the MITRE ATT&CK STIX data; keep an "unattributed" class and report how many indicators fall there.
4. **Models.** Kaplan-Meier by type/actor (descriptive), Cox PH with time-varying covariates, accelerated failure time models, and a gradient-boosted/random survival forest. Metrics: concordance index, integrated Brier score, time-dependent AUC, calibration curves.
5. **Validation protocol.** Rolling-origin temporal split (train on months 1..k, test on k+1) and **leave-one-actor-out** to test generalisation. Report all results with block-bootstrap CIs.
6. **Downstream detection replay.** Replay a benign DNS/proxy corpus (Tranco top sites as benign background plus lab-generated user traffic) and injected malicious connections drawn from held-out periods. Compare policies: vendor-default TTL, fixed 7/30/90-day TTL, model-predicted TTL. Metrics: stale-match false-positive rate, indicator-set size, time-to-first-valid-match, fraction of true malicious matches retained.
7. **Adaptive adversary stress test.** Simulate an adversary that shortens lifetimes once expiry is exposed (parametric rotation-rate multiplier; also a game-theoretic best-response variant). Report degradation vs multiplier.
8. **Threats to validity.** Feed-specific bias, definition sensitivity, reflexivity; report a cross-feed replication using at least two disjoint feed families.

### 8. Feasible Datasets & Lab Telemetry Sources
- **abuse.ch** ThreatFox, URLhaus, MalwareBazaar, Feodo Tracker (public APIs/CSV) - indicator lifecycle and malware tags.
- **AlienVault/LevelBlue OTX** pulses and the **CIRCL OSINT feed via MISP** - multi-vendor overlap analysis.
- **MITRE ATT&CK STIX data** (`mitre-attack/attack-stix-data`) - actor/software labels.
- **Certificate Transparency (crt.sh)**, **RDAP/WHOIS**, own periodic DNS resolution - infrastructure features.
- **Tranco top-sites list** - benign background for false-positive measurement.
- **Lab telemetry:** a small DNS resolver + proxy farm generating benign user-like traffic; sandbox-detonated samples (CAPE) to obtain contemporaneous C2 indicators. *(All dataset availability to be verified.)*
- **Evidence anchors:** E01, E02, E06, E14, A16, A17, A18.

---

## PS-02 - Incident Response

### 1. Title
**Sub-Minute Containment Decision-Making: Calibrated Hazard Prediction of Secondary-Actor Hand-off from First-Stage Alerts Under Asymmetric Response Costs**

### 2. Domain Mapping
- **Primary:** Incident Response
- **Secondary:** Threat Intelligence (actor/loader linkage), Malware analysis (first-stage tooling), SIEM/SOAR (response automation)

### 3. Current State of the Art
- IR playbooks and SOAR automate enrichment and ticketing; commodity-malware alerts are usually low priority. MDR providers staff 24x7 but still operate on investigation timescales.
- Research on autonomous response uses RL in simulators (CybORG/CAGE) and, recently, LLM agents (arXiv 2505.04843) **[A22]**. Benchmarks for LLM-SOC (ExCyTIn-Bench, CyberSOCEval, Simbian) measure investigation quality **[A13, A14]**.

### 4. The Live Industry Pain Point
- Hand-off to a secondary group: more than 8 hours in 2022 to **22 seconds** in 2025; Mandiant advises treating routine malware alerts as high priority and remediating "before interactive hands-on-keyboard operations begin" **[E01, P]**.
- CrowdStrike reports an average breakout of ~29 minutes and a fastest of 27 seconds **[E02, S]**; Sophos reports a median of 11 hours to a first Active Directory attempt **[E06, S]**. These populations differ (eCrime telemetry vs IR/MDR case mix), which is itself a measurement gap.
- Blanket isolation of every first-stage alert is operationally unaffordable; the decision is cost-asymmetric (business disruption vs breach cost) and must be made in seconds.

### 5. The Academic/Technical Gap
- RL defenders trained in simulators lose performance when attacker strategy, topology or scale changes **[A22]**; real-system transfer is untested.
- LLM-SOC benchmarks score investigation correctness; none found score **containment decisions under time budget and asymmetric cost** **[A13, A14]**. Simulation realism of attack data is itself "an open question" **[A21]**.
- **Why non-trivial:** (a) labelled hand-off timelines are scarce and mostly proprietary; (b) *missing counterfactual* - once you contain, you never observe the hand-off, so naive supervised learning is biased by the policy that generated the data; (c) concept drift across families/brokers; (d) timing features are attacker-controllable (an adversary can delay hand-off to evade a timing model); (e) calibration, not accuracy, determines decision value.

### 6. Core Research Question
Given only the telemetry available at first-stage alert time, can a calibrated discrete-time hazard model estimate P(hands-on-keyboard activity within tau seconds) with a calibration error below a pre-registered bound (hypothesis: ECE <= 0.05) such that a cost-sensitive containment policy attains lower expected loss than (i) isolate-all, (ii) severity-rule triage and (iii) zero-shot LLM triage - and how fast does this advantage erode when the attacker deliberately delays hand-off?

### 7. Concrete Evaluation Methodology
1. **Timeline corpus.** Extract time-stamped event chains (initial access, loader execution, C2 beacon, discovery, credential access, lateral movement, hands-on activity) from public DFIR case write-ups. Two annotators code each case; report Cohen's kappa. Keep the corpus small and clearly labelled "real", then add:
2. **Lab replay corpus.** In an isolated AD range, replay documented chains with an emulation framework (MITRE Caldera and Atomic Red Team plans), parameterising the loader-to-operator delay from a distribution spanning seconds to hours so that the 22-second regime is represented. Collect Sysmon/EDR-class, Zeek and identity logs.
3. **Features at decision time only** (no future leakage): first-stage tool/family, parent-child lineage, egress destination class, host role/criticality, identity context, time of day.
4. **Models.** Discrete-time hazard (GBM or neural), with conformal prediction for coverage guarantees. Baselines: severity rules; isolate-all; zero-shot LLM triage; a CybORG-trained RL policy.
5. **Decision layer.** Expected-loss minimisation with a cost matrix varying by host role; sensitivity analysis over cost ratios. Where logging propensities are known (lab), use inverse-propensity or doubly robust off-policy evaluation to correct for the contain-and-never-observe problem.
6. **Metrics.** Concordance, Brier score, ECE; decision metrics: contained-before-hand-off rate, false-isolation host-hours, expected loss, analyst-minutes saved.
7. **Adaptive-adversary test.** Insert a hand-off delay/jitter strategy optimised against the deployed hazard model (black-box search) and quantify the expected-loss regret.
8. **Threats to validity.** Small real-case N, lab/real mismatch, annotation subjectivity; report sensitivity to each.

### 8. Feasible Datasets & Lab Telemetry Sources
- **The DFIR Report** public case write-ups (timelines for annotation).
- **MITRE Caldera**, **Atomic Red Team**, **Splunk Attack Range**, **GOAD (Game of Active Directory)** for lab replays.
- **Splunk `attack_data`**, **OTRF Security-Datasets (Mordor)**, **EVTX-ATTACK-SAMPLES**, **DARPA OpTC** for background and technique telemetry.
- **CybORG / CAGE Challenge** environments for the RL baseline.
- **ExCyTIn-Bench** and **CyberSOCEval** for the LLM triage baseline *(verify release terms)*.
- **Priors only (not training data):** M-Trends, Sophos, CrowdStrike aggregates for hand-off/breakout distributions.
- **Evidence anchors:** E01, E02, E06, A13, A14, A21, A22.

---

## PS-03 - Digital Forensics

### 1. Title
**Tamper-Evident Timeline Reconstruction for Telemetry-Poor Planes: Minimal Out-of-Band Artifact Sets for Hypervisor Management-Plane and Network-Edge Compromise**

### 2. Domain Mapping
- **Primary:** Digital Forensics
- **Secondary:** XDR (missing telemetry planes), Incident Response (scoping), Malware/Ransomware (hypervisor-level encryption)

### 3. Current State of the Art
- Host forensics (disk and memory acquisition, timeline tools) assumes a general-purpose OS. Cloud forensics relies on provider audit logs, snapshots and pre-positioned collection.
- For appliances and hypervisors, practice is vendor integrity tools, syslog forwarding and configuration diffs against known-good baselines. Government guidance (ASD/ACSC, Feb 2025; UK NCSC) asks vendors to provide structured logging, authenticated collection and defined forensic interfaces **[E17]**.
- LLM-for-DFIR work is benchmarked on knowledge and string-search tasks (DFIR-Metric) **[A15]**.

### 4. The Live Industry Pain Point
- Edge devices have "minimal onboard storage and cannot support traditional security tooling," forensics leaves "limited artifacts," and BRICKSTORM "routinely survives standard remediation efforts and system reboots" with dwell time of nearly 400 days **[E01, P]**.
- Hypervisors are "Tier-0": encrypting datastores or the storage layer takes down every VM at once; Akira and Qilin affiliates target backups, identity and virtualization management **[E01, P]**.
- Vendor DFIR reports describe decentralised ESXi logging and attacker disabling or redirecting syslog; one aggregator attributes a rise of hypervisor-focused ransomware from 3% to 25% of incidents in 2025 to Huntress data (**unverified**) **[E18]**.
- A default 90-day log retention leaves defenders "completely blind to the initial access vector" **[E01, P]**.

### 5. The Academic/Technical Gap
- No peer-reviewed treatment was surfaced (in this session's searches) of ESXi/hypervisor-plane or embedded-appliance forensics, nor of evidence sufficiency for those planes **[academic_gaps/03]**. The DFRWS literature may contain relevant work that the blocked environment prevented me from reaching.
- **Why non-trivial:** (a) closed, heterogeneous firmware limits acquisition and verification; (b) volatile storage (logs on RAM disks) is lost on reboot; (c) acquisition on a production hypervisor can perturb evidence or availability; (d) **evidence sufficiency** is undefined - there is no accepted way to state "these artifacts are enough to bound timeline error"; (e) adversary anti-forensics (log disabling, selective deletion) must be modelled, not ignored.

### 6. Core Research Question
For two compromise classes - (a) hypervisor management-plane takeover with datastore-level impact and (b) network-edge appliance implant - what is the **smallest set of out-of-band, tamper-evident artifacts** (e.g., config-hash attestations, control-plane syslog to an append-only sink, datastore/VM-disk metadata journals, flow data, periodic memory or firmware measurements) that lets an examiner reconstruct the attacker's action timeline with bounded error (hypothesis: >=90% of ground-truth actions recovered within +/-5 minutes) **after** the adversary performs documented anti-forensic actions?

### 7. Concrete Evaluation Methodology
1. **Build two labs.** (a) A virtualization cluster (ESXi if licensing allows; otherwise KVM/Proxmox or Hyper-V as an explicitly labelled proxy whose transferability must be argued). (b) Virtual network appliances (open-source or licensed evaluation images) in a routed topology.
2. **Ground-truth recorder.** Capture attacker actions out-of-band (serial console/hypervisor-level recorder, or a second-level monitor not reachable by the adversary) to create a verified action timeline for scoring.
3. **Action library (benign analogues).** Scripted post-compromise actions based on public reports: enabling remote shell access, altering or redirecting logging, config tampering, a benign "marker" persistence implant, datastore-level encryption of throw-away VMs using a lab-written benign encryptor. Randomise order and timing; include 3-4 anti-forensic strategies.
4. **Artifact arms (ablation ladder).** A0 default logging only; A1 + remote syslog; A2 + shell/audit logging; A3 + periodic config/firmware hash attestation to an append-only store; A4 + memory snapshots; A5 + flow/netflow; A6 + storage-layer metadata journal. Run >=30 randomised trials per arm.
5. **Metrics.** Action recall and precision, timeline error (minutes), tamper-detection rate (can you prove logs were altered?), evidence integrity (hash-chain verification), acquisition overhead and availability impact, marginal evidentiary value per artifact (Shapley decomposition).
6. **Optional LLM-assisted timeline synthesis.** Compare an LLM-assisted workflow with a manual baseline on consistency and hallucinated-event rate (reuse DFIR-Metric-style repeated trials) **[A15]**.
7. **Threats to validity.** Proxy hypervisors, simplified anti-forensics, lab scale; state limits explicitly and avoid claiming vendor-specific guarantees.

### 8. Feasible Datasets & Lab Telemetry Sources
- **Lab-generated** (primary): hypervisor logs, appliance syslog/config snapshots, flow records, memory/disk images with ground-truth timelines.
- **Public forensic corpora:** NIST CFReDS images, Digital Corpora, DFRWS challenge datasets, Volatility Foundation memory samples *(verify)*.
- **Public case studies for action-library design:** Blackpanda Qilin/ESXi case study, SRA ESXi hunting write-up, Mandiant M-Trends 2026 **[E01, E18]**.
- **Guidance to operationalise:** ASD/ACSC and UK NCSC network-device forensic requirements **[E17]**.
- **Cloud extension (optional):** public CloudTrail-based challenge environments for ephemeral-workload evidence.
- **Evidence anchors:** E01, E17, E18, E19, A15.

---

## PS-04 - Vulnerability Management

### 1. Title
**Prioritisation in the Negative Time-to-Exploit Regime: Pre-Disclosure Exploitation Signals and Leakage-Free Evaluation of EPSS- and KEV-Based Triage**

### 2. Domain Mapping
- **Primary:** Vulnerability Management
- **Secondary:** Threat Intelligence (exploitation evidence), CTEM (exposure prioritisation)

### 3. Current State of the Art
- CVSS (severity), EPSS (probability of exploitation in 30 days), CISA KEV (known exploited), SSVC decision trees, commercial risk scores and reachability analysis. NVD enrichment supplies CPE/CVSS inputs; since April 2026 NIST enriches a prioritised subset **[E12]**.

### 4. The Live Industry Pain Point
- Mandiant's estimated mean time-to-exploit is **-7 days**: exploitation often precedes patch availability; exploits are the top initial infection vector for the sixth consecutive year (32%) **[E01, P]**.
- VulnCheck: **28.96%** of KEVs showed exploitation evidence on or before CVE publication day in 2025 (32.1% at mid-year; 23.6% in 2024); 884 newly evidenced exploited CVEs **[E10, S]**.
- Verizon's 2026 DBIR reports exploitation overtaking credentials as the leading initial access vector (~31%) **[E05, S]**; CISA added 245 KEV entries in 2025 (+20%), 94 of them older CVEs **[E11, S]**.
- NVD backlog estimates range from more than 27,000 to more than 40,000 unenriched CVEs, with an Inspector General projection above 60,000 **[E12, S]** - SLA clocks keyed to enrichment start late.

### 5. The Academic/Technical Gap
- Evaluations of EPSS **disagree**: one study (arXiv 2508.13644; listing says CCS 2025) finds <20% of 1,226 KEV CVEs had EPSS >= 0.5 before listing, 8.3% ever reached >= 0.9 and >22% had no EPSS score **[A07]**; another argues retrospective tests leak time and, corrected, EPSS beats a CVSS-only baseline across 134,821 CVEs **[A08]**.
- Reachability studies show strong false-positive reduction in specific ecosystems (31/121 confirmed in 12 Go projects) but are language-specific and ignore infrastructure mitigations **[A09]**.
- **Why non-trivial:** (a) CVEs exploited *before* disclosure by definition lack public per-CVE features at decision time, so models must use non-CVE signals (vendor patch release notes, commit/patch diffs, advisory text, early scan/honeypot traffic); (b) the label "KEV date-added" lags first exploitation and reflects CISA's visibility; (c) rolling-origin evaluation needs *as-of-date* feature snapshots, which are rarely archived; (d) remediation capacity constrains what ranking improvements are worth.

### 6. Core Research Question
Under strict temporal validity (all features frozen as of the decision time; labels defined by first evidence of exploitation rather than KEV date-added), can a model that adds pre-disclosure signals (patch-diff security-relevance, vendor advisory language, asset-exposure context, early-scanner/honeypot activity) improve recall@k of eventually-exploited CVEs at disclosure and at disclosure + 24 h over EPSS, CVSS and KEV-only baselines, and how many exposure-days are avoided per remediation-hour under realistic capacity limits?

### 7. Concrete Evaluation Methodology
1. **Leakage-free dataset.** Reconstruct as-of-date states: daily EPSS score archives; CVE record version history from the CVE List V5 repository commits; NVD last-modified history; KEV `dateAdded`; first exploitation evidence from VulnCheck's community KEV; Exploit-DB/Metasploit module dates; PoC repository creation dates.
2. **Label design.** Define E0 (exploited on or before publication), E7, E30 with right-censoring; run sensitivity with alternate label sources.
3. **Feature families.** (i) Static: vendor, product class, CWE, attack vector. (ii) Pre-disclosure: patch-diff features for open-source fixes, advisory embeddings, vendor patch cadence. (iii) Exposure: internet-visible instance counts (where obtainable), asset context. (iv) Early signals: scanner/honeypot activity volumes if available.
4. **Baselines.** CVSS thresholds, EPSS (as-of-date), KEV-only, SSVC-like tree, a reachability-augmented variant for open-source libraries.
5. **Replication study first.** Reproduce the contradictory EPSS findings with explicit as-of-date handling to identify *where* the disagreement originates (score snapshotting, population, label date).
6. **Evaluation.** Rolling-origin, per-vendor-block bootstrap. Metrics: recall@k, precision@k by E0/E7/E30, lead-time gain (days earlier than KEV), decision-curve analysis and a capacity-constrained simulation (patch budget sweep) reporting exposure-days avoided.
7. **Robustness.** Evaluate on edge-device-only and open-source-library-only subsets, where the economics differ.
8. **Threats to validity.** Survivorship in KEV, vendor visibility bias, label noise; report both optimistic and pessimistic label definitions.

### 8. Feasible Datasets & Lab Telemetry Sources
- **CISA KEV** JSON/CSV; **NVD** JSON/API; **CVE List V5** GitHub repository (record history).
- **FIRST EPSS** daily score archive; **VulnCheck KEV** (community access); **Exploit-DB**; **Metasploit** module metadata; **PoC-in-GitHub** style trackers; **GitHub Advisory Database / OSV**; vendor **CSAF** advisories *(verify each for licence and historical depth)*.
- **GreyNoise community** (scanner behaviour) and a **lab honeypot fleet** (e.g., T-Pot) for early-signal features.
- **Lab:** reproduce a small set of vulnerable service containers to validate patch-diff feature extraction.
- **Evidence anchors:** E01, E05, E10, E11, E12, A07, A08, A09.

---

## PS-05 - Malware / Ransomware Analysis

### 1. Title
**Adversarial Robustness of Ransomware Detection Under Parameterised Intermittent Encryption and Pre-Encryption EDR Neutralisation: Detectability Bounds and Fused Precursor Signals**

### 2. Domain Mapping
- **Primary:** Malware / Ransomware Analysis
- **Secondary:** XDR/EDR (sensor tampering), AI (adversarial ML evaluation)

### 3. Current State of the Art
- Detectors use whole-file or windowed entropy/chi-square tests, canary files and I/O-burst heuristics, learned histogram CNNs, and online-learning classifiers; defenders also rely on immutable backups **[A04, A05]**.
- Industry tracks EDR killers as a separate pre-encryption stage (ESET: ~90 tools, 54 BYOVD over 35 drivers, plus script-based, anti-rootkit and emerging driverless approaches) **[E13]**.

### 4. The Live Industry Pain Point
- Ransomware families expose configurable intermittent-encryption modes (BlackCat "dot pattern," "smart pattern," "head only"; related settings in Akira and LockBit) that preserve most plaintext structure while rendering files unusable **[A04]**.
- Affiliates choose and deploy EDR killers immediately before encryption; The Gentlemen standardise a suite for affiliates; ESET suspects AI-assisted development of some tools **[E13, S]**.
- Encryption has moved to hypervisor datastores and backup planes **[E01, P]**, and operators increasingly rely on exfiltration (76% of Q3-2025 cases; exfil-only payment rate 19%) **[E07, S]**, so encryption-centric detection is only one part of the harm model.

### 5. The Academic/Technical Gap
- Detectability limits have been derived for histogram-based detectors against fixed intermittent settings **[A04]**; not derived for **fused** detectors or against **adaptive** attackers who tune stride/ratio/entropy shaping against the detector (the "attacker moves second" problem **[A10]**).
- Evaluation metrics emphasise accuracy/F1; the operational metric - **bytes or files lost before first alert** - is rarely reported. Malware-ML evaluations often suffer temporal leakage **[A06]**.
- Detection at the storage/hypervisor layer (VM disk images, datastores) and the coupling between EDR-killer precursors and the encryption stage are not studied in the surfaced literature.
- **Why non-trivial:** intermittent encryption reduces the statistical footprint per file by design; entropy can be manipulated; legitimate high-entropy files (compressed media, archives, VM images) create a hard FPR floor; precursor signals (driver loads) are also produced by legitimate software.

### 6. Core Research Question
What is the empirical detectability frontier of byte-statistics-based ransomware detection as a function of encrypted fraction, stride and file type - and can fusing it with pre-encryption precursor signals (anomalous kernel-driver loads, security-service tampering, backup/shadow-copy deletion) keep a pre-registered operating point (hypothesis: TPR >= 0.95 at FPR <= 1e-3, with <= 1% of bytes encrypted before first alert) when the adversary adaptively optimises intermittent parameters and EDR-neutralisation method against the detector?

### 7. Concrete Evaluation Methodology
1. **Safe encryptor emulator.** Write a non-malicious emulator in an isolated VM range that applies full, head-only, fixed-stride, percentage-based and randomised patterns (parameterised as in A04) and optional byte-level encodings, only to synthetic/benign corpora with snapshot rollback.
2. **Corpus.** Govdocs1-style mixed documents, media, archives, databases and VM disk images (VMDK/qcow2), with a dedicated benign-high-entropy subset for FPR stress.
3. **Detectors.** (a) whole-file entropy baseline; (b) windowed histogram/chi-square; (c) the histogram CNN from A04 (reproduce); (d) I/O-pattern sequence model using write offsets/sizes (Linux eBPF or Windows minifilter telemetry); (e) fused model adding precursor features.
4. **Detectability analysis.** Derive and empirically validate minimum detectable encrypted fraction vs sample size for each detector family; report ROC surfaces over (fraction, stride, file type).
5. **Adaptive attacker.** Black-box optimisation (Bayesian optimisation / evolutionary search / RL) over emulator parameters with the detector in the loop; also a white-box variant for differentiable detectors. Report worst-case TPR at fixed FPR and bytes-lost-before-detection.
6. **Precursor stage.** Emulate Impair-Defenses and Inhibit-System-Recovery behaviours using public adversary-emulation test cases; do not load vulnerable drivers outside a throw-away VM. Evaluate precursor lead time before encryption starts.
7. **Real-sample grounding.** In an isolated CAPE sandbox, analyse public ransomware samples to extract real configuration patterns and compare with the emulator's coverage (requires institutional approval and containment).
8. **Time-aware evaluation.** Temporal splits for any learned component to avoid leakage **[A06]**. Report CPU/IO overhead.

### 8. Feasible Datasets & Lab Telemetry Sources
- **Synthetic corpora** built from Govdocs1 (Digital Corpora) plus generated media/VM images *(verify)*.
- **MalwareBazaar / VirusShare** samples (restricted-use, isolated) and **CAPE Sandbox** reports for configuration extraction.
- **Atomic Red Team** tests for defence impairment and recovery inhibition; **Sigma** driver-load rules for baseline precursor detectors.
- **ESET public IoCs/analysis** for EDR-killer families *(verify availability)*.
- **EMBER** (static features baseline) and **CIC-MalMem-2022** (memory-based malware dataset) *(verify)*.
- **malware-traffic-analysis.net** captures for network-side correlation.
- **Lab telemetry:** eBPF/minifilter block-I/O traces, Sysmon driver-load events, hypervisor datastore I/O counters.
- **Evidence anchors:** E01, E07, E13, A04, A05, A06, A10.

---

## PS-06 - AI in Cybersecurity

### 1. Title
**Victim-Side Attribution of LLM-Orchestrated Intrusions: Scaffold-Agnostic Telemetry Signatures of Agentic Tradecraft versus Human and Scripted Operators**

### 2. Domain Mapping
- **Primary:** AI in Cybersecurity
- **Secondary:** Threat Intelligence (actor/tooling attribution), XDR/SIEM (behavioural detection)

### 3. Current State of the Art
- Evidence about offensive AI comes mainly from **provider-side** visibility: account and prompt monitoring that led to Anthropic's GTG-1002 disclosure and Google GTIG's malware-family reporting **[E14, E15]**.
- Victim-side detection uses conventional behavioural analytics tuned on human-operated intrusions (dwell, working hours, command cadence) and scripted frameworks. Research agents such as PentestGPT-style systems measure offensive *capability*, not defender-visible *signatures*.

### 4. The Live Industry Pain Point
- Anthropic reports that in a state-sponsored campaign (detected mid-September 2025) an agentic framework built around Claude Code and MCP tools executed an estimated 80-90% of tactical operations against about 30 targets, using role-play to jailbreak and no custom malware (**provider-reported, secondary coverage**) **[E15, S]**.
- GTIG documents malware querying LLMs at runtime (PROMPTFLUX, PROMPTSTEAL) and FruitShell with prompts aimed at LLM-based analysers **[E14, S]**; CrowdStrike reports AI-enabled adversary operations up 89% **[E02, S]**.
- Mandiant explicitly cautions that 2025 was *not* the year breaches were the direct result of AI **[E01, P]**. Defenders therefore cannot tell how much of their intrusion load is agent-driven, and cannot tune controls to it.

### 5. The Academic/Technical Gap
- No public labelled dataset of agent-driven intrusion telemetry was found; behavioural features for agentic operators are uncharacterised.
- Existing attribution assumes human-operator behaviour. An LLM agent can be rate-limited, jittered or instructed to "act human," so naive timing features are brittle.
- **Why non-trivial:** (a) generating training data requires running offensive agents, which is dual-use and must be contained; (b) signatures depend on model, scaffold, tool set and prompt; (c) benign automation (scanners, Ansible, SOAR) shares features with agents; (d) hallucination artefacts (probing non-existent hosts, accounts or paths) may be the strongest signal but are model-dependent.

### 6. Core Research Question
Do LLM-orchestrated intrusion traces exhibit statistically separable features - inter-command latency distributions, command-syntax regularity, error-recovery loops, enumeration breadth-versus-depth, and a **phantom-entity rate** (attempts against non-existent hosts/users/files) - that distinguish them from human red-team and deterministic automation traces, **generalise across unseen scaffolds and models**, and **remain separable when the operator applies humanisation countermeasures**?

### 7. Concrete Evaluation Methodology
1. **Contained range.** An isolated AD plus cloud range with no external egress and full telemetry (Sysmon/auditd, Zeek, identity logs, cloud audit logs). Objectives are benign capture-the-flag style goals.
2. **Operator classes.** (a) Human operators: use existing public human-operated datasets where possible; if new human sessions are collected, obtain ethics approval and consent. (b) Deterministic automation: Atomic Red Team and Caldera plans, vulnerability scanners, IT automation. (c) LLM agents: at least three open-weight and, where the provider's acceptable-use policy permits security research, API models across at least three scaffolds, run only inside the contained range.
3. **Features.** Timing distributions, n-gram and entropy statistics of command streams, process lineage graphs, authentication patterns, tool-call burst structure, phantom-entity rate, retry/backoff patterns.
4. **Models.** Interpretable baselines (logistic regression, GBM) and a sequence model; **open-set** detection for unseen scaffolds.
5. **Generalisation tests.** Leave-one-scaffold-out and leave-one-model-out; temporal holdout as models update.
6. **Adaptive operator.** Instruct agents to humanise (jitter, rate limit, avoid error loops) and measure feature drift; optimise countermeasures against the classifier (black-box).
7. **Operational metrics.** Minimum observation window (commands needed to reach 95% TPR at 1% FPR), false-positive rate on benign automation, feature-importance stability.
8. **Cross-check.** Compare derived signatures qualitatively against behaviours described in public provider reports (GTG-1002, GTIG) to test plausibility without claiming validation.
9. **Release policy.** Publish features and detectors; withhold agent prompts and operational scaffolding.

### 8. Feasible Datasets & Lab Telemetry Sources
- **Lab-generated** agent traces (primary contribution; none found publicly).
- **Human-operated baselines:** OTRF Security-Datasets, Splunk `attack_data`, DARPA OpTC red-team portion *(verify)*.
- **Deterministic automation baselines:** Atomic Red Team, MITRE Caldera, Splunk Attack Range, GOAD, CloudGoat.
- **Agent scaffolds / capability references:** PentestGPT, Cybench *(verify)*, CyberSOCEval for context **[A13]**.
- **Provider disclosures** for qualitative cross-checks **[E14, E15]**.
- **Evidence anchors:** E01, E02, E14, E15, A10, A13.

---

## PS-07 - Extended Detection & Response (XDR)

### 1. Title
**Partial-Observability Incident Graph Fusion for XDR: Probabilistic Provenance Correlation Across Identity, Endpoint and Edge Telemetry with Node-Level Attribution under Session-Token Replay and LOLBin Chains**

### 2. Domain Mapping
- **Primary:** XDR
- **Secondary:** SIEM (normalisation and correlation), Digital Forensics (provenance), Identity security (token theft)

### 3. Current State of the Art
- Commercial XDR correlates alerts across endpoint, identity, email, cloud and network using rules and ML; Open XDR approaches normalise to schemas such as OCSF/ECS and trade depth for breadth.
- Academic provenance-based IDS (PIDS) learn from system audit graphs with GNN or sequence models; Bilot et al. re-implemented eight state-of-the-art systems and open-sourced the framework and pre-processed datasets **[A01]**; Cai et al. built a more realistic dataset **[A02]**.

### 4. The Live Industry Pain Point
- Edge/VPN/router devices "typically lack standard EDR telemetry" and are among the most targeted technologies **[E01, P]**; attackers harvest "long-lived OAuth tokens and session cookies" and "interactive social engineering frequently bypasses traditional MFA" **[E01, P]**.
- Identity-based attacks rose 32% in 1H-2025 and are the dominant theme of Red Canary's 2026 report (a 53% share appeared in a garbled source - **low confidence**) **[E03, E04, S]**; ~82% of detections are malware-free (secondary) **[E02]**.
- Independent tests of commercial EDR/XDR report partial cross-host correlation and command-line visibility gaps **[E21, S/V]**.

### 5. The Academic/Technical Gap
- PIDS results do not translate: a simple neural network matches state of the art on five of seven DARPA datasets, alerts span tens of thousands of nodes/events, and node-level attribution is weak **[A01]**. Existing datasets are dated, short and have incomplete attack chains **[A02]**.
- No peer-reviewed study measures cross-domain correlation correctness under OCSF normalisation (information loss, entity-resolution error) with **missing telemetry planes**.
- **Why non-trivial:** (a) the user-session-device-process join is probabilistic and the errors compound; (b) edge devices produce no host provenance, so the graph has latent hops; (c) normalisation maps heterogeneous semantics into common fields, losing meaning; (d) adversaries tamper with or repurpose the sensors (EvilEDR **[A03]**); (e) scale - graph construction on 1e6-1e8 events with bounded latency.

### 6. Core Research Question
Can a probabilistic provenance-graph model that explicitly represents **unobserved (latent) nodes** and **cross-domain entity-resolution uncertainty** yield incident subgraphs with higher node-level attribution F1 and a pre-registered alert-compression ratio (hypothesis: >=20x) than (i) rule-based SIEM/XDR correlation and (ii) the PIDS baselines from A01, when whole telemetry planes (edge devices, parts of endpoint coverage) are missing at rate m, and does the advantage persist against session-token replay and LOLBin chains?

### 7. Concrete Evaluation Methodology
1. **Data assembly.** Use DARPA TC/OpTC and the NDSS-2025 dataset for host provenance **[A02]**; add an identity and edge layer in a lab: an open-source IdP (e.g., Keycloak) with OIDC sessions, a virtual VPN/firewall appliance emitting syslog/flow, and endpoint telemetry from an AD range.
2. **Normalisation.** Implement an OCSF mapping pipeline; quantify field-level information loss and entity-resolution error rates per source.
3. **Attack scenarios (lab, benign payloads).** Session-token replay from a different host/network; help-desk-style credential reset followed by access; LOLBin chains executed with built-in admin tools; edge-pivot where the first hop is an appliance with no EDR.
4. **Baselines.** Re-run the open-sourced PIDS framework from A01; Sigma-based correlation in a SIEM; simple entity-linking heuristics.
5. **Proposed method.** Factor-graph/Bayesian entity resolution plus provenance graph inference with latent nodes (MAP/variational), followed by attack-story extraction with calibrated per-node confidence.
6. **Missingness experiments.** Randomly remove telemetry planes and endpoint coverage under MCAR and MNAR schemes (e.g., drop edge entirely; drop endpoints with a specific OS); measure degradation curves.
7. **Metrics.** Node/edge precision, recall and F1; subgraph size distribution; alert-compression ratio; entity-resolution accuracy; calibration error; runtime and memory vs event volume.
8. **Adversarial tests.** Sensor tampering and EDR-repurposing scenarios **[A03]**, spoofed IP/user-agent during token replay, delayed multi-stage chains.
9. **Threats to validity.** Lab identity layer is synthetic; DARPA data age; label provenance.

### 8. Feasible Datasets & Lab Telemetry Sources
- **DARPA Transparent Computing (E3/E5)** and **DARPA OpTC** host provenance; the **NDSS 2025 PIDS dataset** and the **open-sourced framework/datasets from Bilot et al.** *(verify release locations)* **[A01, A02]**.
- **LANL Comprehensive Multi-Source Cyber-Security Events** (authentication, process, flows, DNS, red-team labels); **CICIDS2017** for network context.
- **OTRF Security-Datasets**, **Splunk `attack_data`**, **EVTX-ATTACK-SAMPLES** for endpoint technique telemetry.
- **OCSF schema** definitions for normalisation.
- **Lab telemetry:** Keycloak OIDC logs, virtual appliance syslog/NetFlow, Sysmon/auditd, Zeek.
- **Evidence anchors:** E01, E02, E03, E04, E21, A01, A02, A03.

---

## PS-08 - Continuous Threat Exposure Management (CTEM)

### 1. Title
**Identity-Aware Multi-Cloud Attack-Path Prioritisation with Safe, Non-Destructive Exposure Validation: Sub-Quadratic Choke-Point Analysis Under Resource Churn**

### 2. Domain Mapping
- **Primary:** CTEM
- **Secondary:** Vulnerability Management (exposure inputs), Cloud/Identity security (IAM, AD CS, OAuth), SIEM (detection coverage of paths)

### 3. Current State of the Art
- CTEM cycles scoping, discovery, prioritisation, validation and mobilisation. Attack-path tools build graphs over AD and cloud identities (BloodHound/OpenGraph-style) and CNAPP platforms; adversarial exposure validation (BAS/AEV) emulates techniques. A hypergraph IAM approach claims sublinear traversal on multi-cloud privilege analysis **[A19]**.

### 4. The Live Industry Pain Point
- Exposures that mattered in 2025 were configuration/identity chains: misconfigured AD CS templates creating admin accounts that bypass password rotation, backup objects deleted from cloud storage, long-lived OAuth tokens, Tier-0 hypervisors and edge devices without EDR **[E01, P]**. These are not CVE counts.
- Gartner's 2026 guidance projects 60% of organisations adopting structured exposure validation by 2029 **[E20, S/V]**, yet its 2022 claim that CTEM adopters would suffer far fewer breaches has no published empirical validation **[E20, S/V]**.
- About 29% of KEVs show exploitation evidence on or before disclosure **[E10, S]**, so CTEM must prioritise before enrichment data exists.

### 5. The Academic/Technical Gap
- Attack graphs can grow exponentially in hosts and vulnerabilities; scalable attack-path analysis remains an open problem **[A20]**; the hypergraph paper's complexity bounds are self-reported **[A19]**.
- Simulation fidelity is "an open question" and BAS tools show gaps in initial-access emulation and update cadence **[A21]**. No peer-reviewed measurement of CTEM efficacy or of BAS predictive validity was found.
- **Why non-trivial:** (a) provider IAM semantics differ (AWS policies/SCPs, Entra roles/consent, GCP bindings, AD ACLs, K8s RBAC); (b) ephemeral resources invalidate graphs within minutes; (c) *validated exposure* is undefined: probing in production must be non-destructive, yet fidelity depends on realism; (d) path counts explode while remediation capacity is fixed; (e) ground truth of "paths attackers really use" is scarce.

### 6. Core Research Question
Can a unified multi-cloud identity-network-vulnerability exposure graph with **probabilistic edge exploitability** (updated by safe validation probes and threat-intelligence priors) identify a minimal remediation (choke-point) set that removes at least a pre-registered fraction (hypothesis: >=80%) of attacker-feasible paths to designated crown jewels, scale with a measured exponent below 2 to ~1e6 nodes with incremental updates under churn, and show **predictive validity** - i.e., high-ranked paths match technique sequences observed in real incidents and in red-team successes better than CVSS/EPSS-only prioritisation?

### 7. Concrete Evaluation Methodology
1. **Environments.** Terraform-defined vulnerable-by-design multi-cloud labs (AWS, Azure, GCP) plus an on-premises AD domain with hybrid federation; synthetic graph generators (calibrated to real degree/role distributions) scaled to 1e6 nodes.
2. **Ingestion and schema.** Collect IAM policies, role-trust and federation, SCPs/org policies, network ACLs/security groups, K8s RBAC, AD ACLs (BloodHound-style collectors), vulnerability findings (scanner outputs), exposure scans; unify in a typed property graph.
3. **Algorithms compared.** BFS/Dijkstra shortest paths (BloodHound-like), reproduction of the hypergraph method **[A19]**, min-cut/max-flow choke points, approximate betweenness, probabilistic reachability, and an incremental/event-driven update algorithm consuming audit-log events.
4. **Safe validation.** Non-destructive proof-of-reachability probes (read-only API calls, canary resources) plus technique emulation limited to the lab (cloud technique emulation toolkits; Atomic Red Team; Caldera). Define a calibrated *validated-exposure* score as P(path feasible | probe outcome, controls).
5. **Predictive validity.** Compare ranked paths against (i) documented real-incident technique chains from public reports and (ii) red-team results; use rank-correlation and top-k hit rate against CVSS/EPSS-only prioritisation.
6. **Detection coupling.** For each validated path, test whether the SIEM/XDR detects the traversal (links to PS-09).
7. **Metrics.** Paths eliminated per fix, size of remediation set, runtime and memory scaling exponent, incremental-update latency under churn (e.g., 1e3 resource changes/min), validation precision/recall, false-negative rate on known-vulnerable paths.
8. **Ablations.** With/without validation; with/without threat-intelligence priors (KEV/EPSS); with/without detection context.

### 8. Feasible Datasets & Lab Telemetry Sources
- **CloudGoat** (AWS), **AzureGoat** and **GCPGoat** (verify current maintenance), **GOAD** (AD), plus Terraform-built hybrid labs.
- **Stratus Red Team**-style cloud attack emulation; **Atomic Red Team**; **MITRE Caldera**; **MITRE ATT&CK Cloud** matrix.
- **BloodHound** collectors and sample datasets; **flaws.cloud / flaws2.cloud** challenge environments and any published CloudTrail logs *(verify)*.
- **Scanner outputs:** Trivy, Grype; **KEV**, **EPSS** for threat-intel priors.
- **Public incident write-ups** for chain validation (e.g., Mandiant M-Trends 2026 AD CS and backup-deletion patterns) **[E01]**.
- **Synthetic graph generator** (to build) for scale tests.
- **Evidence anchors:** E01, E10, E20, A19, A20, A21.

---

## PS-09 - SIEM

### 1. Title
**Execution-Validated Detection Coverage: Rule-Liveness Verification via Data-Lineage Analysis and Technique-Variant Replay, with Outcome-Grounded Evaluation of LLM-Generated Detection Rules**

### 2. Domain Mapping
- **Primary:** SIEM
- **Secondary:** AI (LLM-assisted detection engineering), Threat Intelligence (technique and variant intake), XDR (data-source dependence)

### 3. Current State of the Art
- Detection content is written in Sigma/KQL/SPL/EQL and tagged to MITRE ATT&CK; coverage is reported as the share of techniques with at least one tagged rule. Detection-as-code pipelines test syntax and sometimes unit-test rules with sample events.
- LLM rule generation is an active area: LLMCloudHunter (99.18% of generated candidates compiled to Splunk queries), EvoSIEM (85% syntactically valid Sigma rules), RulePilot (executes rules in Splunk) and RuleGenie (rule-set redundancy) **[A24]**.

### 4. The Live Industry Pain Point
- CardinalOps (2025): average SIEM coverage is **21%** of ATT&CK techniques while 13% of rules are **non-functional** (misconfigured sources, missing fields); SIEMs ingest an average of 259 log types and ~24,000 unique sources, enough to detect more than 90% of techniques - the authors blame manual rule creation and brittle pipelines, not missing data **[E08, S/V]**.
- SANS 2025: 73% rank false positives as the top detection challenge; 42% send all data to the SIEM without a strategy (press summary) **[E09, S/V]**.
- Mandiant: 90-day retention and reliance on static IOCs leave blind spots; edge devices provide no EDR telemetry **[E01, P]**.

### 5. The Academic/Technical Gap
- Coverage numbers are **mapping-based**, not **execution-based**; no peer-reviewed measurement of rule liveness or execution-validated ATT&CK coverage was found **[academic_gaps/09]**.
- LLM-rule papers mostly report syntax validity, compile success or text similarity; a rule that parses and resembles a reference can still never fire, or fire constantly on benign activity **[A24]**. Learned log-anomaly detectors suffer leakage-prone evaluation **[A23]**.
- **Why non-trivial:** (a) a rule's liveness depends on schema drift, parser versions, log-source health and field-value domains that change continuously; (b) technique "variants" (obfuscation, binary substitutions) are an open set; (c) benign background is enterprise-specific, so FP rate is not portable; (d) coverage must be decomposed (missing source vs broken rule vs missing rule vs fragile rule) to be actionable.

### 6. Core Research Question
Can data-lineage-aware static analysis (rule field dependencies x observed ingestion schema x log-source health) predict rule liveness with precision above a pre-registered bar (hypothesis: >=0.95 at recall >=0.9 versus injected-event ground truth), and can **execution-validated** coverage (atomic technique emulation + variant mutation + benign replay) explain the gap between the reported ~21% mapping-based coverage and the >90% data-supported potential - and rank LLM-generated rules by detection outcome rather than text similarity?

### 7. Concrete Evaluation Methodology
1. **Reproducible testbed.** An open-source SIEM stack (e.g., Elastic or Wazuh) plus a Splunk Attack Range deployment for comparison; Windows/Linux endpoints, Active Directory, cloud audit logs.
2. **Rule corpora.** SigmaHQ, Elastic detection-rules, Splunk security_content, Microsoft Sentinel content; convert across backends with a rule-conversion toolchain.
3. **Liveness analyser.** Build field-dependency graphs; join with schema observed in live ingestion (field presence/cardinality over time, parse-failure rates, ingestion lag). Validate predictions by **minimal-event injection**: synthesise the smallest event satisfying each rule, inject, and record whether it fires.
4. **Execution-based coverage.** For each ATT&CK technique with a test case, run atomic tests plus automated **variant mutators** (command-line obfuscation, alternate binaries, path and case variants, parent-process changes); record detection rate, time-to-detect and alert quality.
5. **Benign replay.** Replay benign enterprise-like telemetry (lab user-activity generators and public benign logs) to estimate per-rule alert rate; report FP per 1k events with CIs.
6. **LLM rule evaluation.** Generate rules from CTI and ATT&CK descriptions with 2-3 methods; score on a staged ladder: syntax valid -> field-valid -> fires on true-positive replay -> FP on benign -> variant robustness. Compute correlation between text-similarity and execution outcomes; hypothesis: weak correlation.
7. **Gap decomposition.** Attribute uncovered techniques to (i) no data source, (ii) data present but rule dead, (iii) no rule, (iv) rule fragile to variants, using Shapley or sequential-ablation attribution. Compare the resulting decomposition with the 21% vs >90% claim.
8. **Threats to validity.** Lab telemetry realism, rule-corpus bias toward Windows, backend conversion artefacts.

### 8. Feasible Datasets & Lab Telemetry Sources
- **SigmaHQ/sigma**, **Elastic detection-rules**, **Splunk security_content**, **Azure Sentinel GitHub content**.
- **Atomic Red Team**, **EVTX-ATTACK-SAMPLES**, **OTRF Security-Datasets (Mordor)**, **Splunk `attack_data`**, **Splunk Boss of the SOC (BOTS) datasets** for replay.
- **MITRE ATT&CK Evaluations** public results for cross-vendor comparison.
- **Aggregate priors** from CardinalOps and SANS reports (not raw data) **[E08, E09]**.
- **Lab telemetry:** Sysmon/Windows Event Logs, auditd, cloud audit logs, Zeek; benign activity generators.
- **Evidence anchors:** E01, E08, E09, A23, A24.

---

## PS-10 - Defender AI

### 1. Title
**Prompt-Injection-Resilient SOC Agents: Provenance-Tracked Taint and Capability Gating for Attacker-Influenced Telemetry, Evaluated Under Adaptive Adversaries and Operational Utility Constraints**

### 2. Domain Mapping
- **Primary:** Defender AI
- **Secondary:** SIEM (log-to-LLM data path), AI in Cybersecurity (adaptive attack evaluation), Incident Response (agent actions)

### 3. Current State of the Art
- SOC copilots and agents summarise alerts, query SIEMs, enrich entities, write tickets and, increasingly, trigger containment. Prompt-injection defenses include classifiers (e.g., Microsoft's XPIA), spotlighting/delimiting, prompt sandwiching, filters and secret-based detectors. The EchoLeak case study proposes prompt partitioning and provenance-based access control **[E16]**.
- Capability benchmarks: CyberSOCEval, ExCyTIn-Bench, Simbian **[A13, A14]**.

### 4. The Live Industry Pain Point
- **EchoLeak (CVE-2025-32711):** a zero-click prompt injection in Microsoft 365 Copilot chained bypasses of the XPIA classifier and a CSP-allowed Teams proxy for data exfiltration (CVSS 9.3 per The Hacker News; no known in-the-wild exploitation) **[E16, S]**.
- **Log-borne injection:** DEF CON 34 poster work tests which attacker-controlled fields survive SIEM ingestion and steer downstream agents; a LevelBlue SpiderLabs post shows agents altering, hiding or inventing events; a vendor blog relays a 2025 Sygnia test in which an injected PowerShell block made an MDR summary describe a Mimikatz run as maintenance **(unverified)** (see `industry_telemetry/10_defender_ai.txt`).
- Adversaries are already shipping prompts aimed at defender LLMs (FruitShell) **[E14, S]**.

### 5. The Academic/Technical Gap
- "Poisoning the Watchtower": across 48 strategy x defense x task combinations with gpt-4o-mini, persona hijacks still suppressed many malicious logs; the best defense cut average injection success from ~26.6% to ~11.8% (non-zero residual); summarisation was riskiest **[A11]** (single small model; preprint).
- "The Attacker Moves Second": 12 published defenses were broken, most at >90% success, by adaptive attacks (gradient, RL, search, human) **[A10]**.
- No capability- or information-flow-based guarantee for SOC tool chains was found, and the **utility cost** of defenses on realistic investigation tasks is unmeasured.
- **Why non-trivial:** logs are attacker-influenced yet must be read semantically; sanitising can destroy evidence; models do not separate instructions from data; agents hold privileges (containment, rule edits, case memory) that turn text manipulation into action; evaluating against adaptive attackers is expensive.

### 6. Core Research Question
Can information-flow control for LLM agents - labelling every telemetry-derived token with a trust level, separating a privileged planner from a quarantined reader that emits only schema-constrained outputs, and gating high-impact tools behind capability tokens - **bound** the integrity damage of attacker-controlled log content to a defined class (hypothesis: attacker text may influence free-text summaries but cannot trigger containment, suppress an alert, or exfiltrate case data), with adaptive-attack success below a pre-registered bound (hypothesis: <=5% at a fixed query budget) while retaining at least 90% of baseline investigation utility?

### 7. Concrete Evaluation Methodology
1. **Reference agent.** Build an open SOC-agent pipeline with tools: SIEM query (read), enrichment (read), ticket write, simulated containment, case memory. Use at least one open-weight and one API model.
2. **Threat model.** Attacker controls specified fields that survive real SIEM normalisation: user-agent, URL path/query, DNS qname, HTTP headers, process command line, email subject/body, filenames, cloud resource tags. Goals: **suppression** (induce false negatives), **fabrication**, **exfiltration** (markdown/link channel as in EchoLeak), **tool misuse** (wrong host/containment/rule disable), **memory poisoning** across cases.
3. **Payload library.** From attack classes in the literature (A11) plus new classes; measure field-length and encoding survival through ingestion.
4. **Defenses compared.** None; classifier filter; spotlighting; sandwiching; dual-model privilege separation; taint tracking plus capability gating; combinations.
5. **Adaptive evaluation.** Per A10: white-box gradient attacks on open-weight models, RL and evolutionary search, and human red-teaming with fixed query budgets; report attack success vs budget curves, not point estimates.
6. **Utility.** Investigation success on ExCyTIn-Bench-style tasks, CyberSOCEval tasks, triage F1 on labelled alerts, and added latency/cost.
7. **Formal check.** Define integrity properties (non-interference variants) over the planner's tool-call graph; verify by static analysis and property-based fuzzing; report which properties hold and which attacks remain.
8. **Pareto analysis.** Plot security (worst-case adaptive ASR) against utility retention for each defense; identify dominated designs.
9. **Threats to validity.** Model dependence, benchmark realism, attacker budget assumptions; replicate on at least two model families.

### 8. Feasible Datasets & Lab Telemetry Sources
- **Capability benchmarks:** ExCyTIn-Bench (Microsoft), CyberSOCEval (Meta), Simbian Cyber Defense Benchmark *(verify access)* **[A13, A14]**.
- **Injection evaluation references:** AgentDojo-style agent injection suites *(verify)*; the "Poisoning the Watchtower" corpus if released *(verify)* **[A11]**.
- **Base telemetry:** Splunk BOTS datasets, OTRF Security-Datasets, Splunk `attack_data`, EVTX-ATTACK-SAMPLES.
- **Lab telemetry:** a SIEM instance with controlled ingestion pipelines to measure which injected field values survive normalisation; synthetic case-management backend.
- **Adaptive attack tooling:** open-weight models for white-box optimisation; internal red-team protocol.
- **Evidence anchors:** E14, E16, A10, A11, A12, A13, A14.

---

## Shared infrastructure and recommended execution order

| Layer | Used by | Notes |
|---|---|---|
| Isolated AD + cloud + hypervisor range with full telemetry | PS-02, 03, 05, 06, 07, 08, 09 | Build once (Terraform/Ansible); snapshot/rollback; no external egress |
| Telemetry normalisation (OCSF) and storage | PS-02, 06, 07, 09, 10 | Reuse the same pipeline to compare conclusions |
| Adaptive-adversary harness | PS-01, 02, 05, 06, 10 | Black-box search + human red-team protocol |
| Temporal-validity library (as-of-date snapshots, rolling-origin splits) | PS-01, 04, 05 | Prevents the leakage failures seen in EPSS and malware ML |

**Suggested order.** (1) PS-09 and PS-04 - mostly public data, quick validation of the "evaluation is the gap" theme. (2) PS-01 and PS-02 - need longitudinal collection, start the clock early. (3) PS-05, PS-07, PS-10 - build on the lab range. (4) PS-03, PS-06, PS-08 - heaviest lab and ethics/safety overhead.

## Verification checklist before publishing any claim

1. Re-read primary reports once network access to `www.crowdstrike.com`, `www.verizon.com`, `www.zscaler.com` and `arxiv.org` is allowed; upgrade ledger rows from S/A to P.
2. Confirm paper venues (the CCS 2025 and CCS 2026 labels came from search listings).
3. Check every dataset's current availability and licence (items marked *verify*).
4. Replace each "hypothesis" threshold with a pre-registered value after a pilot.
5. Obtain institutional/ethics approval for PS-05 real-sample analysis and PS-06 human or agent data collection.
