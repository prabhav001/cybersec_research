# Research Log and Limitations

Session date: 2026-11-02

## Method
- ~40 web searches (standard mode) across the ten domains, split into industry telemetry and academic/frontier fronts, followed by targeted fetches.
- Search results were summaries produced by the search tool; they often flagged their own conflicts. Those flags were propagated into the ledger.

## Access limitations (must be fixed to upgrade evidence from S/A to P)
The cloud session's network policy blocked these hosts (DNS did not resolve):
- `www.crowdstrike.com` (CrowdStrike Global Threat Report)
- `www.verizon.com` (DBIR executive summary)
- `www.zscaler.com` (hosted copy of the Red Canary 2026 executive summary)
- `arxiv.org` (all papers)

Only `cloud.google.com` was fetched successfully, so **only Mandiant M-Trends 2026 is verified at primary level (P)**.
Once the hosts above (and ideally `usenix.org`, `ndss-symposium.org`, `dl.acm.org`, `ieeexplore.ieee.org`, `microsoft.com`, `cisa.gov`, `first.org`) are added under the environment's Network access -> Allowed domains, the verification TODO list below can be executed.

## Verification TODO (priority order)
1. CrowdStrike 2026 GTR: breakout time, malware-free %, pre-disclosure exploitation %.
2. Verizon 2026 DBIR: exploitation share, time-to-remediate for KEV/edge devices.
3. Microsoft DDR 2025: AI-phishing click-through, identity-attack statistics, token-theft share.
4. Red Canary 2026 TDR: ranked technique list and the identity share.
5. Anthropic GTG-1002 report: automation percentage and hallucination wording.
6. arXiv: 2510.15133, 2508.13644, 2510.09023, 2605.24421, 2511.15837 - read fully; check venues (CCS 2025 / CCS 2026 claims in search listings were not confirmed).
7. CardinalOps and SANS primary reports.
8. Dataset availability checks (see problem_statements.md section 8 of each problem).

## Interpretation choices
- "Defender AI" is interpreted as AI systems deployed on the defensive side (SOC copilots/agents, Defender / Security Copilot-class tooling), analysed both as capability and attack surface.
- Academic items older than 2025 (TESSERACT 2019, Le & Zhang 2022, attack-graph 2023) are included only as foundational context where no 2025/26 equivalent was found.
- No academic paper was read in full. Academic evidence is at abstract/snippet level.
