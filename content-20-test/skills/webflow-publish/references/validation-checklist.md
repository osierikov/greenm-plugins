# Validation checklist + troubleshooting

## Post-CMS-write sanitization check (CRITICAL)

Run this **immediately after** every `create_collection_items` and `update_collection_items` call. Webflow Rich Text API silently sanitizes body content with inter-tag whitespace — the API response echoes your input verbatim, masking the bug. Detection requires re-reading the stored item.

```python
# After create/update, fetch the item back
result = list_collection_items(collection_id, slug=slug)
stored_body = result['items'][0]['fieldData']['post-body']

submitted_body = the_html_you_submitted

# Critical element counts must match
discrepancies = []
for tag in ['<a ', '<ul>', '<ol>', '<hr', '<li>', '<div class="w-embed"']:
    submitted = submitted_body.count(tag)
    stored = stored_body.count(tag)
    if submitted != stored:
        discrepancies.append(f"{tag}: submitted={submitted}, stored={stored}")

# Extra: if body had any tables, they MUST be inside embeds
table_count = submitted_body.count('<table>')
embed_count = submitted_body.count('<div class="w-embed"')
if table_count > embed_count:
    discrepancies.append(f"tables outside embed: {table_count - embed_count} bare <table> — Webflow will strip them. Re-run Step 4.6 (transform_tables).")

# Extra: every body <img> must be inside an embed for full-width proportional display
import re
bare_img = 0
for m in re.finditer(r'<img\s', submitted_body):
    # Peek 30 chars behind — if 'class="w-embed"' precedes, this img is already wrapped
    left = submitted_body[max(0, m.start()-40):m.start()]
    if 'class="w-embed"' not in left:
        bare_img += 1
if bare_img > 0:
    discrepancies.append(f"body images outside embed: {bare_img} bare <img> — will render at natural size with side margins. Re-run Step 4.7 (transform_body_images).")

if discrepancies:
    print("WEBFLOW SANITIZATION DETECTED:")
    for d in discrepancies:
        print(f"  {d}")
    print("Fix: collapse inter-tag whitespace with re.sub(r'>\\s+<', '><', html) and re-submit.")
else:
    print("Body persisted intact ✅")
```

If counts mismatch, you have stale data in CMS. Re-emit body with continuous-string fix and call `update_collection_items` again.

## Pre-publish validation (Step 7 — fieldData assembly)

Assert these before calling `create_collection_items`. Fail fast if any fail.

```python
assert 140 <= len(fieldData["excerpt"]) <= 180, f"Excerpt {len(fieldData['excerpt'])} chars, need 140-180"
assert len(fieldData["tldr"]) >= 300, f"TLDR {len(fieldData['tldr'])} chars, need ≥300"
assert 40 <= len(fieldData["tldr"].split()) <= 80, f"TLDR {len(fieldData['tldr'].split())} words, need 40-80"
assert len(fieldData["seo-title"]) <= 65, f"SEO Title {len(fieldData['seo-title'])} chars, target 50-60"
assert len(fieldData["seo-description"]) <= 165, f"SEO Desc {len(fieldData['seo-description'])} chars, target 150-160"
takeaways = [l for l in fieldData["key-takeaways"].split("\n") if l.strip()]
assert 3 <= len(takeaways) <= 5, f"Key Takeaways {len(takeaways)} bullets, need 3-5"
assert "<br>" not in fieldData["key-takeaways"] and "\r" not in fieldData["key-takeaways"], "Key Takeaways must use \\n only"
for url_field in ["main-image", "thumbnail-image"]:
    assert fieldData[url_field].get("fileId"), f"{url_field} missing fileId"
    assert "cdn.prod.website-files.com" in fieldData[url_field].get("url", ""), f"{url_field} URL not on Webflow CDN"
assert fieldData["published-date"].endswith(".000Z"), "published-date needs .000Z suffix"
assert fieldData["published-date-iso"].endswith("Z") and ".000" not in fieldData["published-date-iso"], "ISO date should not have milliseconds"
```

## Post-publish validation (Step 10)

Run after publish completes + 20-second CDN propagation wait.

### Automated curl check

```bash
URL="https://greenm.io/post/{slug}"
curl -s -L "$URL" | python3 -c "
import sys, re
h = sys.stdin.read()
checks = {
  'title': re.search(r'<title[^>]*>([^<]*)</title>', h),
  'robots': re.search(r'<meta[^>]*name=[\"\\']robots[\"\\'][^>]*content=[\"\\']([^\"\\']*)', h),
  'desc':  re.search(r'<meta[^>]*name=[\"\\']description[\"\\'][^>]*content=[\"\\']([^\"\\']*)', h),
  'og:title':       re.search(r'property=[\"\\']og:title[\"\\'][^>]*content=[\"\\']([^\"\\']*)', h),
  'og:description': re.search(r'property=[\"\\']og:description[\"\\'][^>]*content=[\"\\']([^\"\\']*)', h),
  'og:image':       re.search(r'property=[\"\\']og:image[\"\\'][^>]*content=[\"\\']([^\"\\']*)', h),
  'twitter:card':   re.search(r'name=[\"\\']twitter:card[\"\\'][^>]*content=[\"\\']([^\"\\']*)', h),
  'canonical':      re.search(r'rel=[\"\\']canonical[\"\\'][^>]*href=[\"\\']([^\"\\']*)', h),
}
for k, m in checks.items():
    val = m.group(1) if m else None
    status = '✅' if (val and (k != 'robots' or val != 'noindex')) else '❌'
    if k == 'robots' and not val: status = '✅'
    print(f'  {status} {k:20s} {val[:80] if val else \"MISSING\"}')
ldjson = len(re.findall(r'<script[^>]*type=[\"\\']application/ld\\+json[\"\\']', h))
print(f'  {\"✅\" if ldjson >= 2 else \"❌\"} json-ld blocks    {ldjson} (expect ≥2: @graph + FAQPage)')
"
```

### Manual validation URLs

Template the URLs with URL-encoded post URL and present to user:

```python
import urllib.parse
post_url = "https://greenm.io/post/{slug}"
enc = urllib.parse.quote(post_url, safe='')
print(f"Rich Results: https://search.google.com/test/rich-results?url={enc}")
print(f"schema.org:   https://validator.schema.org/#url={enc}")
print(f"FB Debugger:  https://developers.facebook.com/tools/debug/?q={enc}")
print(f"LI Inspector: https://www.linkedin.com/post-inspector/inspect/{enc}")
# GSC: user opens https://search.google.com/search-console, picks property, URL Inspection
```

Expected results:

| Tool | Pass criteria |
|---|---|
| Google Rich Results Test | 3 valid items: Articles + Breadcrumbs + FAQ |
| schema.org validator | 0 errors, 0 warnings, 3 elements (BlogPosting, BreadcrumbList, FAQPage) |
| GSC URL Inspection | "Indexing requested" toast after clicking Request Indexing |
| FB Debugger | After clicking Scrape Again: shows correct OG title/image/description |
| LinkedIn Post Inspector | Preview card with image + title + description. Note: "No author found" warning is non-blocking (LinkedIn doesn't parse JSON-LD Person) |

## Troubleshooting tree

### Live page shows HTTP 200 but `<title>` is "GreenM"

**Cause:** Blog Posts Template Page Settings → Title Tag is not bound to `{Item SEO Title}` CMS field. Template renders a static fallback.

**Fix:** Webflow Designer → Pages → Blog Posts Template → Page Settings → SEO Settings → Title Tag → Add Field → SEO Title. Save + Publish site.

### Live page has `<meta name="robots" content="noindex">`

**Cause 1:** Per-CMS-item "Exclude this item from search engine indexing" toggle is on. (Legacy from dev period — MCP-created items default to indexable, but UI-created items may have inherited this.)

**Fix:** Webflow CMS → Blog Posts → open the item → scroll to bottom → uncheck the toggle → Save + Publish.

**Cause 2:** Site Settings → SEO → "Webflow subdomain indexing" is set incorrectly. Should be OFF for *.webflow.io but should not affect prod greenm.io.

### No OG meta tags on live page

**Cause:** Blog Posts Template Page Settings → Open Graph settings not configured.

**Fix:** Template Page Settings → Open Graph Settings:
- OG Title: "Same as SEO title tag" (check)
- OG Description: "Same as SEO meta description" (check)
- OG Image: bind to {Main Image} CMS field
Save + Publish.

### `<link rel="canonical">` missing site-wide

**Cause:** Site Settings → SEO → Global canonical tag URL is empty.

**Fix:** Site Settings → SEO → Global canonical tag URL → `https://greenm.io` (no trailing slash, no www) → Save → Publish. Site-wide, one-time.

### JSON-LD @graph is missing or only FAQPage renders

**Cause:** Blog Posts Template Custom Code → Inside `<head>` tag is missing the `<script type="application/ld+json">` block with @graph (BlogPosting + Person + BreadcrumbList).

**Fix:** Re-add the @graph script in template Custom Code (see GRO-381 + GRO-382 history). Bind fields: Slug, Title, TLDR (→ description), Main Image, Published Date ISO, Last Updated ISO, Author refs, Category Name.

### FAQ snippet is malformed

**Cause:** `faq-schema-json-2` field has extra wrapping or escaping.

**Fix:** Submit field value as raw JSON string (no `<script>` wrapper, no escaped quotes — template wraps it). Use `json.dumps(faq, separators=(',', ':'))` to produce.

### Key Takeaways render as one long paragraph, not bullets

**Cause:** Newlines are `\r\n` or `<br>` instead of `\n`.

**Fix:** Submit with `\n` only. Template JS does `raw.textContent.split('\n').map(s => s.trim()).filter(Boolean)`.

### `asset_tool` MCP returns "Unable to connect to Webflow Designer"

**Cause:** Webflow Designer is not open in an active browser tab.

**Fix:** Share the Designer link from the error message and ask user to open it foreground.
