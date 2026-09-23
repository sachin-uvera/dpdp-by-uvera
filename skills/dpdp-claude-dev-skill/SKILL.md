---
name: dpdp-claude-dev-skill
description: India DPDP Act 2023 + DPDP Rules 2025 compliance for software development — use while designing, building, reviewing or auditing any product that touches personal data of people in India. Turns the law into engineering controls (consent ledger, notices, rights APIs, retention/erasure jobs, breach runbook, child age-gating, logging, encryption) and gives a verification checklist with section/rule citations. Use this skill whenever the user mentions DPDP, DPDPA, Indian privacy law, personal data, PII, consent flows, sign-up/onboarding forms, user deletion, data retention, audit logs, breach/incident response, children's data, privacy policy, cookie/consent banners, or asks "is this compliant", "review this build/schema/API for privacy", or is building any app, SaaS, IoT or AI product for Indian users or clients — even if DPDP is not named.
---

# DPDP Dev Skill — building software that complies with India's DPDP law

Source texts (read in full when this skill was written):
- Digital Personal Data Protection Act, 2023 (No. 22 of 2023) — cited as **S.** (section)
- Digital Personal Data Protection Rules, 2025, G.S.R. 846(E), notified 13 Nov 2025 — cited as **R.** (rule) and **Sch.** (schedule)
- PIB explainer "DPDP Rules, 2025 Notified" (17 Nov 2025)

This skill is an engineering translation, not legal advice. When a finding hinges on interpretation (e.g. whether a use is a "legitimate use", Significant Data Fiduciary status, sector laws like RBI/IRDAI/health), say so and recommend legal review.

---

## 1. How to use this skill

Pick the mode from the user's request:

| Mode | Trigger | Output |
|---|---|---|
| **Design** | New feature/product, architecture, data model | Data inventory + controls list mapped to §5, plus open questions |
| **Build** | Writing code, schema, APIs, UI copy | Code/config that implements the control; cite S./R. in comments where it helps reviewers |
| **Review / Audit** | "Is this compliant?", PR, schema, existing app | Findings report using the template in §9 |

Always start with §3 (does DPDP apply, and in what role). Then walk §5 control by control. Don't dump the whole checklist — report only what's relevant to the artefact in front of you, but never skip Security (C8), Breach (C9) and Retention (C10) for any system that stores personal data.

---

## 2. Legal status and timeline (as of Sept 2026)

| Date | What's in force |
|---|---|
| 13 Nov 2025 | R.1, 2, 17–21: definitions, Data Protection Board setup, Board procedure |
| 13 Nov 2026 | R.4: Consent Manager registration and obligations |
| **13 May 2027** | R.3, 5–16, 22, 23: notice, security safeguards, breach intimation, retention/erasure, contact info, children, SDF duties, rights, cross-border, appeals — i.e. **almost every developer-facing obligation** |

A Jan 2026 MeitY consultation floated shortening the 18-month window; treat as unconfirmed unless a gazette notification says otherwise. Recommend building now: products shipped today will still be running in May 2027, and retrofitting consent and deletion into a live data model is expensive. If you can search, check for newer notifications before stating dates as final.

---

## 3. Scope and roles — answer these first

**Does the Act apply? (S.3)** Yes if the system processes *digital personal data* (data about an identifiable individual, S.2(t)) that is:
- processed in India (collected digitally, or collected on paper and digitised later), or
- processed outside India in connection with offering goods/services to people in India.

**Out of scope:** purely personal/domestic use; data the individual herself made public, or that someone was legally required to publish (S.3(c)).

**Partial exemption worth knowing (S.17(1)(d)):** an India-based company processing personal data of people *outside* India under a contract with a foreign party (typical offshore dev/outsourcing for GCC/US/EU clients) is exempt from most of Chapters II–III — **but S.8(1) (accountability) and S.8(5) (security safeguards) still apply.** The client's own law (GDPR, UAE PDPL, Saudi PDPL, etc.) will usually apply instead.

**Roles (S.2):**
- **Data Fiduciary (DF)** — decides purpose and means. Owns compliance, including for its processors (S.8(1)).
- **Data Processor (DP)** — processes on the DF's behalf. Needs a valid contract (S.8(2)); the DF must flow down security (R.6(1)(f)), erasure (S.8(7)(b)), consent withdrawal (S.6(6)) and retention (R.8(3)).
- **Data Principal** — the individual; for a child (<18) includes parent/guardian; for a person with disability, the lawful guardian (S.2(j)).
- **Significant Data Fiduciary (SDF)** — notified by government (S.10). Extra duties, see C14.

When building for a client, record who is DF and who is DP. An agency building a client's app is usually the DP; if the agency also runs analytics for its own purposes on that data, it becomes a DF for that purpose.

---

## 4. The seven principles (the lens for every decision)

Consent & transparency · purpose limitation · data minimisation · accuracy · storage limitation · security safeguards · accountability.

Practical test for any field, table, log or third-party call: *Which purpose needs this? What's the lawful basis? Who can see it? When is it deleted? Can we prove all of that?* If any answer is "don't know", that's a finding.

---

## 5. Controls: requirement → what to build → how to verify

### C1. Lawful basis for every purpose (S.4, S.7)
Processing is allowed only for a lawful purpose with **consent** or a **legitimate use**. Legitimate uses (S.7) that matter most to product teams:
- (a) the person voluntarily gave the data for a specific purpose and hasn't objected — e.g. phone number given to receive an order receipt. Stops when she says she no longer needs the service.
- (d)/(e) legal obligation or court order; (f)–(h) medical emergency, epidemic, disaster; (i) employment purposes (HR, payroll, preventing espionage/IP leakage).

**Build:** a *purpose registry* (code or config) — `purpose_id, description, lawful_basis (CONSENT | LEGIT_USE_7a | LEGIT_USE_7i | LEGAL_OBLIGATION …), data_items[], retention_rule, processors[]`. Every column that holds personal data maps to ≥1 purpose.
**Verify:** no personal-data field without a purpose; no purpose with basis "TBD"; marketing/analytics/AI-training are never riding on 7(a).

### C2. Notice before or with every consent request (S.5, R.3)
The notice must:
- stand alone — understandable without reading the privacy policy or T&Cs (R.3(a));
- itemise the personal data collected and state each specific purpose and the goods/services/uses it enables (R.3(b));
- give the link/means to **withdraw consent**, **exercise rights**, and **complain to the Data Protection Board** (S.5(1), R.3(c));
- be available in English **or any of the 22 Eighth Schedule languages** at the user's option (S.5(3));
- show contact of the DPO or authorised person (S.6(3)).

**Legacy users (S.5(2)):** users who consented before commencement must be sent a notice (email/in-app) as soon as reasonably practicable; processing may continue until they withdraw.

**Build:** versioned notice objects (`notice_id, version, language, data_items[], purposes[], links{withdraw, rights, board_complaint}, published_at`); i18n keys, not hardcoded strings; notice shown inline at the point of collection, not buried behind a link.
**Verify:** every form/SDK that collects personal data renders a notice version; language switch works; the three links resolve.

### C3. Valid consent (S.6(1)–(3))
Consent must be **free, specific, informed, unconditional, unambiguous, by clear affirmative action**, limited to data necessary for the purpose.
- One toggle per purpose; nothing pre-ticked; no "by continuing you agree".
- Don't bundle unnecessary data with the core service. Act's own example: a telemedicine app asking for the contact list — consent to contacts is not valid because it isn't needed for telemedicine (S.6(1) illustration).
- Any consent clause that breaks the law is void to that extent — e.g. "I waive my right to complain to the Board" (S.6(2)).
- Clear, plain language (S.6(3)).

**Verify:** try to use the core service while declining optional purposes — it must work. Look for dark patterns: pre-checked boxes, confirmshaming, consent hidden in T&Cs.

### C4. Consent ledger — proof is on you (S.6(10))
If challenged, the DF must prove notice was given and consent obtained properly.

**Build:** append-only, tamper-evident consent log:
```sql
consent_event(
  event_id        UUID PK,
  principal_id    ...,
  purpose_id      ...,          -- one row per purpose
  notice_id, notice_version, notice_language,
  action          ENUM('GIVEN','DENIED','WITHDRAWN'),
  method          ...,          -- 'checkbox_click', 'otp_confirm', 'consent_manager'
  channel         ...,          -- web/android/ios/api/consent_manager
  consent_manager_id NULL,
  occurred_at     TIMESTAMPTZ,
  client_meta     JSONB,        -- minimised: app version, not full fingerprint
  prev_hash, hash               -- hash chain or WORM storage
)
```
Current state = latest event per (principal, purpose). Never UPDATE/DELETE rows (except under the erasure rules in C10).
**Verify:** for a random user, you can show which notice version she saw, in which language, and what she clicked, per purpose.

### C5. Withdrawal as easy as giving (S.6(4)–(6))
- Withdraw in the same number of steps/channel as consent was given (one tap in → one tap out).
- On withdrawal: stop processing within reasonable time **and make processors stop** (S.6(6)); erase unless law requires retention (S.8(7)) — see C10 for the one-year log rule.
- Withdrawal doesn't unwind past lawful processing, and already-paid orders can still be fulfilled (S.6(5) illustration).

**Build:** `consent.withdrawn` event on a bus → consumers (marketing, analytics, CRM, processors' webhooks) unsubscribe/purge; per-purpose feature flags check current consent at runtime, not a cached signup-time value.
**Verify:** withdraw marketing consent → no emails after the propagation SLA; processor confirms.

### C6. Consent Managers (S.6(7)–(9), R.4, Sch.1)
Users may give/withdraw consent via a Board-registered Consent Manager (India-incorporated, ≥₹2 cr net worth, must not be able to read the data it routes, keeps consent records ≥7 years). Registration opens 13 Nov 2026.
**Build:** keep consent ingestion channel-agnostic (`channel = consent_manager`, `consent_manager_id`) so a CM integration later is an adapter, not a redesign.

### C7. Accuracy (S.8(3))
If data is used for a decision about the person (credit, eligibility, pricing, AI scoring) or shared with another DF, ensure completeness, accuracy and consistency.
**Build:** validation at input, source-of-truth per field, correction propagation to downstream copies.

### C8. Reasonable security safeguards (S.8(4)–(5), R.6) — highest penalty tier (₹250 cr)
Minimum controls under R.6(1):
| R.6(1) | Requirement | Engineering control |
|---|---|---|
| (a) | Encryption, obfuscation, masking, or tokens mapped to the data | TLS 1.2+ in transit; encryption at rest (KMS-managed keys); field-level encryption or tokenisation for high-risk fields (Aadhaar, financial, health, precise location, biometrics); masking in UIs, logs and non-prod |
| (b) | Access control to computer resources | SSO + MFA for staff, least-privilege RBAC/ABAC, service-to-service auth, no shared accounts, secrets in a vault, prod access by approved just-in-time grant |
| (c) | Visibility via logs, monitoring, review to detect/investigate/remediate unauthorised access | Audit log of *who accessed which principal's data, when, why*; alerts on bulk export / anomalous queries; periodic access reviews |
| (d) | Continuity if data is lost/compromised | Encrypted, tested backups; restore drills; defined RPO/RTO |
| (e) | Retain those logs and personal data **1 year** (unless another law says otherwise) | Log retention ≥ 365 days in tamper-resistant storage |
| (f) | Security terms in processor contracts | DPA with every vendor: cloud, email/SMS, analytics, support tools, LLM APIs |
| (g) | Technical + organisational measures | Secure SDLC, dependency scanning, secret scanning, pen tests, training, incident runbook |

Also apply common-sense hygiene the Rule implies: no personal data in URLs/query strings, no PII in application logs or error trackers, no production data in dev/test without masking, rate limiting and IDOR checks on every endpoint returning personal data.
**Verify:** grep logs and analytics payloads for emails/phones/IDs; attempt cross-tenant/IDOR access; check key rotation; confirm backup restore was tested.

### C9. Breach intimation (S.8(6), R.7) — ₹200 cr tier
A "personal data breach" is broad: any unauthorised processing, accidental disclosure, alteration, destruction or **loss of access** affecting confidentiality, integrity or availability (S.2(u)). Ransomware and prolonged outages of personal data count. There is **no materiality threshold** — every breach is reportable.

**To each affected Data Principal — without delay**, via her user account or registered contact, in plain language (R.7(1)):
(a) what happened — nature, extent, timing; (b) likely consequences for her; (c) mitigation done/in progress; (d) safety steps she can take; (e) business contact for queries.

**To the Board (R.7(2)):**
- *Without delay:* description — nature, extent, timing, location, likely impact.
- *Within 72 hours of becoming aware* (extendable on written request): updated details; facts, circumstances and reasons; mitigation; findings about who caused it; remediation to prevent recurrence; report on the intimations sent to principals.

**Build:** incident runbook with a 72-hour clock starting at awareness; ability to query "which principals were affected" (needs the access logs from C8); pre-drafted notification templates carrying all five R.7(1) fields; bulk notifier through in-app + registered email/SMS; processor contracts requiring the processor to notify you immediately.
**Verify:** tabletop exercise — can you produce the affected-user list and the Board report inside 72 hours?

### C10. Retention and erasure (S.8(7)–(8), S.12(3), R.8, Sch.3) — the tricky one
Three rules interact; design for all three:
1. **Erase when purpose ends or consent is withdrawn, whichever is earlier** — and make processors erase too — unless a law requires retention (S.8(7)). Example from the Act: a bank keeps KYC for 10 years because banking law says so.
2. **Inactivity deletion for large platforms (R.8(1)–(2), Sch.3):** e-commerce ≥2 crore registered users, online gaming ≥50 lakh, social media ≥2 crore → erase after **3 years** from the user's last interaction (or from rules commencement, whichever is latest), except data needed for account access or stored-value tokens/wallets. **Notify the user at least 48 hours before erasure**; logging in or contacting you resets the clock. Even if you're not in this class, an inactivity policy is good evidence of storage limitation.
3. **Minimum 1-year retention of personal data, traffic data and processing logs (R.8(3), R.6(1)(e))** from the date of processing, even if the user deletes her account — Rule's own example: an e-book platform keeps order, payment and delivery records for a year after account deletion. Then erase, unless another law needs longer.

**Build — two-tier deletion:**
- On account deletion / withdrawal / purpose end → immediately *remove from active systems* (profile, marketing, analytics, search indexes, caches, processors' active copies) and move the minimal transaction/log record into a **restricted legal-hold store** (encrypted, access-logged, no product use).
- Scheduled purge job deletes legal-hold records when `max(1 year from processing, other statutory period)` expires. Backups age out on a documented cycle.
- `retention_rule` lives on each purpose in the purpose registry; the purge job reads it — no ad-hoc per-table cron scripts.
- For Sch.3 platforms: `last_active_at` per user, 48-hour pre-erasure notice job, then erase.

**Verify:** delete a test account → gone from prod DB, search, analytics, CRM, email tool within SLA; present only in legal-hold with restricted access; purge job test with time-travel.

### C11. Data Principal rights (S.11–14, R.14)
| Right | Build |
|---|---|
| **Access** (S.11) — summary of data and processing activities; identities of all other DFs and processors it was shared with + description of what was shared | Self-serve "My data" page or export generated from the data inventory; maintain a **sharing register** (recipient, purpose, data items) — you can't answer this without one |
| **Correction / completion / update** (S.12(1)–(2)) | Editable profile + request flow for fields users can't self-edit; propagate to processors |
| **Erasure** (S.12(3)) | Delete-account flow feeding the C10 pipeline; refuse only where retention is needed for the purpose or by law, and say which |
| **Nomination** (S.14, R.14(4)) | Let the user nominate one or more people to act on death/incapacity; store nominee contact + verification method |
| **Grievance redressal** (S.13, R.14(3)) | Ticketed grievance channel; **publish your response timeline — max 90 days**; user must use this before going to the Board |

R.14(1): prominently publish on the site/app *how* to make a request and *which identifiers* are needed (username, email, customer ID, etc.). R.9: publish contact of DPO/authorised person and include it **in every response** to a rights request.
**Build:** `rights_request(id, principal_id, type, received_at, due_at = received_at + ≤90d, status, verified_by, response_sent_at)` with SLA alerts; identity verification proportional to risk (re-auth for logged-in users).
**Verify:** submit each request type end-to-end; responses include contact details; the SLA dashboard exists.

### C12. Children — under 18 (S.9, R.10, R.12, Sch.4) — ₹200 cr tier
- **Verifiable parental consent before processing any child's data** (S.9(1)). Verify the parent is an identifiable adult using either reliable identity/age details you already hold (parent is an existing verified user), or details voluntarily provided — directly or via a **virtual token from an authorised entity, e.g. DigiLocker** (R.10). R.10 illustrations cover four flows: child declares parent (parent registered/unregistered) and parent opens account for child (parent registered/unregistered).
- **Never:** processing likely to harm a child's well-being; **tracking or behavioural monitoring of children; targeted advertising directed at children** (S.9(2)–(3)).
- **Exemptions (Sch.4)** from parental consent and the tracking ban, each limited to what's necessary: clinical/mental health establishments and healthcare professionals for health services; educational institutions for educational activities and child safety; crèches/day care for safety; school transport for location tracking during travel; email-only accounts; real-time location for the child's safety; blocking harmful content/ads; and **age verification itself**.

**Build:** neutral age gate (don't nudge users to lie); child-mode flag that disables analytics SDKs, ad IDs, personalisation and recommendations; parental-consent flow with DigiLocker/token option; consent ledger records parent identity-verification method.
**Verify:** sign up as a 16-year-old → no ad SDK calls, no behavioural events leave the device, parental consent required before account creation.

### C13. Persons with disability with a lawful guardian (S.9(1), R.11)
Consent comes from the guardian; verify the guardian was appointed by a court, a designated authority under the RPwD Act 2016, or a local level committee under the National Trust Act 1999. Store the verification reference.

### C14. Significant Data Fiduciary extras (S.10, R.13) — ₹150 cr tier
Only if notified by government (based on volume/sensitivity, risk to rights, sovereignty, elections, security, public order). Then:
- DPO **based in India**, reporting to the board, point of contact for grievances;
- independent data auditor; **DPIA + audit every 12 months**, report of significant observations to the Board;
- due diligence that **algorithmic software** (hosting, display, upload, sharing, recommendations) doesn't risk users' rights — relevant for AI/ML features;
- data localisation: personal data and its traffic data in government-specified categories must not leave India (R.13(4)).
**Build even if not an SDF:** keep a lightweight DPIA template per major feature; it's cheap and makes SDF status survivable.

### C15. Cross-border transfer (S.16, R.15)
Transfers outside India are allowed by default, **except** to countries the government restricts, and subject to any requirements it sets for making data available to foreign states or their agencies. Sector laws (e.g. RBI payment data localisation) can be stricter and still apply (S.16(2)).
**Build:** know and document the region of every datastore and vendor (cloud region, LLM API, email/SMS, analytics, support tools); keep region configurable so an India-only deployment is a config change.
**Verify:** data-flow diagram lists every region.

### C16. Processors and third parties (S.8(1)–(2), (7)(b))
Every vendor touching personal data needs a written contract covering purpose limitation, security (R.6(1)(f)), breach notification to you, erasure on instruction, 1-year log retention (R.8(3)), and sub-processor disclosure. Third-party SDKs (analytics, ads, crash reporting, chat widgets, session replay) are processors or separate DFs — inventory them and gate them behind consent where the basis is consent.

### C17. Exemptions — don't assume them
- Startups (S.17(3)): government *may* notify exemptions from notice, accuracy, erasure, SDF and access duties. **None apply unless notified** — don't design around a hoped-for exemption.
- Research/archiving/statistics (S.17(2)(b), R.16, Sch.2): exempt only if no decision is taken about the individual and Sch.2 standards (lawful, necessary, minimised, accurate, retained only as needed, secured, accountable) are followed.
- Legal claims, investigations, court functions, approved mergers, loan-default asset checks (S.17(1)): exempt from most obligations — **but security (S.8(5)) still applies**.

---

## 6. Data inventory — the artefact everything else depends on

Create or update this before anything else in Design or Audit mode:

| Field / dataset | Personal? | Category (identity, contact, financial, health, location, biometric, device, behavioural, child) | Purpose(s) | Lawful basis | Source | Stored where (system, region) | Shared with | Retention rule | Encrypted/masked? |
|---|---|---|---|---|---|---|---|---|---|

Rules of thumb: IP addresses, device IDs, precise location, sensor/camera data tied to a person, and pseudonymous IDs you can link back are personal data. Aggregated, truly anonymised data is not — but be sceptical of "anonymised" claims on small cohorts.

---

## 7. SDLC checklist

**Design** — inventory (§6) · purpose registry · DF/DP roles · lawful basis per purpose · children possible? · vendors & regions · retention per purpose · DPIA for high-risk features (AI decisions, location, health, children).

**Build** — standalone multilingual notice · granular un-ticked consent · consent ledger · runtime consent checks · withdrawal ≤ consent effort · encryption/tokenisation/masking · RBAC + MFA · access audit logs (≥1 yr) · no PII in logs/URLs/non-prod · rights APIs + 90-day SLA · delete-account → two-tier deletion · age gate + child mode · processor DPAs.

**Test** — decline-optional-consent path · withdraw → propagation · export my data includes the sharing register · delete account → verify all stores · IDOR/cross-tenant tests · log scan for PII · child sign-up path · breach tabletop (72 h).

**Release** — privacy notice + contact/DPO + grievance timeline + rights "how-to" published (R.9, R.14) · legacy users re-noticed (S.5(2)) · runbooks owned.

**Operate** — access reviews · purge job monitoring · backup restore drills · vendor re-assessment · annual DPIA/audit if SDF · watch for new MeitY notifications.

---

## 8. Red flags to call out immediately

Pre-ticked or bundled consent · "by using this app you agree" · privacy notice only in the policy page · no withdraw option or withdrawal buried in email-support · personal data in logs, URLs, Slack or analytics events · production dumps in dev/staging · soft-delete flag treated as erasure (data still used) · hard-deleting everything instantly including transaction logs (breaks the 1-year rule) · no record of which vendors receive data · ad/analytics SDKs active for minors · contact list, location or camera requested without a purpose that needs it · shared admin accounts · no way to find out which users a breach affected · LLM/AI vendor receiving personal data with no contract or region check.

---

## 9. Review / audit report template

Use this structure for Review mode:

```
# DPDP review — <system/PR/feature>
Scope: <what was reviewed> · Role: <DF / DP / both> · Applies because: <S.3 reason>
Children possible: <yes/no> · SDF: <no / notified / unknown>

## Summary
<2–4 sentences: overall posture and the top risks>

## Findings
| # | Control | Finding | Evidence (file:line / screen / table) | Law | Severity | Fix |
|---|---|---|---|---|---|---|
| 1 | C8 Security | Aadhaar stored in plaintext | users.aadhaar_no | S.8(5), R.6(1)(a) | Critical | Tokenise; KMS field encryption |

## Passing controls
<short list>

## Open questions for the team / legal
<interpretation calls, missing info>
```

Severity guide (tied to the S.33 Schedule penalty tiers so priorities match legal exposure):
- **Critical** — security safeguards (up to ₹250 cr), breach-notification capability (₹200 cr), children's obligations (₹200 cr)
- **High** — SDF obligations (₹150 cr); consent validity, notice, erasure, rights, processor contracts (general tier up to ₹50 cr, but core to lawful processing)
- **Medium** — documentation, SLAs, publication gaps
- **Low** — hygiene / best practice beyond the text

---

## 10. Penalty schedule (S.33, Schedule) — for context, not scare tactics

| Breach | Max penalty |
|---|---|
| Failure to take reasonable security safeguards (S.8(5)) | ₹250 crore |
| Failure to notify Board/principals of a breach (S.8(6)) | ₹200 crore |
| Children's obligations (S.9) | ₹200 crore |
| SDF additional obligations (S.10) | ₹150 crore |
| Breach of a voluntary undertaking (S.32) | Up to the applicable amount for the original breach |
| Any other provision of Act or Rules | ₹50 crore |
| Data Principal duties (S.15 — e.g. impersonation, false complaints) | ₹10,000 |

The Board weighs gravity, duration, data type, repetition, gain, **timeliness and effectiveness of mitigation**, proportionality (S.33(2)) — which is why logs, runbooks and documented controls matter. Two or more penalties can lead to government-ordered blocking of the service (S.37).

---

## 11. Style when using this skill

- Cite the specific section/rule for every requirement so reviewers can check it.
- Prefer concrete fixes (schema change, code snippet, config) over restating the law.
- Separate "the law requires" from "good practice" — don't inflate best practices into legal mandates.
- Flag interpretation calls and sector-specific laws (RBI, SEBI, IRDAI, health, telecom) for legal review rather than guessing.
- Before stating current enforcement dates or exemptions as settled, check for newer MeitY notifications if search is available.
