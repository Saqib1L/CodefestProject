# Estate Unlock (working title)

**Civic tech track · Malaysia · Project explanation · 11 October 2026**

A guided, trackable path for Malaysian families to get a deceased relative's assets released, with a progress bar that shows how close they are to unfreezing them.

---

## 1. One-line pitch

When someone dies in Malaysia, their assets freeze until an heir starts a legal process that most families do not understand. Estate Unlock tells a family which route applies, calculates and explains each heir's share, prepares the paperwork, and tracks the case to the point where every asset is actually released.

## 2. The problem

**What happens at death.** Assets in the deceased's name are frozen. Nothing can be sold, transferred or withdrawn until someone obtains the right authority from JKPTG, Amanah Raya or the High Court. The state does not go looking for these estates: the system cannot track an estate unless the heirs file an application.

**Scale (cite carefully).**
- More than RM90 billion in inheritance assets is widely reported as frozen. The figure has been repeated since around 2021 and mostly concerns Muslim estates. It is an estimate, not an official statistic, and other estimates exist. Present it as "more than RM90 billion, widely reported, estimates vary".
- JKPTG reported 84,384 small estate cases resolved in 2025, worth RM19.01 billion, with 46,707 still unresolved on 31 December 2025.
- The standard time for a complete, undisputed small estate application is four months. Real cases often run much longer.

**Why families get stuck (per JKPTG and researchers).**
- Heirs delay applying, often because discussing death feels improper.
- Multi-generation deaths: an heir dies before the estate is settled.
- Incomplete documents and weak knowledge of the procedure.
- Heirs refusing to attend hearings, and heirs who cannot be traced.
- Disputes over the split.
- Not knowing which of three routes applies.

## 3. What we are solving, and what we are not

| We address | We do not claim to fix |
|---|---|
| Not knowing where to start | Heir disputes |
| Choosing the wrong route | Court and land office backlogs |
| Incomplete or wrong paperwork | Assets with no heirs |
| No visibility of progress | East Malaysia (out of MVP scope) |
| Not knowing what is owed to whom | Legal advice |

The honest framing for judges: the tool removes the largest barrier, which is **starting and getting it right**. It does not unlock RM90 billion by itself.

## 4. Users

- **Primary:** a family member (often not the eldest) facing a death with no estate planning.
- **Secondary:** elderly people and others planning ahead through the estate checkup.
- **Handoff targets:** lawyers, mediators, Amanah Raya, JKPTG offices.

## 5. Product

| # | Module | What it does |
|---|---|---|
| 1 | Guided intake | Plain-language questions in BM and English. The family can type or speak in their own words. |
| 2 | Route finder | Applies the decision rules and shows the route with the reason. |
| 3 | Share calculator | Works out each heir's share and shows the rule behind every number. |
| 4 | Paperwork builder | Personalised checklist and a ready-to-upload data pack for the chosen route. The user submits it; the app does not file on their behalf. |
| 5 | Case tracker | Steps, evidence, expected waiting windows, stall alerts, and a progress percentage. |
| 6 | Estate checkup | Ongoing prompts to write a will, hibah or nomination as life events happen. This gives the product repeat use. |
| 7 | Dispute and lawyer handoff | When the case needs a lawyer or mediator, the app says so and hands over a clean summary. |

## 6. How the real process works

The stages are the same for every religion and route. Religion changes how shares are calculated and adds a few Muslim-only steps.

### 6.1 Route rules (evaluate in order)

1. Remove assets that sit outside the estate: EPF or insurance with a nomination, and jointly held assets that pass by survivorship.
2. Non-Muslim with a will under the Wills Act 1959: High Court probate, any value.
3. Muslim with a wasiat or hibah: honour it first, up to one third, then faraid for the rest.
4. No will, estate up to RM5 million: JKPTG small estate (filed on MyLand).
5. No will, movable assets only, up to RM600,000: Amanah Raya, or JKPTG.
6. Above RM5 million, or contested or complex: High Court Letters of Administration, lawyer required.

The RM5 million small estate limit took effect on 15 July 2024. It was RM2 million before, and many guides still say RM2 million, which is exactly why the rules library stores dated, sourced rules.

### 6.2 Phases and progress weights

| Phase | What | Weight | Cumulative |
|---|---|---|---|
| 1 | Prepare: death certificate, will check, heirs, assets, values | 15% | 15% |
| 2 | Choose route and applicant | 10% | 25% |
| 3 | File | 15% | 40% |
| 4 | Review and hearing | 20% | 60% |
| 5 | Order or grant issued | 15% | 75% |
| 6 | Execute transfers, asset by asset | 25% | 100% |

The weights are a product decision, not official numbers.

### 6.3 Progress rules

- The bar moves only on evidence (receipt, notice, order PDF), never on elapsed time.
- Phase 6 is per asset: progress = 25% x (assets transferred / assets listed). Show "assets unlocked: 2 of 5" next to the percentage.
- Never show 100% until every listed asset is transferred.
- Adding a missed asset later lowers the percentage, and the app explains why (JKPTG's follow-up application, Form P).
- Waiting steps show an expected window and trigger a stall alert when exceeded.

### 6.4 Highlights per route

- **JKPTG small estate:** MyLand account, Form A petition signed before a Magistrate or Commissioner for Oaths, document upload, hearing notice by email, hearing where all heirs state the method (faraid, Distribution Act 1958, or voluntary agreement of all heirs), then a Distribution Order or Letter of Administration within 30 days of the hearing ending.
- **Amanah Raya:** open a file at a branch, Amanah Raya verifies heirs and values, receives authority, and claims assets from each agency.
- **High Court:** lawyer files an originating summons and affidavit, heirs agree an administrator and sign consents, sureties are arranged unless waived, hearing, then a Grant of Probate or Letters of Administration.

The complete step list, 20 special cases (multi-generation deaths, minors, heirs abroad, untraceable heirs, loans, joint property, missed assets and more) and per-asset transfer guidance are in the companion document, "Malaysia estate unlock: complete case workflow".

## 7. Technical design

### 7.1 Architecture

Five layers, top to bottom:

1. **Experience:** guided intake, share breakdown, case tracker, estate checkup.
2. **AI assist:** story-to-facts extraction, document reader, plain-language explainer, output guard.
3. **Deterministic core:** route finder, faraid and intestate share engine, versioned rules library, form builder.
4. **Platform:** encrypted case database, scheduler and alerts, auth and consent, audit log.
5. **Outside world:** JKPTG and courts (mocked in the demo), WhatsApp or SMS, lawyer referral, official source updates.

### 7.2 The key design decision

**The AI never calculates.** Every legal decision (route, share, required form) is made by plain, testable code. The AI layer only converts messy human input into a fixed schema and explains results afterwards.

Why:
- A model that is right 98% of the time can still get a widow's share wrong, which causes family disputes.
- Deterministic logic is auditable and repeatable, and can be tested against worked examples.
- It answers the judge's question: "isn't this just ChatGPT with a form?"

### 7.3 Trust boundary

For each case: the family tells the story, the AI extracts structured facts, **the family confirms or edits those facts**, and only confirmed facts reach the rules engine. The output guard rejects anything that fails schema validation or comes back with low confidence.

### 7.4 Rules as data

Faraid rules, thresholds, form requirements and agency details live in a versioned rules library. Each rule carries its source, a date and a reviewer's sign-off. New routes can be added and thresholds updated without redeploying the engine.

### 7.5 Reasoning trail

The engine returns a trail, not just numbers: which rule applied, why, and what changed because a particular heir was present or absent. Families trust results they can follow, and a lawyer can check them quickly.

### 7.6 Suggested stack

| Part | Choice | Why |
|---|---|---|
| Frontend | Next.js PWA, bilingual (BM/EN) | Works on any phone, no install, low bandwidth |
| Backend | Python FastAPI | Strong for rules logic, OCR and PDF generation |
| Rules engine | Pure functions plus versioned JSON or YAML rule files | Testable and reviewable, no AI involved |
| AI | Claude via API with schema-constrained output | Extraction and explanation only |
| Data | PostgreSQL | Heirs, assets and cases are relational |
| Documents | PDF templates filled programmatically | Checklists and pre-filled data packs |
| Jobs | Scheduler plus WhatsApp or SMS provider | Reminders and stall alerts |
| Login | Mocked MyDigital ID flow | Shows the real integration path without needing access |

### 7.7 Privacy and security

Estate data is highly sensitive. Collect only what a calculation needs, encrypt at rest, record explicit consent, keep an audit log of every calculation, host in a Malaysian cloud region, and align with the PDPA.

## 8. Demo plan (five minutes)

Use one realistic family, for example: a father dies without a will, leaving a house and savings, an EPF account without a nomination, ASNB units and a car. Survivors are a widow, two sons and a daughter, and one son lives abroad.

1. The family describes the situation in their own words. The AI extracts facts and the family corrects one of them.
2. The route finder shows the JKPTG small estate route and explains why.
3. The share engine shows each heir's share with the rule behind it.
4. The app generates the checklist and data pack, and flags the heir abroad needing a consent form for the hearing.
5. The tracker shows progress at a given milestone with a stall alert example.
6. The estate checkup shows how the same family could have avoided the freeze.

State clearly which parts are real and which are mocked.

## 9. How this maps to the judging criteria

| Criterion (prelim / final weight) | Our argument |
|---|---|
| Impact (30% / 30%) | A national, recurring failure with real numbers: tens of thousands of cases pending, a widely reported RM90 billion+ frozen, and a state process that cannot find estates on its own. |
| Tech (30% / 25%) | A deterministic legal rules engine with reasoning trails, an AI extraction layer behind a validation boundary, a versioned and sourced rules library, and evidence-based progress tracking. |
| Usability (25% / 25%) | Plain-language, bilingual guided flow, a single progress percentage, and checklists that replace guesswork for people with no legal background. |
| Feasibility (15% / 15%) | Works without waiting for government APIs; mocks only agency status. Honest scoping, with a clear route to JKPTG or Amanah Raya partnership. |

## 10. Why this idea (and what we dropped)

The idea came out of a filter the team set: civic tech only (not finance, health, cybersecurity or education), citizen-facing, a fix not just a warning, national scale, and not something a search engine already does.

Dropped along the way: scam detection (cybersecurity track), debt early warning (finance track), flood prediction (it warns but cannot stop the flood), a generic price comparison app (finance and already crowded), and a unified government login (MyGOV and MyDigital ID already shipped it).

The frequency concern, that a person only faces this once, is answered by the estate checkup module, which brings people back at each life event.

## 11. Four-week build plan

| Week | Focus |
|---|---|
| 1 | Rules engine and test suite, validated by a syarie lawyer or practitioner. Everything depends on this being correct. |
| 2 | Intake, AI extraction, family confirmation step, output guard. |
| 3 | Paperwork builder, case tracker with progress model, explainer. |
| 4 | Estate checkup, polish, mocked agency integrations, demo rehearsal. |

## 12. Risks and how to handle them

| Risk | Handling |
|---|---|
| Legal accuracy | Expert review of the rules engine; frame the app as guidance and drafting help, not legal advice. |
| Outdated rules | Dated, sourced rules with an owner; show the rule version in the app. |
| Disputes | The app cannot resolve them. Say so, and provide a lawyer or mediator handoff. |
| Sensitive data | Data minimisation, encryption, consent, audit log, PDPA alignment. |
| Over-claiming impact | Use cautious wording for the RM90 billion estimate and present the tool as removing the biggest barrier, not unlocking everything. |
| Scope creep | Peninsular Malaysia only for the MVP; East Malaysia is a later rule set. |

## 13. Open items to verify before launch

1. Whether a printed copy of Form A must follow the online MyLand filing.
2. Whether the JKPTG hearing replaces a Syariah Court faraid certificate, and which agencies insist on the certificate.
3. The exact JKPTG procedure when heirs disagree.
4. How multi-generation deaths, minors and untraceable heirs are filed.
5. Current order fees and Amanah Raya charges.
6. Stamp duty and registration fees for transfer by inheritance, including any Budget 2027 changes.
7. The order of deductions before faraid is applied.
8. Whether a Muslim wasiat changes which court issues the grant.
9. East Malaysia rules (Sabah and Sarawak statutes and native customary law).

## 14. Sources

- MyGov: Division of Inheritance, Small Inheritance Application Procedure, Procedure for Applying for a Large Inheritance, Important Things Before Dividing Inheritance for Muslims (malaysia.gov.my)
- JKPTG: Pembahagian Harta Pusaka and Pusaka Kecil FAQ (jkptg.gov.my)
- Utusan, "84,384 kes harta pusaka kecil selesai tahun 2025" (January 2026)
- Sinar Harian, on the RM5 million limit effective 15 July 2024
- Distribution Act 1958 summaries: Johor Bar, Chia Lee and Associates, Malaysian Bar
- Faraid and wasiat: Conventus Law, Azmi and Associates
- EPF, insurance and bank accounts: Conventus Law, SAYS, RinggitPlus
- Practitioner guides on route, timelines and costs: RinggitWise, Clickbina, AskLegal, PropertyGuru, IQI

*This document is research and planning material, not legal advice.*
