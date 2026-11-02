# Phase 3 - Technical Gap Analysis Matrix

Derived only from entries in `extracted_evidence/evidence_ledger.md` (IDs in brackets).
Evidence levels: **P** primary read, **S** secondary, **A** abstract-only, **V** vendor. See the ledger for caveats; nothing here should be cited as verified beyond its level.

## 1. Telemetry silos and signal-to-noise flaws

| # | Where correlation or detection logic breaks | Mechanism | Evidence | Domains |
|---|---|---|---|---|
| T1 | **Sensor-less planes.** Edge/VPN/router appliances, hypervisor management planes and SaaS carry the attacker activity but have no EDR-class telemetry. | Correlation engines can only join what is observed; unobserved hops become unexplained gaps in the incident graph. | [E01 P] edge devices "typically lack standard EDR telemetry"; [E17 S] guidance on limited device logging; [E18 S/V] ESXi logging decentralized | XDR, Forensics, IR |
| T2 | **Identity-to-endpoint join failure.** A stolen OAuth token or session cookie yields a valid IdP event and no endpoint artifact. | Entity resolution across user/session/device/process is probabilistic; vendors treat it as deterministic. | [E01 P] long-lived OAuth tokens/session cookies harvested; [E03 S] identity attacks +32%; [E04 S, low] identity-heavy detections | XDR, SIEM |
| T3 | **Coverage illusion in SIEM.** Rules are counted by ATT&CK tag, not by execution. | 13% of rules cannot fire (missing fields/sources); average technique coverage 21% while data would support >90%. | [E08 S/V] | SIEM, CTEM |
| T4 | **Alert fatigue as a rule-quality symptom.** 73% rank false positives as top challenge, yet coverage is low. | Unvalidated rules are simultaneously noisy and blind; no execution-based tuning loop. | [E09 S/V], [E08] | SIEM, IR |
| T5 | **Provenance alerts too coarse.** PIDS emit alerts spanning tens of thousands of nodes with weak node-level attribution. | Anomaly scores at graph level do not convert to analyst-consumable stories. | [A01 A] | XDR, IR |
| T6 | **Indicator feed fragmentation.** Provider overlap <=4.5%; no per-actor decay model. | Aggregation without survival modelling yields stale matches and duplicate effort. | [A16 A], [A17 A], [E01 P] static IOCs insufficient | Threat Intel, SIEM |
| T7 | **Retention horizon.** 90-day default retention vs 14-day median and 122-day espionage dwell. | Evidence of initial access ages out before discovery. | [E01 P] | Forensics, IR, SIEM |

## 2. Adversarial defeating mechanisms

| # | Mechanism | How it defeats the control | Evidence | Domains |
|---|---|---|---|---|
| D1 | **Intermittent / partial encryption** (head-only, strided, percentage patterns; entropy shaping). | Whole-file entropy and fixed-window histogram detectors lose power; speed advantage reduces dwell-in-encryption. | [A04 A], [A05 A] | Malware/Ransomware, XDR |
| D2 | **EDR killers / BYOVD / driverless sensor blinding.** ~90 tools; 54 BYOVD over 35 drivers. | Removes the sensor before encryption; endpoint detection never sees the payload stage. | [E13 S], [A03 A] (EDR repurposing) | Malware/Ransomware, XDR |
| D3 | **Malware-free tradecraft / LOLBins / valid accounts.** ~82% of detections malware-free (secondary); 56% of Sophos cases "logged in". | No file to scan; behavior is indistinguishable from admin activity without context. | [E02 S], [E06 S], [E04 S low] | XDR, SIEM, Threat Intel |
| D4 | **Token / session theft and MFA-bypassing social engineering** (vishing 11% of M-Trends intrusions; help-desk targeting). | Authentication succeeds legitimately; MFA is bypassed through the human. | [E01 P], [E03 S] | XDR, IR |
| D5 | **Sub-minute hand-off.** 22 s to secondary actor; ~29 min average breakout (secondary). | Response playbooks sized for hours cannot act before hands-on-keyboard. | [E01 P], [E02 S] | IR, Threat Intel |
| D6 | **Pre-disclosure exploitation.** Mean time-to-exploit -7 days; ~29% of KEVs exploited on/before publication. | Patch-SLA programs start the clock too late. | [E01 P], [E10 S] | Vulnerability Mgmt, CTEM |
| D7 | **Hypervisor / backup / identity plane attacks.** Datastore encryption; backup deletion; AD CS abuse. | Defeats recovery and endpoint-centric controls simultaneously. | [E01 P] | Malware, Forensics, CTEM |
| D8 | **Prompt injection into defender AI** (log-field injection; zero-click EchoLeak chain) and malware with prompts aimed at LLM analysers. | Attacker-controlled data becomes instructions to the SOC agent; adaptive attacks break published defenses (>90% ASR on 12). | [E16 S], [A11 A], [A10 A], [E14 S] | Defender AI, SIEM, AI |
| D9 | **Adaptive adversary vs static evaluation.** Defenses and detectors scored against fixed attack sets. | Reported robustness collapses under adaptive attack; same bias for ML malware (temporal leakage). | [A10 A], [A06 A], [A23 A] | AI, Malware, SIEM |
| D10 | **Runtime-LLM malware** (PROMPTFLUX self-rewrite, PROMPTSTEAL command generation). | Signature and IOC persistence is weaker when code and commands are regenerated per execution. | [E01 P], [E14 S] | Threat Intel, Malware, AI |

## 3. Architectural scaling failures (dynamic, ephemeral, multi-cloud)

| # | Failure | Why it breaks at scale | Evidence | Domains |
|---|---|---|---|---|
| S1 | **Attack-graph blow-up.** Exponential growth in hosts x vulns; scalable attack-path analysis remains open. | CTEM path analysis cannot be recomputed on every ephemeral change. | [A20 A]; hypergraph claim O(sqrt n) is self-reported [A19 A] | CTEM |
| S2 | **Multi-cloud IAM semantics.** Traditional graphs cannot efficiently represent multi-party IAM relationships across providers. | Federation/trust semantics differ (AWS/Azure/GCP/AD); exposure graphs diverge from reality. | [A19 A], [E01 P] AD CS + cloud mis-trust | CTEM, XDR |
| S3 | **Ephemeral workload evidence loss.** Containers/functions terminate before collection; provider logs incomplete/disabled by default. | Forensics assumes persistent hosts. | [E19 V] | Forensics, IR |
| S4 | **Validation cannot run in production safely, and fidelity is unquantified.** Simulation realism is "an open question"; tool gaps in initial access. | Exposure validation (CTEM stage 4) is the scarcest capability and the least measured. | [A21 A], [E20 S/V] | CTEM |
| S5 | **Vulnerability data supply chain.** NVD enrichment backlog (tens of thousands) and a risk-based triage from Apr 2026; EPSS missing for >22% of KEV CVEs pre-listing. | Pipelines that depend on enrichment inherit delay and gaps. | [E12 S], [A07 A] | Vulnerability Mgmt |
| S6 | **Learned-policy brittleness.** RL defenders degrade with topology/attacker shift; LLM agents scored on narrow, synthetic tasks. | Policies do not transfer from simulator to production; benchmarks ignore operational outcomes. | [A22 A], [A14 A/S], [A13 A] | IR, AI, Defender AI |
| S7 | **Log volume without strategy.** 259 log types / ~24,000 sources per SIEM; 42% send all data without strategy. | Cost and parse-failure surface scale faster than detection engineering capacity. | [E08 S/V], [E09 S/V] | SIEM |

## 4. Cross-cutting synthesis (what the matrix implies)

1. **Validation, not collection, is the common bottleneck.** SIEM rules, CTEM exposures, XDR correlations, CTI indicators and defender-AI robustness are all deployed on the strength of unexecuted claims (mapping-based coverage, retrospective EPSS, static injection tests, non-adaptive ransomware detectors).
2. **Time is the dominant new variable.** Hand-off (22 s), breakout (~29 min), time-to-exploit (-7 d) and IOC decay all shrink the useful decision window; existing workflows are calibrated to hours or days.
3. **Attackers move down-stack and sideways.** Hypervisors, backups, edge devices, tokens and defender AI sit where telemetry, forensics and evaluation methodology are weakest.
4. **Evaluation methodology is itself a gap.** Temporal leakage (EPSS, malware ML, log anomaly), static attacks (prompt injection), text-similarity scoring (LLM rule generation) and simulator-only validation recur across domains.

Mapping of gaps to problem statements: T6/D5/D10 -> PS-01, PS-02; T7/S3/D7 -> PS-03; D6/S5 -> PS-04; D1/D2/D9 -> PS-05; D3/D10/AI-prevalence -> PS-06; T1/T2/T5/D3/D4 -> PS-07; S1/S2/S4 -> PS-08; T3/T4/S7 -> PS-09; D8/D9/S6 -> PS-10.
