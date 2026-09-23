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
