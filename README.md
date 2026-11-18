# 🔐 Trust Wallet v2.68 Supply-Chain Compromise: Cyber Incident Report

A group analysis of the December 2025 compromise of the **Trust Wallet Chrome extension (v2.68)**, in which attackers used a leaked Chrome Web Store API key to push a malicious update and drain user wallets. The report works through the incident using industry frameworks (ENISA, MITRE ATT&CK, CWE/CVSS, VERIS, STRIDE-LM, Microsoft Threat Modeling Tool) and ends with root cause, eradication and a comparison to a similar attack.

![Incident](https://img.shields.io/badge/incident-Dec%202025-red)
![Type](https://img.shields.io/badge/type-software%20supply%20chain-orange)
![Frameworks](https://img.shields.io/badge/frameworks-MITRE%20ATT%26CK%20%7C%20STRIDE--LM%20%7C%20CVSS%20%7C%20VERIS-blue)
![Format](https://img.shields.io/badge/format-report%20(.docx)-lightgrey)

![Trust Wallet organisational data flow diagram showing supply-chain attack pathways](images/combined-dfd.png)
*Combined organisational data flow diagram from the report: how stolen credentials led to tampered code, malicious auto-updates, seed-phrase exfiltration and brand damage.*

---

## Table of Contents

- [Incident at a Glance](#incident-at-a-glance)
- [Attack Chain](#attack-chain)
- [What the Report Covers](#what-the-report-covers)
- [Assets and Owners](#assets-and-owners)
- [Key Findings](#key-findings)
  - [Classification (ENISA, VERIS, STRIDE-LM)](#classification-enisa-veris-stride-lm)
  - [Vulnerabilities and Risk Scoring](#vulnerabilities-and-risk-scoring)
  - [Threat Modelling and Mitigations](#threat-modelling-and-mitigations)
  - [Detection and Forensics](#detection-and-forensics)
  - [Root Cause and Eradication](#root-cause-and-eradication)
- [Repository Contents](#repository-contents)
- [Selected References](#selected-references)
- [Authors](#authors)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## Incident at a Glance

| | |
|---|---|
| **Target** | Users of the Trust Wallet browser extension (Trust Wallet is owned by Binance) |
| **Compromised version** | Chrome extension **v2.68** |
| **Attack window** | 24-26 December 2025 |
| **Initial access** | Chrome Web Store (CWS) API key and developer secrets exposed by the **Shai-Hulud** npm supply-chain worm (November 2025) |
| **Wallets affected** | **2,520** (Trust Wallet's official count) |
| **Estimated loss** | roughly **7-8.5 million USD**, depending on source |
| **Goal** | Financial: steal wallet seed phrases and drain funds |
| **Attacker identified?** | No |

> Figures vary between the sources cited in the report (wallet counts from 2,500 to 3,000 and losses from about 7 to 8.5 million USD appear in coverage). The table uses Trust Wallet's official wallet count and the range of reported losses.

---

## Attack Chain

1. **Credential theft (November 2025):** the Shai-Hulud worm spread through malicious npm packages and harvested developer secrets, including the CWS API key and GitHub secrets.
2. **Staging (weeks before):** the attackers registered a lookalike domain (`api[.]metrics-trustwallet[.]com`) to receive stolen data.
3. **Code tampering:** malicious JavaScript was hidden in the extension's PostHog analytics logic so it looked like routine telemetry.
4. **Malicious publish (24 Dec 2025):** v2.68 was pushed through the official Chrome Web Store using the stolen key, bypassing Trust Wallet's internal review. The timing, Christmas Eve, meant reduced staffing.
5. **Seed-phrase theft:** when a user opened or unlocked the wallet, the code exfiltrated their recovery phrase to the attacker's server.
6. **Draining and laundering:** wallets were emptied and funds moved through swaps, cross-chain bridges and centralised exchanges (e.g. ChangeNOW).
7. **Follow-on phishing:** fake "support" accounts offered refunds to trick victims into handing over recovery phrases.
8. **Response:** v2.68 was recalled, Chrome revoked the key and tightened publishing, and Trust Wallet committed to reimbursing victims. It reported more than 5,000 claims, many suspected to be false.

---

## What the Report Covers

The report follows its own topic numbering (there is no Topic 3 section).

| Topic | Question addressed | Frameworks / methods |
|:-----:|--------------------|----------------------|
| **1** | What was attacked, how, who was affected, impact | Incident description, stakeholder analysis |
| **2** | Which threat category, and how stakeholders were impacted | ENISA Threat Landscape 2024, CIA triad |
| **4** | Asset identification, sensitivity, classification | Asset register, one owned asset per member |
| **5** | Techniques used; prevention and early detection | MITRE Red Report 2025, ATT&CK techniques, mitigations and detections |
| **6** | Exploited weaknesses; attacker capability | CWE, CVE, CORAS threat diagrams |
| **7** | Threat actor classification | VERIS (External / Internal / Partner) |
| **8** | Severity and risk | CVSS / NVD calculator, 5x5 risk matrix, STRIDE-LM, risk statements |
| **9** | Critical threat per asset and its mitigation | Data flow diagrams, Microsoft Threat Modeling Tool, control categories |
| **10** | Evidence and investigation | IOAs / IOCs, log sources, analysis tools, interview questions |
| **11** | Root cause, eradication, similar incident | Root cause analysis, comparison to the Axios npm compromise |

---

## Assets and Owners

Assets were ranked from most to least critical, and each member owned one for the rest of the analysis.

| Rank | Asset | Owner | Classification |
|:----:|-------|-------|----------------|
| 1 | Cryptocurrency / wallet contents | Liam Duignan | High Value Asset |
| 2 | Chrome Web Store API key | Emmett Leahy | Secret / credential asset (confidential) |
| 3 | Trust Wallet extension source code | Joel Biju | Proprietary / IP asset (confidential or restricted) |
| 4 | Trust Wallet brand reputation | Nithin Kumaran | Public-facing intangible asset |

---

## Key Findings

### Classification (ENISA, VERIS, STRIDE-LM)

**ENISA Threat Landscape categories:** Malware, Supply-Chain Attack (primary) and Social Engineering (the follow-on refund-scam phishing, and a possible role in obtaining credentials).

**VERIS actor classification**

| Actor | Assessment |
|-------|------------|
| External | Organised, financially motivated criminal group (the preparation points to more than a lone attacker) |
| Internal | Possible insider, raised publicly by Binance's CEO; **no confirmed evidence** |
| Partner | The Shai-Hulud compromise of vendor/dependency tooling that exposed the key |

**STRIDE-LM mapping:** every category was judged relevant except Denial of Service, since the extension kept working during the attack.

| Category | How it appeared |
|----------|-----------------|
| Spoofing | Stolen API key used to impersonate a legitimate publisher |
| Tampering | Malicious code added to the extension source |
| Repudiation | Legitimate-looking credentials, plus laundering that obscured the trail |
| Information disclosure | Seed phrases and keys sent to an unauthorised server |
| Denial of service | Not applicable |
| Elevation of privilege | The key raised the attacker to a trusted publisher position |
| Lateral movement | From publishing access to the source code |

### Vulnerabilities and Risk Scoring

Each member analysed a weakness relevant to their asset, scored it with CVSS, and rated risk on a 5x5 likelihood x impact matrix.

| Asset | Weakness analysed | CVSS | Likelihood x Impact | Risk |
|-------|-------------------|:----:|:-------------------:|------|
| Wallet contents | CWE-200: exposure of sensitive information | ~8.1 (estimated) | 4 x 5 = 20 | Very High |
| CWS API key | CVE-2025-10894 (Nx npm compromise, used as a reference for the Shai-Hulud-style mechanism) | 9.6 (per Red Hat) | 4 x 5 = 20 | Critical |
| Extension source code | CWE-494: download of code without integrity check | 8.9 | 3 x 5 = 15 | High |
| Brand reputation | CWE-693: protection mechanism failure | 8.9 | 3 x 5 = 15 | High |

Other weaknesses referenced: CWE-284, CWE-312, CWE-345, CWE-506, CWE-1009, CWE-1357.

The group concluded that CVSS and the risk matrix are **not directly comparable**: CVSS rates technical severity in isolation, while the matrix captures business likelihood and impact. Both agreed the incident was serious.

### Threat Modelling and Mitigations

| Asset | Most critical threat (STRIDE) | Mitigation modelled |
|-------|-------------------------------|---------------------|
| Wallet contents | Tampering | Signed builds with integrity verification before publication |
| Extension source code | Elevation of privilege | Stronger publisher verification and internal code signing, so one API key cannot publish |
| CWS API key | Spoofing | MFA (FIDO2 / TOTP) on publish, plus mandatory HSM-backed code signing |
| Brand reputation | Spoofing | MFA on all publish actions plus internal code signing |

**Common theme:** a single stolen credential should never be enough to publish software to users. The proposed controls are preventive and fall under integrity.

### Detection and Forensics

**Indicators of attack:** newly registered lookalike domain; update published outside normal hours with no internal release record; CWS authentication from an unrecognised IP; published package that does not match internal build hashes; unexpected post-install scripts in CI.

**Indicators of compromise:** DNS/HTTPS connections to `api[.]metrics-trustwallet[.]com` between 24 and 26 December 2025; extension v2.68 containing the modified analytics code; on-chain drains from affected wallets to new addresses; a v2.68 publish-log entry from a non-Trust-Wallet source.

<details>
<summary><b>Log sources and tools the report proposes</b></summary>

| Source | What it would show |
|--------|--------------------|
| Chrome Web Store developer publish / audit log | When, and from which IP, v2.68 was published |
| GitHub organisation audit log and Actions secrets log | Secret access and repository changes outside normal workflows |
| npm install / debug logs (`~/.npm/_logs/`) | Compromised packages and post-install scripts from the November builds |
| Chrome debug log and extension activity log | Extension load events and outbound requests |
| Windows DNS Client and Application event logs | Lookups of the attacker domain, Chrome runtime entries |
| Public blockchain explorers | Draining and laundering trail |

**Tools:** `jq`, `git log` / `git diff`, `grep` / `cat`, `npm audit`, DB Browser for SQLite, Event Viewer and PowerShell.

</details>

The report also lists interview questions for Trust Wallet's security and DevOps teams, Google, affected users and the communications team.

### Root Cause and Eradication

- **Root cause:** the Chrome Web Store API key exposed in the November 2025 Shai-Hulud incident was not revoked and rotated, so it stayed valid in the attackers' hands until the December attack.
- **Eradication:** rotate all suspected-compromised credentials immediately, audit the development environment for compromised npm packages, and require MFA plus code signing on publishing.
- **Similar incident:** the Axios npm package compromise (31 March 2026) also abused maintainer credentials to ship malicious code through a trusted channel. It was contained in about 3 hours, versus roughly 2 days for the malicious Trust Wallet extension.

---

## Repository Contents

```
.
├── TrustWallet2.docx      # Full incident report
├── images/
│   └── combined-dfd.png   # Combined organisational data flow diagram
└── README.md
```

The full report also contains the per-asset DFDs, CORAS threat diagrams, CVSS calculator screenshots, risk matrices and Microsoft Threat Modeling Tool outputs.

---

## Selected References

The report's full reference list has 38 entries. Key sources:

- [Trust Wallet: Browser Extension v2.68 Incident update](https://trustwallet.com/blog/announcements/trust-wallet-browser-extension-v268-incident-community-update)
- [Koi Security: Inside the code that stole $7M on Christmas Eve](https://www.koi.ai/blog/trust-wallet-binance-compromised-inside-the-code-that-stole-7m-on-christmas-eve)
- [SlowMist: Christmas Heist, analysis of the Trust Wallet extension hack](https://slowmist.medium.com/christmas-heist-analysis-of-trust-wallet-browser-extension-hack-bdb35c3cc6dd)
- [The Hacker News: Trust Wallet Chrome extension hack drains $8.5M](https://thehackernews.com/2025/12/trust-wallet-chrome-extension-hack.html)
- [SecurityWeek: Shai-Hulud supply chain attack led to $8.5M Trust Wallet heist](https://www.securityweek.com/shai-hulud-supply-chain-attack-led-to-8-5-million-trust-wallet-heist/)
- [ENISA Threat Landscape 2024](https://op.europa.eu/en/publication-detail/-/publication/e71394ea-85f0-11ef-a67d-01aa75ed71a1/language-en)
- [MITRE ATT&CK T1195: Supply Chain Compromise](https://attack.mitre.org/techniques/T1195/)
- [Red Hat: CVE-2025-10894](https://access.redhat.com/security/cve/cve-2025-10894)
- [Microsoft: Mitigating the Axios npm supply chain compromise](https://www.microsoft.com/en-us/security/blog/2026/04/01/mitigating-the-axios-npm-supply-chain-compromise/)

---

## Authors

| Name | Asset analysed |
|------|----------------|
| Emmett Leahy | Chrome Web Store API key |
| Nithin Kumaran | Trust Wallet brand reputation |
| Joel Biju | Trust Wallet extension source code |
| Liam Duignan | Cryptocurrency / wallet contents |

---

## Disclaimer

This is an independent analysis built from public reporting and is not affiliated with Trust Wallet, Binance or Google. Some conclusions, such as attacker sophistication, a possible insider and several impact statements, are marked in the report as assumptions or speculation. Indicators of compromise are defanged (`[.]`) so they are not clickable.

---

## License

<!-- No license specified. Add one (e.g. CC BY 4.0 for the report) and update this section. -->
No license has been specified yet.
