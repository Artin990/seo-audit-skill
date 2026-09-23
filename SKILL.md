---
name: wp-elementor-seo-audit
description: Audit and fix on-page/technical SEO for a WordPress page built with Elementor (and typically Rank Math). Use whenever the user asks why a WordPress/Elementor page isn't ranking, wants an SEO audit of such a page, or wants help fixing title/meta/schema/Core-Web-Vitals/accessibility issues on it. Covers gathering raw HTML/PageSpeed/Rich-Results data (never fetch the site directly — network access to arbitrary sites is usually blocked), diagnosing title/canonical/og/schema problems, diagnosing CLS/LCP/Speed-Index causes, and giving concrete Elementor/Rank Math/WP-Rocket fix steps. Works in Persian (Farsi/RTL) or English.
---

# WordPress + Elementor SEO Audit

A step-by-step workflow for diagnosing why a WordPress page (usually built with Elementor, with Rank Math as the SEO plugin) isn't ranking, and for walking the user through fixing it — one page at a time.

This skill was distilled from a real audit-and-fix session on a live site, so it encodes the actual failure patterns that show up on WordPress/Elementor/Rank Math stacks, not just generic SEO advice.

## Ground rules

- **Don't try to fetch the site directly.** Network access from the sandbox to arbitrary domains is normally blocked (`host_not_allowed`). Always ask the user to copy/upload what's needed instead of attempting `curl`/fetch first. If a fetch is available in a given environment, try it once — if it fails, immediately fall back to asking the user.
- **Never guess at HTML you haven't seen.** A markdown/plain-text export of a page strips `<head>` tags entirely (`<link rel=canonical>`, `<meta property=og:*>`, `<script type=application/ld+json>`, `<meta name=...>` sometimes survives, but not reliably). Always ask for the **raw HTML** (View Source / Ctrl+U, saved as a `.html` file) before making any claim about canonical tags, OG tags, or schema. State this limitation explicitly if only a converted/extracted text file is available.
- **Support every claim with the evidence in front of you.** Quote the exact string, the exact number, the exact diagnostic name. Don't say "probably" when you can point at the line.
- **Re-diagnose when the user corrects your assumption about what an element is.** If the user says "that's actually a decorative floating badge, not a testimonials section," drop the previous conclusion immediately and reason from the new information — don't keep arguing for the earlier read.
- **Keep site-wide fixes separate from page-specific fixes.** Some problems (broken Organization schema, a duplicated Article/FAQ schema pattern, missing `main` landmark from the theme, contrast issues from the theme) affect every page built from the same template/plugin config. Others (a wrong title, a mismatched og:image:alt, unset image width/height on a given image) are local to one page. When the user says "just this page," only list what's actually local to it.
- **Numbers fluctuate — don't chase a single reading.** PageSpeed Insights loads the page live each time; server load, network path, and cache state all shift the result. Always recommend 3–5 runs and judging by the trend, especially for Speed Index and LCP. The "first run" (cold cache) number is the one that matters most, because it best approximates a new visitor or Googlebot.

## Workflow

### Step 1 — Discovery

Ask the user for (can be requested together or one at a time depending on how much they want to share up front):

1. **robots.txt** (plain text is fine) — check for accidental `Disallow` on important paths and a correct `Sitemap:` line.
2. **Sitemap** contents — check the target page/post is actually listed.
3. **Raw HTML of the page** (`.html` file, full View Source) — this is the one piece of data almost everything else depends on.
4. **Rich Results Test result** (`search.google.com/test/rich-results`) — exact count and exact text of every warning/error.
5. **PageSpeed Insights, Mobile** (mobile-first indexing means desktop is optional unless the user wants a comparison):
   - the four category scores (Performance / Accessibility / Best Practices / SEO)
   - LCP, FCP, SI, CLS, TBT
   - the red/yellow **Insights** items with their "Est savings"
   - the **Diagnostics** items
   - the **Accessibility** audit failures

### Step 2 — Parse the raw HTML

From the `<head>` and body, check:

- **`<title>`** — unique, not duplicated (a real recurring bug: Rank Math templates sometimes render `{title} - {title}` when a fallback and a set value both resolve to the same string), reasonable length.
- **`<meta name="description">`** — unique, ~150–160 characters.
- **`<link rel="canonical">`** — must self-reference the correct URL.
- **`og:*` / `twitter:*`** — should not just repeat a broken title; `og:image:alt` in particular is a common place to find copy-pasted leftovers from a *different* client's page — check the alt text actually describes the image that's actually set as og:image, and actually belongs to this site/client.
- **JSON-LD (`<script type="application/ld+json">`)** — parse every block:
  - If more than one schema type covers similar ground (e.g. both `Article` and `FAQPage`), check whether one is stale/duplicated/empty. `Article` schema on a non-blog page (a service/landing page) is usually a leftover default and not meaningful — removing it is often correct, especially when it also contains a broken nested duplicate (see next point).
  - Watch for a schema block nested inside another (e.g. an empty `FAQPage` sitting inside an `Article`'s `subjectOf`) with **malformed dates** — Rank Math on some configurations emits the Persian/Jalali calendar string with a literal backslash (`"1404-04-23\\12:31:32"`) instead of ISO 8601 (`"2025-07-14T16:01:32+03:30"`). This is a real, recurring bug pattern, not a one-off — flag it as systemic (fix once in Rank Math's schema settings/generator, not per page) rather than as a single-page issue.
  - Watch for placeholder values that were never replaced (`"legalName": "webwebg"` style junk) in `Organization`/`Person` schema — this is site-wide (usually set once in the SEO plugin's general settings), not a page bug.
  - `Product` schema type is for physical/purchasable goods (expects things like GTIN, review aggregates). A page selling a **service** (web design, consulting, photography packages, etc.) should generally use `Service` or `Offer`-under-`Service`, not `Product`.
  - Schema should describe the content that's actually on the page. If a page has 7 FAQ questions visible, the FAQPage schema should have all 7 — a mismatch between visible content and schema can cause rich results to be dropped or flagged.
- **Heading structure** — exactly one `H1`; check the sequence for skipped levels (H1 → H3 with no H2 in between). Before touching any heading, figure out **what the element actually is**:
  - If it's real content structure (a testimonials section header, a service list header), fix the *level*, don't strip the tag.
  - If it's a decorative/UI element (a floating stat badge, an icon label, a UI chip) that got a heading tag by accident, the fix is to change its tag to `span`/`div`, not to renumber it — it was never supposed to be a heading. Don't broadly convert *all* similar-looking cards to a different level just because one triggered an audit warning; find the one real gap (usually a missing section-level H2) and fix that specific thing.
- **Image alt text** — compute the percentage of `<img>` tags with an empty or missing `alt`. A "Name of project — url-without-https" pattern on portfolio/work-sample images is a legitimate, well-established convention — don't flag it as wrong. What actually matters: does the alt text describe *this specific image*, and does it belong to *this* client/page (watch for copy-paste mismatches, e.g. one portfolio image carrying a competitor's project name).
- **Image filenames** — camera-default names (`IMG_20251106_131501_912.jpg`) vs. descriptive ones.
- **`loading="lazy"`** should not be present on the page's LCP element (hero image/background).

### Step 3 — Diagnose PageSpeed results

Priority order, worst offender first:

1. **CLS high?** → Almost always missing explicit `width`/`height` **HTML attributes** on an `<img>` (not CSS — CSS controls displayed size, the HTML attribute is what lets the browser reserve layout space before the image loads). Section background images and hero images are the first suspects. If the user has DevTools access, point them at Performance tab → Layout Shift culprits for the exact element, don't guess.
2. **LCP high?** → Usually the hero image or first large content block. Check it isn't lazy-loaded, is compressed, and is served in a modern format (WebP/AVIF).
3. **Render-blocking requests?** → Caching plugin (WP Rocket or equivalent): enable "Delay JavaScript execution" and "Optimize CSS Delivery" / critical-CSS generation.
4. **Speed Index high?** → Usually a downstream symptom of render-blocking + heavy main-thread JS work, not a separate root cause. Re-test after fixing #3 before treating it as its own problem.
5. **Image weight / "Improve image delivery" still flagged after compressing to WebP?** → The likely remaining cause is **resolution, not format** — the file is served at its original (large) dimensions even though it's displayed much smaller. Check actual file dimensions (DevTools → Network → the image request) against rendered size (DevTools → Elements). Fix: enable responsive image generation (`srcset`) in Elementor/the image-optimization plugin, or resize the source file to the real display width before upload.

### Step 4 — Accessibility

Common findings and their usual fix on this stack:

| Finding | Likely fix |
|---|---|
| "Document does not have a main landmark" | Check the page's Elementor **Page Layout** setting — "Canvas" strips the theme's structural wrapper (including `<main>`); switch to "Default"/"Full Width". Fast workaround without touching the theme: add a Custom Attribute `role\|main` to the page's outer container. |
| "Heading elements are not in a sequentially-descending order" | See Step 2's heading guidance — find the *specific* skipped level, don't blanket-renumber. |
| Insufficient color contrast | Check with a contrast tool, target WCAG AA (4.5:1 for normal text). |
| "Links do not have a discernible name" | Icon-only links (social icons, etc.) need an `aria-label`. |
| Skip link not focusable | Usually a theme-level fix, not a per-page one. |

These four issues repeat almost identically across pages built from the same theme/template — treat a fix here as a template-level fix, not a one-page task, unless the user explicitly wants to scope to a single page.

### Step 5 — Summarize and hand off

After each round of fixes, give the user:
- A before/after table for the scores and key metrics.
- A clear list of what's done vs. what's still open, with the *reason* it's still open if it's blocked on something (e.g. "need the raw HTML of page X to check its schema").
- If asked for a step-by-step action list, order it: critical/quick wins first (title, schema, alt text) → performance (render-blocking, images, fonts) → accessibility (landmark, heading order, contrast, link names).

## Common mistakes to avoid

- Don't conclude a tag is missing (canonical, og:*, schema) from a markdown/plain-text export — that's a conversion artifact, not evidence.
- Don't flag a portfolio image's "Project Name — url" alt-text pattern as an SEO problem; it's a convention, not an error.
- Don't tell the user to make *all* heading elements of a similar type match the one that triggered an audit warning — find and fix the specific gap.
- Don't recommend percentage-based `width`/`height` for the CLS fix — the browser needs the image's real pixel dimensions to reserve space; percentages have nothing to resolve against before layout.
- Don't treat a single PageSpeed run as ground truth, especially for Speed Index — always suggest 3–5 runs.
- Don't force keyword repetition counts (e.g. "use the keyword exactly 3 times") — modern ranking systems read for topical intent, not keyword density, and forced repetition reads as stuffing.
- Don't reuse the exact same `alt` text across many different images "for SEO" — that's also keyword stuffing and provides zero descriptive value per image.
- Don't use `Product` schema for a service.

## Reference

See `references/checklist.md` for a copy-pasteable, plain checklist version of this workflow (useful to hand to a fresh chat along with a URL, without the full explanatory text above).
