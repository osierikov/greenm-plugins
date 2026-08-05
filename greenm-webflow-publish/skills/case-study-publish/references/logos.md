# Shared logo library (`Logos/`)

Case studies pull logos from a shared library at `Lucy - growth/docs/content/case studies/Logos/`, not (only) each case's `imgs/`. Three kinds, three publishing rules. Logos are **never generated** — real brand/trust assets.

## Clients — `Logos/Clients/`
Per-case client logo. File: `{Client} logo.png` (e.g. `NRC health logo.png`). The skill matches by client name, uploads it, and sets the `client-logo` image field. A `client-logo.*` in the case's own `imgs/` overrides the shared one. If no match is found, stop and ask — never generate a client logo.

## Compliance / trust badges — `Logos/commpliance/` → Security Logos collection
The 10 SVGs here are the source of the **Security Logos** CMS collection (`6a59f8908cbfa8b087d3baf3`): aws, cloud infrastructure, private infrastructure, fhir, hipaa, gdpr, iso 27001, soc 2, dpa 2018, dspt. Uploaded once as collection items (non-draft). Per case: select the applicable badges and set the `security-logos-2` MultiReference. Rendered in the security/trust grid. Reuse the collection items — do not re-upload per case.

## Technology — text pills, not logos

Technology is published as **text pills** in the `tech-stack` Rich Text field: a flat `<ul>` of tool names, **core tech only, no SaaS** (e.g. no Front). We deliberately do NOT publish technology logos as images.

Why pills, not a logo grid: the source cases were inconsistent (some icons, some text-tags, most nothing); pills are one text field you type into and style once; logos would need a per-tech logo library, a dedicated reference collection, and a Collection List grid in Designer — and Case Studies is at 60/60 fields, so an extra reference field has no room. Pills cost nothing extra and read well for AEO.

A `Logos/Tech/` folder, if present, is a **reference/source library only** — its images are not placed on the page.

## Never generate

Client logos and compliance badges are real assets from this library — never generated. Technology is text (`tech-stack`), not an image. Krea generation (Step 2b) covers Hero/OG/section illustrations only.
