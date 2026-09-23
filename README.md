# wp-elementor-seo-audit

یک **Claude Skill** برای بررسی و اصلاح سئوی صفحات وردپرسی/المنتور (معمولا با Rank Math)، بر پایه یه جلسه واقعی audit-و-اصلاح روی یه سایت زنده.

## این چیه؟

یه [Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) برای Claude — وقتی نصب بشه، Claude خودکار موقع سوال درباره‌ی «چرا سایت وردپرسی/المنتوریم رتبه نمی‌گیره» این مراحل رو دنبال می‌کنه:

1. جمع‌آوری داده (robots.txt، sitemap، HTML خام، Rich Results Test، PageSpeed Insights)
2. تحلیل تگ‌های on-page (title، meta، canonical، og، schema)
3. تشخیص علت مشکلات سرعت (CLS، LCP، Speed Index)
4. تشخیص مشکلات Accessibility
5. جمع‌بندی و اولویت‌بندی اقدامات

مهم‌ترین ارزشش الگوهای خطای *تکراری* هستن که تو استک وردپرس/المنتور/RankMath واقعا دیده شدن (نه صرفا توصیه‌های عمومی سئو) — مثل:

- باگ تاریخ شمسی به‌جای ISO 8601 تو schema خودکار Rank Math
- Article schema تکراری/بی‌ربط که یه FAQPage خالی رو تو خودش قایم می‌کنه
- CLS ناشی از نبود width/height پیکسلی رو تگ `<img>`
- تشخیص اشتباه heading روی المان‌های تزئینی (badge/UI) به‌جای محتوای واقعی

## نصب

### Claude.ai / Claude Desktop / Claude Code

این پوشه رو کپی کن به مسیر skills خودت:

```bash
git clone https://github.com/<your-username>/wp-elementor-seo-audit.git
cp -r wp-elementor-seo-audit ~/.claude/skills/wp-elementor-seo-audit
```

(مسیر دقیق بسته به کلاینت فرق می‌کنه — مستندات رسمی Agent Skills رو چک کن.)

### استفاده بدون نصب Skill (ساده‌ترین راه)

فایل [`references/checklist.md`](./references/checklist.md) یه نسخه خلاصه و مستقله. کافیه محتواش رو کپی کنی و به هر چت جدید (Claude یا هر مدل دیگه) بدی، بعد URL صفحه مورد نظرت رو بگی — نیازی به نصب Skill نیست.

## ساختار پروژه

```
wp-elementor-seo-audit/
├── SKILL.md              # فایل اصلی اسکیل (instructions کامل)
├── README.md              # همین فایل
└── references/
    └── checklist.md        # نسخه خلاصه/چک‌لیستی، قابل کپی‌پیست مستقیم
```

## نحوه استفاده (مثال جلسه)

```
کاربر: این صفحه رو بررسی کن، چرا رتبه نمی‌گیره؟
       https://example.com/some-page/

Claude: [درخواست robots.txt، sitemap، HTML خام، Rich Results، PageSpeed]
        [تحلیل title/meta/canonical/og/schema]
        [تشخیص CLS/LCP/Speed Index از روی داده‌های PageSpeed]
        [تشخیص Accessibility]
        [جدول جمع‌بندی + لیست اقدامات اولویت‌بندی‌شده]
```

## قوانین کلیدی این Skill

- **هیچوقت مستقیم fetch نمی‌کنه سایت رو** — چون تو خیلی از محیط‌ها دسترسی شبکه به دامنه‌های دلخواه بسته‌ست. همیشه از کاربر می‌خواد داده‌ها رو دستی بده.
- **از روی متن استخراج‌شده (نه HTML خام) درباره canonical/og/schema نتیجه نمی‌گیره** — چون این تگ‌ها تو تبدیل HTML→متن حذف میشن.
- **هر ادعا رو با مدرک دقیق پشتیبانی می‌کنه** — عدد، متن خطا، خط کد.
- **وقتی کاربر یه فرض رو اصلاح می‌کنه، فورا تحلیل رو عوض می‌کنه**، نه اینکه رو حدس اول لجبازی کنه.
- **اصلاحات سایت‌وایه رو از تک‌صفحه‌ای جدا نگه می‌داره.**

## مشارکت

اگه یه الگوی خطای تکراری دیگه‌ای تو استک وردپرس/المنتور/RankMath پیدا کردی که این‌جا نیست، PR بده یا Issue باز کن.

## لایسنس

MIT



# wp-elementor-seo-audit

A **Claude Skill** for auditing and fixing SEO on WordPress/Elementor pages (typically using Rank Math), based on a real audit-and-fix session on a live site.

## What is this?

A [Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) for Claude — once installed, Claude automatically follows this workflow whenever you ask "why isn't my WordPress/Elementor page ranking":

1. Gather data (robots.txt, sitemap, raw HTML, Rich Results Test, PageSpeed Insights)
2. Analyze on-page tags (title, meta, canonical, og, schema)
3. Diagnose the root cause of speed issues (CLS, LCP, Speed Index)
4. Diagnose accessibility issues
5. Summarize and prioritize the fix list

Its main value is the *recurring* failure patterns actually observed on a real WordPress/Elementor/Rank Math stack — not just generic SEO advice. For example:

- A Persian/Jalali-date-instead-of-ISO-8601 bug in Rank Math's auto-generated schema
- A duplicated/irrelevant `Article` schema hiding an empty `FAQPage` node inside it
- CLS caused by missing pixel-based `width`/`height` HTML attributes on `<img>` tags
- Misdiagnosing decorative/UI elements (floating stat badges) as content headings

## Installation

### Claude.ai / Claude Desktop / Claude Code

Copy this folder into your skills directory:

```bash
git clone https://github.com/<your-username>/wp-elementor-seo-audit.git
cp -r wp-elementor-seo-audit ~/.claude/skills/wp-elementor-seo-audit
```

(The exact path depends on your client — check the official Agent Skills docs.)

### Using it without installing a Skill (easiest option)

[`references/checklist.md`](./references/checklist.md) is a standalone, condensed version (in Persian). You can just copy its contents into any new chat (Claude or any other model), then give it the URL of the page you want audited — no Skill installation required.

## Project structure

```
wp-elementor-seo-audit/
├── SKILL.md              # The main skill file (full instructions)
├── README.md              # This file
└── references/
    └── checklist.md        # Condensed checklist version (Persian), copy-paste ready
```

## Example session

```
User: Audit this page, why isn't it ranking?
      https://example.com/some-page/

Claude: [asks for robots.txt, sitemap, raw HTML, Rich Results, PageSpeed]
        [analyzes title/meta/canonical/og/schema]
        [diagnoses CLS/LCP/Speed Index from the PageSpeed data]
        [diagnoses accessibility issues]
        [summary table + prioritized fix list]
```

## Key rules this skill follows

- **Never fetches the site directly** — sandboxed environments usually block network access to arbitrary domains. It always asks the user to supply the data manually.
- **Never draws conclusions about canonical/og/schema from a converted/extracted text file** — those tags get stripped in HTML-to-text conversion; it insists on the raw HTML.
- **Backs every claim with exact evidence** — the exact number, the exact error text, the exact line.
- **Immediately re-diagnoses when the user corrects an assumption**, instead of arguing for its first read.
- **Keeps site-wide fixes separate from page-specific fixes.**

## Contributing

If you've found another recurring failure pattern in the WordPress/Elementor/Rank Math stack that isn't covered here, open a PR or an Issue.

## License

MIT
