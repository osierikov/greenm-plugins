# Case Study template — gotchas & fixes

Template page `6a5628d1699b78cee3a45075` (`detail_case-studies`). Classes use the `cstpl-` prefix (template-specific, editable — NOT the protected design-system classes).

## 1. Webflow Rich Text silent sanitization — THE BIG ONE
Webflow silently drops `<ul>`, `<ol>`, `<hr>`, and any `<a>` inside them when submitted HTML has newlines/whitespace **between top-level tags**. The create/update response echoes input verbatim, so the bug is invisible until you re-read the item or open the live page.

Fix — collapse inter-tag whitespace before submitting each Rich Text field:
```python
html = re.sub(r'>\s+<', '><', html)
```
Emit every Rich Text field as one continuous string. Detect: after each write, re-read via `list_collection_items` and grep for `<ul>`, `<ol>`, `<a `, `<blockquote>`, `<hr` — anything in input but missing in storage got sanitized.

## 2. Empty stat cards (hide + stretch)
Stats bind CMS text directly onto `cstpl-stat-val` (number) and `cstpl-stat-lab` (label); an empty field gets `w-dyn-bind-empty`. Template head CSS (Page Settings → Custom Code → head):
```css
.cstpl-stat-val.w-dyn-bind-empty,
.cstpl-stat-lab.w-dyn-bind-empty { display: none; }
.cstpl-stat:has(.cstpl-stat-val.w-dyn-bind-empty):has(.cstpl-stat-lab.w-dyn-bind-empty) { display: none; }
```
A card hides only when BOTH number and label are empty; a label-only card is a valid badge and stays. Container `cstpl-stats` uses `grid-template-columns: repeat(auto-fit, minmax(200px, 1fr))` (was fixed 4×`1fr`, which left a gap when a card hid) so visible cards stretch full width; `tiny` = 1 column.

## 3. Empty delivery phases (hide)
`cstpl-grid3` = flex (wrap, gap); cards `cstpl-rich-text-delivery-card` = `flex: 1 1 0` + min-width 260px. Empty phases hidden via head CSS:
```css
.cstpl-rich-text-delivery-card.w-dyn-bind-empty { display: none; }
```
Fields: `delivery`, `delivery-phase-2-2`, `delivery-phase-3-2`, `delivery-phase-4` (optional). Leave phase 4 empty for 3-phase cases.

## 4. Reference items must be non-draft
Services Provided (`services-provided`) and Security Logos (`security-logos-2`) render only when the referenced items are NOT Draft.

## 5. Head CSS/schema don't apply in the Designer canvas
Custom head code (the CSS above, JSON-LD schema) runs only on published/preview — never in the editor canvas. Verify on staging (`greenm.webflow.io`).

## 6. Field limit 60/60
Collection is full. Do not create fields — fill `fieldData` only. The legacy `outcomes-quote-text-author-role` slot is spare (outcomes quote now lives inside `outcomes` as `<blockquote>`).

## 7. "Changes in draft" after any edit
Editing an item flips it to "Changes in draft". To see it on staging, set `isDraft:false` and publish the site to the subdomain (`data_sites_tool.publish_site`, `publishToWebflowSubdomain:true`, `customDomains:[]`).

## 8. MCP can't write page head code
The Webflow MCP does not write a page's custom head code. The stat/delivery CSS and JSON-LD schema are added manually in Designer Page Settings. If a fresh template lacks them, flag it with the snippets above.

## 9. External link target
The client "Visit website" link (`cstpl-link`, bound to `client-link-url`) opens in a new tab via a template head script setting `target="_blank"` + `rel="noopener noreferrer"` on `.cstpl-link` (the link is a component-property binding, so the per-instance "Open in new tab" checkbox is unavailable).
