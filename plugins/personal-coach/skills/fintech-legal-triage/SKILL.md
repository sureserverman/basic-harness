---
name: fintech-legal-triage
description: Use when a fintech feature, contract, partnership, or operational change may have regulatory exposure — KYC/AML, payment licensing, lending limits, crypto/MiCA, data localisation, customer T&Cs, marketing claims, sanctions screening. Asks jurisdiction first, walks the issue checklist, and produces a list of named regulations and questions to take to a real licensed lawyer. Triggers on "is this legal", "what regs apply", "do we need a license", "review this contract", "what about KYC", "is this GDPR-compliant". Issue-spotter, not legal advice — every output ends with a take-to-counsel block.
---

# Fintech Legal Triage

Issue-spotter for fintech work. The skill exists because most legal exposure in fintech doesn't come from a single bad clause — it comes from a missing question. The user usually doesn't need a verdict; they need to know which questions belong on the agenda for the company's lawyer, and which regulations to pull off the shelf.

**Announce at start:** "Using the fintech-legal-triage skill. I'll surface the issues and the regulations they live under, but I will not give legal advice — every issue I find is a question for your licensed lawyer."

<HARD-GATE>
Phase 0 (jurisdiction) is mandatory. Without jurisdiction, regulatory analysis is fiction. Refuse to proceed if the user won't answer the jurisdiction question.
</HARD-GATE>

<HARD-GATE>
Every output ends with the **take-to-counsel** block (Phase 5). No exceptions, even for trivial-seeming questions. The pattern is what makes the skill safe to use.
</HARD-GATE>

## Phase 0 — Jurisdiction (mandatory)

Ask, in order:

1. **Where is the company incorporated?** (e.g., "Estonia", "Delaware (USA)", "England & Wales (UK)", "Cyprus", "Singapore", "Russia", "UAE — DIFC", "UAE — ADGM".)
2. **Where do the customers sit?** (Same question, repeated for each material customer base. EU + UK + US is three jurisdictions, not one.)
3. **What's the activity?** Pick the closest:
   - **Payments / e-money** — moving funds between users.
   - **Lending / credit** — issuing loans, BNPL, revolving credit.
   - **Investments / brokerage** — securities, funds, robo-advisory.
   - **Crypto / digital assets** — custody, exchange, stablecoin, on/off-ramp.
   - **Banking-as-a-service / sponsor bank model** — riding on a regulated partner.
   - **B2B fintech infrastructure** — APIs to other regulated firms.
   - **Insurance / insurtech** — underwriting or distribution.
   - **Other / ambiguous** — describe in one sentence.

If the user says "we operate worldwide", push back: regulators don't accept "worldwide" — pick the top 3 by customer count or revenue and treat them as the working set.

If the user is in a jurisdiction the assistant doesn't have grounded knowledge of, **say so explicitly**: "I don't have reliable knowledge of <country>'s fintech rules. The methodology below still applies — I can flag categories of issue, but the specific regulation names will need to come from your local counsel."

## Phase 1 — Pick the issue checklist

Match jurisdiction × activity to the relevant body of regulation. Below is the working set the skill is grounded in — additions require a citation.

### EU / EEA

- **Payments / e-money** — PSD2 (Directive (EU) 2015/2366) and the proposed PSD3/PSR; EMI vs PI authorisation under EMD2 (Directive 2009/110/EC); SCA / strong customer authentication (RTS under PSD2).
- **AML/KYC** — 5AMLD (Directive (EU) 2018/843), 6AMLD (Directive (EU) 2018/1673), and the new AML Package (AMLR + AMLD6 + AMLA, 2024).
- **Crypto/digital assets** — MiCA (Regulation (EU) 2023/1114); DAC8 (Directive (EU) 2023/2226) for tax reporting; Travel Rule under TFR (Regulation (EU) 2023/1113).
- **Data protection** — GDPR (Regulation (EU) 2016/679); ePrivacy Directive (2002/58/EC, amended).
- **Consumer credit** — CCD (Directive 2008/48/EC) and the revised CCD2 (2023).
- **Investments** — MiFID II (Directive 2014/65/EU); PRIIPs (Regulation (EU) 1286/2014).
- **Marketing** — Unfair Commercial Practices Directive (2005/29/EC); national marketing rules.

### UK

- **Payments / e-money** — Payment Services Regulations 2017 (PSRs); Electronic Money Regulations 2011 (EMRs); FCA authorisation.
- **AML/KYC** — Money Laundering Regulations 2017 (as amended); JMLSG Guidance; FCA's Financial Crime Guide.
- **Crypto** — FCA's cryptoasset financial-promotions regime (2023); MLRs registration for cryptoasset firms; HM Treasury's stablecoin and broader crypto regulatory roadmap.
- **Data protection** — UK GDPR; Data Protection Act 2018.
- **Consumer credit** — Consumer Credit Act 1974 and FCA CONC sourcebook.
- **Marketing** — FCA financial promotions regime (s.21 FSMA); Consumer Duty (PRIN 2A, in force July 2023).

### US (federal-level only — state law is its own world)

- **Payments / money transmission** — Bank Secrecy Act; FinCEN MSB registration; state-by-state money transmitter licensing (NMLS).
- **AML/KYC** — BSA/AML; OFAC sanctions; CIP rule (31 CFR 1020.220).
- **Crypto** — FinCEN MSB rules for "money transmitters"; SEC enforcement under *Howey*; CFTC for derivatives; NYDFS BitLicense (NY only).
- **Lending** — Truth in Lending Act (TILA); ECOA; UDAAP under CFPB; state usury caps.
- **Data** — sectoral: GLBA for financial data; state laws (CCPA/CPRA in California, etc.).

### Russia / EAEU

- **Banking and payments** — 161-FZ "On the National Payment System"; CBR (Bank of Russia) licensing for credit organisations.
- **AML/KYC** — 115-FZ "On Combating Legalisation of Income".
- **Crypto** — 259-FZ "On Digital Financial Assets" (2020); 31-FZ amendments restricting crypto-payments domestically.
- **Data** — 152-FZ "On Personal Data" (data localisation requirement under amendments to 152-FZ and 242-FZ).
- **Sanctions exposure** — must be cross-checked against EU/UK/US lists for any cross-border activity.

### Singapore

- **Payments** — Payment Services Act 2019 (PSA); MAS PS-N02 AML/CFT notice.
- **Crypto** — PSA Major Payment Institution licence with DPT (digital payment token) service; MAS guidance on DPT consumer protection (2023).
- **Data** — Personal Data Protection Act 2012.

### UAE

- **Mainland** — Central Bank of the UAE (CBUAE) for payments; SCA for securities; VARA in Dubai for virtual assets (Law 4/2022).
- **DIFC** — Dubai Financial Services Authority (DFSA) regime.
- **ADGM** — Financial Services Regulatory Authority (FSRA) regime, including the FSRA Crypto Asset Framework (Guidance, 2018 / updates).

If the activity × jurisdiction combination isn't in this list, do not invent regulation names. Fall back to: "The applicable framework should be confirmed by local counsel; relevant categories include <AML, payment licensing, data protection, consumer protection, marketing>."

## Phase 2 — Walk the issue list for the matched cell

For each matched body of regulation, ask the user a short yes/no series. Below is the canonical set; pick the relevant subset for the activity.

### KYC / AML (universal)

1. Do you collect and verify customer ID before account opening? (CDD trigger.)
2. Do you screen against sanctions lists at onboarding **and** ongoing? Which lists?
3. Do you risk-rate customers (low / medium / high)?
4. Do you apply enhanced due diligence (EDD) for PEPs, high-risk countries, and complex structures?
5. Do you have a designated MLRO (UK) / Compliance Officer (EU) / BSA Officer (US)?
6. Do you file SARs/STRs with the right FIU? Do you train staff to spot red flags?
7. Source-of-funds and source-of-wealth on high-risk customers?
8. Travel Rule data on crypto transfers above the de minimis threshold?
9. Record retention period? (5-7 years depending on regime.)

### Payment licensing

1. Are you holding customer funds at any moment? If yes, you're likely an EMI / PI / MSB and need authorisation.
2. Are you safeguarding customer funds in segregated accounts at a credit institution? (Mandatory under PSD2 art. 10.)
3. Are you relying on a sponsor / partner bank's licence? Have you reviewed your agency / agent agreement for licence-perimeter compliance?
4. Initial capital and own-funds requirements for the relevant licence?
5. Consumer dispute resolution path? (Voluntary or mandatory ADR scheme.)

### Crypto / digital assets

1. Do your tokens fall in scope of MiCA (utility / asset-referenced / e-money / NFT carve-out)? **Issuers** and **CASPs** have different obligations.
2. Whitepaper requirements for token offerings? Marketing communications consistency?
3. Custody segregation and bankruptcy-remoteness?
4. Travel Rule (TFR) data fields for crypto transfers — originator and beneficiary, with the EUR 1,000 threshold.
5. Sanctions screening on wallet addresses? Chain analytics provider?
6. Tax-reporting (DAC8 in EU, IRS in US)?

### GDPR / data protection (universal in EU / UK)

1. Lawful basis for each processing purpose? (Consent ≠ legitimate interest ≠ contract necessity.)
2. Special category data (biometrics in onboarding selfies — Article 9)?
3. International transfers — SCCs in place? Transfer Impact Assessment for non-adequate countries?
4. DPIA done for high-risk processing? (Onboarding involving biometric + automated decision-making is a classic trigger.)
5. Data subject rights workflow — access, rectification, erasure, portability — with named SLAs?
6. Breach notification — 72-hour rule to supervisory authority; notification to data subjects when high-risk?
7. DPO appointed if required (large-scale systematic monitoring or special category processing)?
8. Records of Processing Activities (Art. 30 ROPA) maintained?
9. Vendors on data processing agreements (Art. 28 DPAs)?

### Customer-facing terms (T&Cs, fees, complaints)

1. Pre-contract disclosure that meets the relevant regime (PSD2 framework contract; CCD pre-contractual info).
2. Fee schedule transparency — standalone or embedded?
3. Variation clauses — notice period, customer's right to terminate without penalty.
4. Complaints handling SLA aligned to local rules (e.g., FCA DISP 1.6 — final response within 8 weeks).
5. Right to terminate; data retention after termination.
6. Governing law and forum — does it match jurisdiction realities, or does it conflict with consumer protection mandatory rules?
7. Dispute resolution — ombudsman / FOS in UK, equivalent ADR in EU member states.
8. UCTD-grade fairness review (Directive 93/13/EEC in EU, equivalents elsewhere) for all consumer terms.

### Marketing and financial promotions

1. Promotional approval by an authorised firm where required? (UK s.21 FSMA, FCA promotions regime.)
2. Risk warnings on regulated products (especially crypto and high-risk investments)?
3. Targeting and exclusions — non-eligible counterparty exclusions, cross-border solicitation rules?
4. Sponsorship / influencer compliance — labelling, retention of approved scripts.

## Phase 3 — Output the issue list

Format the output as a structured issue list, **not** as a verdict. Each issue cites the regulation by name and points at what the user / their counsel needs to do.

```markdown
# Fintech legal triage — <one-sentence topic>
Date: <YYYY-MM-DD>
Jurisdiction(s): <list>
Activity: <category from Phase 0.3>

## Issues identified

### Issue 1 — <short title>
- **What's at risk:** <one sentence>
- **Regulation(s):** <named regs / directives / acts>
- **Question for counsel:** <the specific question>
- **Confidence this is in scope:** <high / medium / low>

### Issue 2 — ...

## Issues explicitly considered and ruled out of scope (with reason)
<two or three; explicit-negative is better than silent>

## Open jurisdictional gaps
<places the user named where the assistant lacks reliable knowledge — flagged so counsel knows>
```

If the user is doing something that looks like an obvious red flag — operating without a licence in a regime that requires one, missing a Travel Rule field, processing biometric data without DPIA — name it as `Confidence: high` and mark it urgent.

## Phase 4 — Save to the legal log

If a personal vault is configured, save to `<vault>/Legal/YYYY-MM-DD-<slug>.md`. Otherwise, `~/.claude/legal-triage/YYYY-MM-DD-<slug>.md`. The log builds a record over time of what was triaged when, which becomes evidence of diligence.

## Phase 5 — The take-to-counsel block (mandatory)

Every output ends with this block, edited only to fill in jurisdiction:

> **This is not legal advice.** I am an AI assistant flagging issues; I am not licensed to practise law in <jurisdiction(s)>. Before acting on anything in the issue list — including the items I marked low-confidence — take this list to a lawyer qualified in the relevant jurisdiction(s). The most expensive mistakes in fintech come from acting on the assumption that "the rules are probably fine"; the cheapest insurance is a 30-minute call with counsel before you ship.

Do not omit this block to be terse. Do not soften it. The block is the skill's safety guarantee.

## Hard refusals

- **No "is this legal" yes/no answers.** The skill returns issue lists, not verdicts.
- **No drafting of regulated documents** (terms of service for a regulated product, key information documents, prospectuses, regulator filings) — those need a lawyer's signoff. The skill can sketch a structure or surface a missing clause, never produce final text.
- **No advising on enforcement strategy** (what to say to a regulator under investigation). That's privileged work.
- **No interpreting case law.** Citations to specific cases are out of scope; the skill stays at the regulation level.
- **No advising on opening a deliberately under-regulated entity** to evade rules ("set up in <country> to avoid AML"). Refuse and explain.

## What this skill is NOT

- Not a substitute for outside counsel. The skill is the *prep work* that makes a 30-minute call with counsel produce more value than a 3-hour call without it.
- Not a compliance management system. It does not track obligations over time, schedule renewals, or replace a GRC tool.
- Not a sanctions screening tool. It can flag that screening is required; it does not perform it.

## In Cowork (connector-aware enrichment)

This is the skill that benefits most from Cowork's document-handling surface. A contract or T&Cs doc is a much better triage input than a verbal summary.

- **Google Drive** — read the contract / T&Cs / partnership agreement / data processing agreement directly from Drive when the user names the file. The skill then walks the issue checklist against the actual clauses, not against a description of them. Confidence values rise accordingly.
- **DocuSign** — if a contract is in flight, read the template / latest version. Surface the issue list **before** signing, not after.
- **PDF / file uploads** — Cowork accepts PDF uploads natively. The user can drag a regulator's guidance PDF into the chat and the skill will cite specific paragraphs in the issue list.
- **Web** — `WebFetch` works in both Code and Cowork. Useful for reading current text of named regulations (e.g., the latest consolidated MiCA text on EUR-Lex). Always cite the URL the user can verify.
- **Gmail** — generally avoid. Lawyer–client correspondence in inbox is privileged and should not be consulted as routine context.

The take-to-counsel block at the end is **non-negotiable regardless of how grounded the analysis is**. Reading the actual contract makes the issues higher-confidence; it does not turn the assistant into a lawyer.

In a cloud Routine: a `legal-triage-on-drive-update.md` Routine watches a designated Drive folder; when a new contract lands, it produces the issue list automatically and pings the user to review with counsel before signing. **Privacy tradeoff: the document text passes through Anthropic's cloud during Routine execution.** If the contract is highly sensitive, run the triage as a desktop Scheduled Task or interactive session instead.

## Sources and rationale

- **EU payments framework** — PSD2 (Directive (EU) 2015/2366); EMD2 (Directive 2009/110/EC); EBA Guidelines.
- **EU AML** — 5AMLD (2018/843); 6AMLD (2018/1673); 2024 EU AML Package (Regulation establishing AMLA + AMLR + AMLD6).
- **EU crypto** — MiCA (Regulation (EU) 2023/1114); TFR (Regulation (EU) 2023/1113); DAC8 (Directive (EU) 2023/2226).
- **EU data** — GDPR (Regulation (EU) 2016/679); EDPB Guidelines, esp. on transfers (post-Schrems II).
- **UK** — FSMA 2000; PSRs 2017; EMRs 2011; MLRs 2017; FCA Handbook (esp. CONC, DISP, PRIN, COBS); Consumer Duty (PRIN 2A, July 2023).
- **US** — Bank Secrecy Act; FinCEN regulations (31 CFR ch. X); OFAC; CFPB enforcement priorities; SEC v. *Howey* (1946) framework.
- **Russia** — 161-FZ; 115-FZ; 152-FZ; 259-FZ; 31-FZ.
- **Singapore** — Payment Services Act 2019; PDPA 2012; MAS notices.
- **UAE** — DFSA Rulebook (DIFC); FSRA Rulebook + Crypto Asset Framework Guidance (ADGM); VARA Regulations (Dubai); CBUAE regulations.
- **AML/KYC fundamentals** — FATF *International Standards on Combating Money Laundering and the Financing of Terrorism & Proliferation* (40 Recommendations, latest revision); Basel Committee CDD principles.

These are the canonical sources the issue checklists are built from. Anything beyond this list requires explicit user confirmation that the source applies to their cell.
