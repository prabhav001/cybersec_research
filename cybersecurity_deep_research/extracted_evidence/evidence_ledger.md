# Evidence Ledger

Every claim used in `problem_statements.md` and `gap_analysis_matrix.md` traces back to an ID here.
Logged 2026-11-02.

**Verification levels**

| Code | Meaning |
|---|---|
| **P** | Primary page fetched and read this session (only the M-Trends 2026 blog on cloud.google.com) |
| **S** | Secondary: press/blog coverage or search-engine summary. Primary NOT opened |
| **A** | Academic abstract/snippet via search results. Full paper NOT read |
| **V** | Vendor-authored or vendor-commissioned (treat as advocacy) |

Numbers marked S, A or V should be re-verified against the primary document before they go into a publication.
Where secondary sources disagreed, the conflict is recorded rather than resolved by guessing.

## Industry telemetry

| ID | Source | Key claims used | Level | URL |
|---|---|---|---|---|
| E01 | Mandiant M-Trends 2026 (Google Cloud) | Median dwell 14 d (from 11); 122 d for espionage/DPRK IT workers; exploits 32% of intrusions (6th year), voice phishing 11%, prior compromise 10% (30% in ransomware, 15% in 2024), email phishing 6%; internal detection 52% (43%); estimated mean time-to-exploit -7 d; hand-off >8 h (2022) -> 22 s (2025); edge devices lack EDR telemetry and limited forensic artifacts; BRICKSTORM ~400 d dwell, survives reboot; hypervisors "Tier-0"; backup/identity/virtualization targeted by Akira/Qilin; AD CS misconfig; OAuth token/session cookie harvesting; PROMPTFLUX/PROMPTSTEAL query LLMs at runtime; 90-day log retention blind spot; "not the year breaches were the direct result of AI" | **P** | https://cloud.google.com/blog/topics/threat-intelligence/m-trends-2026 |
| E02 | CrowdStrike 2026 Global Threat Report | Avg breakout ~29 min, fastest 27 s; AI-enabled adversary ops +89%; 280+ adversaries; ~82% malware-free detections; ~42% vulns exploited pre-disclosure | S | https://www.businesswire.com/news/home/20260224017260/en/ ; https://www.infotrust.com.au/resource-library/key-findings-crowdstrike-2026-global-threat-report |
| E03 | Microsoft Digital Defense Report 2025 | Identity attacks +32% (1H25); >97% password-based; phishing-resistant MFA prevents up to 99%; AI phishing 54% vs 12% click-through (4.5x; one outlet says 3x - **conflict**); >half attacks ransomware/extortion; ~82% ransomware w/ data theft; >40% hybrid; +87% Azure-targeting campaigns (single source); token-theft/AiTM <3% (single source) | S | https://www.helpnetsecurity.com/?p=346007 ; https://www.paubox.com/blog/microsoft-ai-makes-phishing-4.5x-more-effective-and-far-more-profitable |
| E04 | Red Canary 2026 Threat Detection Report | Identity theme; "53%" figure seen only in a garbled PDF snippet (**low confidence**); 2025 edition: Cloud Accounts #1 | S | https://www.zscaler.com/resources/industry-reports/red-canary-2026-threat-detection-report-executive-summary.pdf |
| E05 | Verizon 2026 DBIR | Exploitation became leading initial-access vector (~31%, from ~20%), overtaking credentials; 22k+ breaches | S | https://www.helpnetsecurity.com/2026/05/25/lessons-from-verizon-dbir-2026-findings/ |
| E06 | Sophos Active Adversary Report 2025 | Median dwell 2 d (IR ransomware 4 / non-ransomware 11.5; MDR 3 / 1); 56% logged in; 11 h to AD attempt; ~73 h to exfil | S | https://www.sophos.com/en-us/blog/2025-sophos-active-adversary-report |
| E07 | Coveware Q3 2025 | 23% paid; median ~$140k; ~76% exfil; exfil-only pay rate 19% | S | https://www.securityweek.com/ransomware-payments-dropped-in-q3-2025-analysis/ |
| E08 | CardinalOps State of SIEM Detection Risk 2025 | 21% ATT&CK technique coverage; 13% rules non-functional; 259 log types, ~24,000 sources; data sufficient for >90% | S/V | https://www.helpnetsecurity.com/2025/06/09/siem-detection-coverage/ |
| E09 | SANS 2025 Detection & Response survey | 73% false positives top challenge; very-frequent FPs 13%->20%; 42% send everything to SIEM (press summary); "66% can't keep pace" attributed only by a vendor glossary (**unconfirmed**) | S/V | https://www.stamus-networks.com/blog/what-the-2025-sans-detection-response-survey-reveals-false-positives-alert-fatigue-are-worsening |
| E10 | VulnCheck State of Exploitation 2026 / 1H-2025 | 28.96% of KEVs exploited on/before CVE publication (1H-25 said 32.1%, revised; 23.6% in 2024); 884 new in 2025 | S | https://www.vulncheck.com/blog/state-of-exploitation-2026 |
| E11 | CISA KEV 2025 | 245 added, 1,484 total (+20%); 24 ransomware-linked; 94 older CVEs | S | https://www.securityweek.com/cisa-kev-catalog-expanded-20-in-2025-topping-1480-entries/ |
| E12 | NVD enrichment backlog | ~42k CVEs enriched in 2025; backlog >27k to >40k (**conflict**); IG projects >60k; risk-based enrichment from 15 Apr 2026 | S | https://www.helpnetsecurity.com/2026/06/01/nist-nvd-management-problems/ ; https://www.techzine.eu/news/security/140531/nist-updates-nvd-not-every-cve-will-be-scrutinized/ |
| E13 | ESET EDR-killer research (Mar 2026) | ~90 tools: 54 BYOVD (35 drivers), 7 script, 15 anti-rootkit/other; driverless emerging; The Gentlemen centralize; AI-assist suspected | S | https://www.eset.com/us/about/newsroom/research/eset-research-deep-dive-edr-killers-ransomware/ |
| E14 | Google GTIG AI-misuse report (5 Nov 2025) | PROMPTFLUX, PROMPTSTEAL (APT28), PROMPTLOCK, FruitShell (prompts aimed at LLM analyzers) | S | https://www.bleepingcomputer.com/news/security/google-warns-of-new-ai-powered-malware-families-deployed-in-the-wild/ |
| E15 | Anthropic GTG-1002 disclosure | 80-90% tactical ops by AI (provider-reported); ~30 targets; MCP tooling; role-play jailbreak; hallucination obstacle (wording unverified) | S | https://www.esecurityplanet.com/threats/inside-the-first-ai-driven-cyber-espionage-campaign/ |
| E16 | EchoLeak, CVE-2025-32711 | Zero-click M365 Copilot prompt injection; XPIA bypass; CSP-allowed Teams proxy; CVSS 9.3 (THN) | S | https://thehackernews.com/2025/06/zero-click-ai-vulnerability-exposes.html |
| E17 | ASD/ACSC (Feb 2025) and UK NCSC forensic-observability guidance | Network devices often have limited logging/investigation support | S | https://www.ncsc.gov.uk/sites/default/files/2026-08/Making-forensic-observability-the-norm-for-network-devices.pdf |
| E18 | ESXi ransomware DFIR vendor reports (Blackpanda, SRA, SOC Prime, Rewterz) | Decentralized ESXi logging; syslog disabling; hypervisor-ransomware 3%->25% (Huntress via aggregator - **unverified**) | S/V | https://www.blackpanda.com/case-studies/qilin-ransomware-esxi-incident-response |
| E19 | Wiz / Darktrace cloud-forensics material | Ephemeral workloads lose evidence; provider logs incomplete | V | https://www.wiz.io/courses/secops-for-cloud/forensics-and-root-cause-analysis/current-state-and-challenges-with-forensics |
| E20 | Gartner CTEM / AEV (via vendors) | 60% adopt exposure validation by 2029; 2022 "3x fewer breaches" prediction unvalidated; 29%/46% adoption (**unverified**) | S/V | https://hadrian.io/resources/2026-gartner-r-market-guide-for-adversarial-exposure-validation ; https://www.vectra.ai/topics/ctem |
| E21 | AVLab EDR/XDR tests (Jun 2026) | Partial cross-host correlation; command-line visibility gaps | S/V | https://avlab.pl/en/wp-content/uploads/2026/06/AVLab-EDR_XDR-CrowdStrike-2026-06-10.pdf |

## Academic / frontier

| ID | Paper / source | Key claims used | Level | URL |
|---|---|---|---|---|
| A01 | Bilot et al., "Sometimes Simpler is Better" - USENIX Security 2025 | 8 PIDS re-implemented; 9 shortcomings; simple NN matches SOTA on 5/7 DARPA sets; coarse alerts of 10^4 nodes; weak node-level attribution | A | https://www.usenix.org/conference/usenixsecurity25/presentation/bilot |
| A02 | Cai et al., NDSS 2025 PIDS dataset | Existing datasets outdated/short/incomplete chains; new emulated dataset | A | https://www.ndss-symposium.org/ndss-paper/auto-draft-665/ |
| A03 | Alachkar et al., "EvilEDR" - USENIX Security 2025 | EDR repurposed as offensive tool | A | https://www.usenix.org/conference/usenixsecurity25/presentation/alachkar |
| A04 | "Intermittent File Encryption in Ransomware" arXiv 2510.15133 | Byte-statistic changes; histogram detectability limits; BlackCat/Akira/LockBit modes; entropy-shaping evasion | A | https://arxiv.org/html/2510.15133v2 |
| A05 | Mahboubi et al., "Data Encryption Battlefield" | 11,928 files, 75 families; online learning | A | https://researchportal.murdoch.edu.au/esploro/outputs/preprint/Data-Encryption-Battlefield-A-Deep-Dive/991005775230907891 |
| A06 | TESSERACT (USENIX Sec 2019) / "Breaking Out from the Tesseract" (2025) / MADCAT | Temporal & spatial bias in malware ML evaluation | A | https://arxiv.org/html/2402.01359v2 |
| A07 | "Conflicting Scores, Confusing Signals" arXiv 2508.13644 (listing: CCS 2025) | <20% of 1,226 KEV CVEs had EPSS>=0.5 pre-KEV; 8.3% >=0.9; >22% no EPSS | A | https://arxiv.org/pdf/2508.13644 |
| A08 | Paul, look-ahead-bias-corrected EPSS evaluation (IEEE DataPort 2025) | 134,821 CVEs 2021-2025; EPSS > CVSS-only after correction | A | https://ieee-dataport.org/documents/how-good-epss-really-five-year-empirical-evaluation-correcting-look-ahead-bias |
| A09 | "A Reality Check on SBOM-based Vulnerability Management" arXiv 2511.20313 | 31/121 vulns confirmed on 12 Go projects (74.4% FP reduction) | A | https://arxiv.org/pdf/2511.20313 |
| A10 | "The Attacker Moves Second" arXiv 2510.09023 | 12 defenses broken, mostly >90% ASR via adaptive attacks | A | https://arxiv.org/abs/2510.09023 |
| A11 | "Poisoning the Watchtower" arXiv 2605.24421 | Log-field prompt injection; 48 combos; gpt-4o-mini; best defense 26.6%->11.8% | A | https://www.alphaxiv.org/abs/2605.24421 |
| A12 | "Context Contamination in LLM Analysis of Network Logs" arXiv 2607.14493 | Related SOC-log injection study (snippet only) | A | https://www.alphaxiv.org/abs/2607.14493.md |
| A13 | CyberSOCEval (Meta) arXiv 2509.20166 | Malware-analysis / CTI reasoning benchmark; models not saturating | A | https://arxiv.org/html/2509.20166v2 |
| A14 | ExCyTIn-Bench (Microsoft), Simbian benchmark, SentinelLabs critique, Elastic suite | Rewards 0.249 avg / 0.368 best; attribution gap; benchmark critique | A/S | https://www.sentinelone.com/labs/llms-in-the-soc-part-1-why-benchmarks-fail-security-operations-teams |
| A15 | DFIR-Metric arXiv 2505.19973 | 700 MCQ + 150 CTF + 500 CFTT cases; 14 LLMs | A | https://web3.arxiv.org/abs/2505.19973 |
| A16 | "Temporal Dynamics of CTI" arXiv 2412.19086 | <=4.5% provider overlap (citing prior studies) | A | https://arxiv.org/html/2412.19086v1 |
| A17 | SANS.edu "Adversary-Aware IOC Retention" | Uniform decay models ignore actor TTPs; 3 APTs | A | https://www.sans.edu/cyber-research/adversary-aware-ioc-retention-analyzing-time-live-patterns-threat-actor-attribution/ |
| A18 | IntelEX arXiv 2412.10872; arXiv 2505.11547; TTPrint arXiv 2605.25836 | LLM CTI extraction vs analyst annotation divergence | A | https://arxiv.org/html/2412.10872v1 |
| A19 | Hypergraph privilege analysis arXiv 2511.15837 | O(sqrt n) claim (self-reported) | A | https://arxiv.org/html/2511.15837 |
| A20 | Attack-graph scalability arXiv 2312.16513 | Exponential growth; open problem | A | https://web3.arxiv.org/pdf/2312.16513v1 |
| A21 | arXiv 2406.07559 (IR within enterprises); StealthCup arXiv 2511.17761 | Simulation realism open; benchmark limits | A | https://arxiv.org/pdf/2406.07559 |
| A22 | Beyond CAGE (2211.15557); KTH (2407.05775); arXiv 2505.04843 | RL defender generalization failures | A | https://ar5iv.labs.arxiv.org/html/2211.15557 |
| A23 | Le & Zhang ICSE 2022, arXiv 2202.04301 | Log anomaly DL leakage (older, foundational) | A | https://arxiv.org/pdf/2202.04301 |
| A24 | LLMCloudHunter 2407.05194; EvoSIEM; RulePilot 2511.12224; RuleGenie 2505.06701 | LLM rule-gen evaluated on syntax/similarity | A | https://arxiv.org/pdf/2407.05194 |

## Known source conflicts (resolved by primary where possible)

1. **M-Trends 2026 initial vectors.** One search summary said phishing 11% and stolen credentials 9%; the primary page says **voice phishing 11%**, **email phishing 6%** and reports no stolen-credential percentage. The primary was used.
2. **CrowdStrike 2026 GTR.** Breakout-time and malware-free percentages appeared with inconsistent comparisons across outlets (e.g., "65% faster" vs "70% below 2021"). Treated as approximate and unverified.
3. **VulnCheck pre-disclosure exploitation.** 32.1% (1H-2025) was revised to 28.96% (full-year). Latest used.
4. **NVD backlog size.** Estimates differ by source (>27k, >40k, >60k projected). Reported as a range.
5. **Microsoft AI-phishing multiplier.** 4.5x vs "three times" in different outlets. Reported as 4.5x (54% vs 12%) with the conflict flagged.
