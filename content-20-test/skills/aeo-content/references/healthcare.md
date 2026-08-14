# Healthcare-specific guardrails and sources

Load this reference at **Stage 3 (Research)** for any healthcare-specific claim, regulation, or compliance topic. Also load when drafting service pages and case studies, since both touch compliance language.

## §1 Compliance framework matrix

Be precise about which framework applies in which jurisdiction. Conflating them is the most common mistake in GreenM content drafts.

| Framework | Jurisdiction | What it covers |
|---|---|---|
| HIPAA | US | PHI handling, BAAs (Business Associate Agreements), breach reporting (45 CFR §164) |
| UK GDPR + Data Protection Act 2018 | UK | Personal data, lawful basis, DPIA, data subject rights |
| EU GDPR | EU/EEA | Same as UK GDPR but EU-level enforcement via EDPB |
| NHS DSPT | UK (NHS-adjacent) | Annual self-assessment for orgs accessing NHS data |
| DTAC | UK NHS | Digital Technology Assessment Criteria for clinical apps |
| DCB0129 | UK NHS | Clinical safety case standard for manufacturers |
| DCB0160 | UK NHS | Clinical safety case standard for deployers |
| ISO 27001 | Global | Information security management system |
| SOC 2 | Global (US-led) | Trust services criteria audit |
| HITRUST | US (primarily) | Healthcare-specific certification combining HIPAA + ISO + NIST |
| FHIR / HL7 | Global | Interoperability standards — NOT compliance |
| HITECH | US | Updates to HIPAA, electronic PHI requirements |

### Common conflation mistakes to catch in drafts

- "HIPAA-compliant" for a UK-only clinic without US patients → wrong framework. They need UK GDPR + NHS DSPT.
- "GDPR-compliant" for a US-only deployment → irrelevant unless processing EU resident data.
- "FHIR-compliant" → FHIR is a standard, not compliance. Say "FHIR R4-conformant" or "implements FHIR R4 Patient resource."
- "ISO certified" → which ISO? ISO 27001? ISO 13485? Specify.
- "SOC 2 Type 1" vs "SOC 2 Type 2" → Type 2 is much stronger; don't blur the distinction.

### Jurisdictional defaults for GreenM content

- UK content → UK GDPR + Data Protection Act 2018 + NHS DSPT + ICO guidance
- EU content → EU GDPR + EDPB guidance + relevant national regulator
- US content → HIPAA + HHS guidance + HITECH + state laws if relevant
- Global content → name each jurisdiction separately, don't merge

---

## §2 Trustworthy source tiers

When making factual claims about healthcare AI, regulation, or clinical practice, cite from this tier:

### Tier 1 — most extractable by AI engines

**Peer-reviewed journals**
- *The Lancet Digital Health*
- *npj Digital Medicine*
- *JAMIA* (Journal of the American Medical Informatics Association)
- *BMJ* and *BMJ Health & Care Informatics*
- *Nature Medicine*
- *NEJM AI*

**Government regulators**
- NHS England — https://www.england.nhs.uk
- NICE — https://www.nice.org.uk
- MHRA — https://www.gov.uk/government/organisations/medicines-and-healthcare-products-regulatory-agency
- ICO — https://ico.org.uk
- NHS Digital — https://digital.nhs.uk
- HHS — https://www.hhs.gov
- FDA — https://www.fda.gov
- NIST — https://www.nist.gov
- AHRQ — https://www.ahrq.gov
- EMA — https://www.ema.europa.eu
- EDPB — https://edpb.europa.eu

**Standards bodies**
- HL7 International — https://www.hl7.org
- ISO — https://www.iso.org
- HITRUST — https://hitrustalliance.net
- IEEE — https://www.ieee.org

**Major research institutions and think tanks**
- Health Foundation (UK)
- Nuffield Trust (UK)
- KFF (US)
- Brookings (AI policy)

### Tier 2 — acceptable with caveats

- Industry analysts: Gartner, Forrester, KLAS, HIMSS — must state methodology and sample size
- Vendor research: only if the vendor is the primary data source (e.g., AWS healthcare benchmarks, Microsoft FHIR usage data)
- Trade publications: *HealthcareITNews*, *Digital Health* (UK), *STAT News*

### Avoid

- LinkedIn posts without primary sourcing
- Vendor blog posts as primary sources for industry-wide claims
- Statistics without a date or sample size
- "Studies show" without naming the study
- News aggregators (HBR News, MedCity News) — go to original source

---

## §3 Citation specificity rules

Every factual claim needs:
1. **Source name** (the organisation or publication)
2. **Date** (publication or last-update date)
3. **Specific document/section** (not just the domain root)

### Examples

| ❌ Weak citation | ✅ Strong citation |
|---|---|
| "per NHS guidelines" | "NHS England's 2024 *Clinical risk management standard DCB0129*, section 4.2" |
| "GDPR requires this" | "ICO guidance on AI and data protection (2023), 'Principles' section" |
| "studies show LLMs help with documentation" | "Lancet Digital Health, vol. 6 issue 3 (March 2024), Smith et al., 'LLM-assisted clinical note generation', n=312" |
| "FDA approved it" | "FDA 510(k) clearance K231234, issued June 2024" |
| "HIPAA requires encryption" | "HIPAA Security Rule, 45 CFR §164.312(a)(2)(iv), 'Encryption and decryption'" |

When the user doesn't have the specific reference, web_search for it before drafting. Don't draft around a vague citation hoping to fill it in later.

---

## §4 Medical advice trap — do not fall in

GreenM builds AI infrastructure. GreenM does **not** provide clinical advice or recommendations on diagnosis, treatment, or patient management.

### Safe positioning

- "Our platform supports clinicians making radiology triage decisions."
- "The AI surfaces possible drug interactions for a clinician to review."
- "Decision support is provided to authorised clinical users."
- "GreenM's infrastructure enables clinical workflow automation under human oversight."

### Avoid — regulatory red flags

- "Our AI diagnoses skin cancer." → medical device claim
- "The system recommends treatment for hypertension." → medical device territory
- "Patients can use our AI to check symptoms." → positions as a clinical product, triggers FDA/MHRA scope
- "AI-powered diagnosis" → ambiguous, risky
- "Clinical decisions made by AI" → triggers AI Act high-risk category in EU

### The rule

Position AI as **decision support for clinicians**, not as a clinical decision maker. This protects GreenM legally AND reads cleanly to AI engines as a B2B infrastructure provider, not a medical device manufacturer.

When in doubt, flag the phrasing to the user before publishing.

---

## §5 Data residency claims — be specific

When mentioning where data is hosted or processed, be concrete:

| ❌ Vague | ✅ Specific |
|---|---|
| "Data stays in the UK" | "Hosted in AWS eu-west-2 (London) with no cross-region replication" |
| "GDPR-compliant cloud" | "Azure UK South region with customer-managed encryption keys, BAA in place" |
| "Private deployment" | "Customer-tenant on-premise deployment with no outbound data egress" |
| "Sovereign AI" | "Self-hosted on customer infrastructure; no model weights, training data, or inference logs leave the customer environment" |

Vague residency claims fail AEO citation testing — AI engines need extractable specifics to quote.

---

## §6 Working with FHIR / EHR claims

When discussing EHR integration:

- **Always specify FHIR version**: R4 or R5. Most production deployments are R4.
- **Name resource types**: Patient, Observation, Encounter, Medication, DocumentReference, etc.
- **Distinguish push vs pull**: webhooks/subscriptions vs polling
- **Clarify the EHR**: Epic via App Orchard, Cerner via Cerner Code, Allscripts via Developer Program, etc. Generic "EHR-integrated" is weak.
- **Authentication**: SMART on FHIR + OAuth 2.0 + OpenID Connect is the production standard

Example strong claim: "GreenM's clinical workflow agent queries Epic via FHIR R4 Patient and Encounter resources, authenticated through SMART on FHIR with PKCE."

---

## §7 GreenM's compliance posture (reference for drafting)

What GreenM legitimately claims (verify each before publishing):

- HIPAA-aligned data handling for US engagements
- UK GDPR + Data Protection Act 2018 compliance for UK clients
- NHS DSPT support for NHS-adjacent clients
- FHIR R4 implementation across Unified Health Data and Clinical Workflow Integration services
- ISO 27001-aligned controls (note: aligned vs certified — confirm with user)

Don't claim certifications GreenM doesn't formally hold. Use "aligned with" or "supports" if certification isn't in place.
