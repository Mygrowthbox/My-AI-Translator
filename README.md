# My-AI-Translator - wordpress plugin
> Automatic AI-powered translation for WordPress + Polylang, with bulk translation, on-demand translation, and SEO support via RankMath.

![Version](https://img.shields.io/badge/version-3.0.0-blue)
![WordPress](https://img.shields.io/badge/WordPress-7.0%2B-21759b)
![PHP](https://img.shields.io/badge/PHP-8.2%2B-777bb4)
![License](https://img.shields.io/badge/license-MIT-green)
![Polylang](https://img.shields.io/badge/Polylang-required-orange)

---

## 📖 Overview

**My AI Translator** is a WordPress plugin that automates content translation using AI providers (OpenAI, Google Gemini, OpenRouter, Groq). It integrates natively with **Polylang** to manage multilingual content, and with **RankMath** to translate SEO metadata alongside each post.

Built for media sites and content-heavy WordPress installations, the plugin supports bulk translation via a background job queue, on-demand single-post translation, and a dashboard that tracks translation progress per language in real time.

---

## ✨ Features

### Translation
- **Bulk translation** — translate all posts/pages in one click via a background job queue (WP-Cron)
- **On-demand translation** — translate any individual post immediately from the admin
- **Multi-language in one click** — detect untranslated articles and translate to EN + ES simultaneously from the dashboard
- **Scope control** — translate all content, untranslated only, or a manual selection
- **Auto-failover** — if the primary AI provider fails, the plugin automatically retries with fallback providers

### AI Providers
| Provider | Free tier | Models |
|---|---|---|
| Google Gemini ⭐ | ✅ 1 500 req/day | gemini-2.5-flash, gemini-2.0-flash |
| OpenRouter | ✅ Free models | DeepSeek, Llama 3.3, Gemma, Qwen |
| OpenAI | ❌ Paid | gpt-4o-mini, gpt-4o |
| Groq | ✅ 100k tokens/day | Llama 3.3, Llama 3.1 |

### Dashboard
- Per-language translation progress (published, drafts, remaining)
- **"Latest untranslated articles"** section — shows recent FR posts missing EN/ES translations with one-click CTAs
- Real-time job queue monitoring
- Uniform CTAs across all target languages

### SEO
- Translates **RankMath** meta fields: `rank_math_title`, `rank_math_description`, `rank_math_og_title`, `rank_math_og_description`
- Respects character limits (60 chars for meta title, 160 for meta description)

### Polylang Integration
- Reads and writes translation links via `pll_get_post_translations()` / `pll_save_post_translations()` — the native Polylang API
- **Repair tool** — scans all MAT-translated posts and recreates missing Polylang links, fixing incorrect stats
- Language auto-detection and assignment for translated posts

---

## 🛠 Requirements

- WordPress 6.0+
- PHP 8.2+
- [Polylang](https://wordpress.org/plugins/polylang/) (free or Pro)
- At least one AI provider API key
- WP-Cron enabled (for bulk translation queue)

---

## 🚀 Installation

1. Download the latest release ZIP
2. In WordPress admin → **Plugins → Add New → Upload Plugin**
3. Upload the ZIP and activate
4. Go to **My AI Translator → Settings** and configure:
   - Source language (e.g. French)
   - AI provider + API key
   - Target languages (auto-detected from Polylang)

---

## ⚙️ Configuration

### Getting API Keys

| Provider | Free | Link |
|---|---|---|
| Google Gemini | ✅ | [aistudio.google.com](https://aistudio.google.com/app/apikey) |
| OpenRouter | ✅ | [openrouter.ai/keys](https://openrouter.ai/keys) |
| OpenAI | ❌ | [platform.openai.com](https://platform.openai.com/api-keys) |
| Groq | ✅ | [console.groq.com](https://console.groq.com/) |

### Recommended Setup (media/news site)
```
Source language: French (fr)
Primary provider: Google Gemini (gemini-2.5-flash)
Fallback: OpenRouter (deepseek/deepseek-v4-flash:free)
Target languages: English (en), Spanish (es)
Publish mode: Draft (review before publishing)
```

---

## 📐 Architecture

```
my-ai-translator/
├── my-ai-translator.php          # Plugin bootstrap, AJAX action registration
├── includes/
│   ├── class-api-handler.php     # AI provider calls, failover logic, chunking
│   ├── class-batch-processor.php # Job queue, bulk translation AJAX handlers
│   ├── class-translator-core.php # Single translation, stats, untranslated detection
│   ├── class-polylang-bridge.php # Polylang integration, link repair
│   ├── class-rankmath-handler.php# RankMath SEO fields translation
│   └── class-language-manager.php# Language utilities
├── admin/
│   ├── class-admin-ui.php        # Menu registration, asset enqueueing
│   └── views/
│       ├── dashboard.php         # Main dashboard with stats + recent untranslated
│       ├── translate.php         # Bulk + on-demand translation UI
│       ├── remaining.php         # List of untranslated posts per language
│       ├── settings.php          # Plugin settings
│       └── debug-log.php         # Logs, system info, Polylang repair tool
└── assets/
    ├── admin.js                  # All admin JS (batch progress, single translate,
    │                             #   multi-lang auto-translate from dashboard)
    └── admin.css                 # Admin styles
```

---

## 🔧 Key Technical Decisions

**Single source of truth for translation status**
All translation checks use `pll_get_post_translations()` — the native Polylang API — rather than custom post meta. This ensures articles translated manually (outside MAT) are correctly detected as translated.

**Failover chain**
The `translate_with_failover()` method tries each provider in order. Gemini retries up to 5× with exponential backoff on overload. OpenRouter cycles through all free models automatically. This makes bulk translation resilient without manual intervention.

**Unified counting**
`get_global_stats()`, `remaining.php`, `get_untranslated_recent_posts()`, and the `untranslated` batch scope all use the same logic: load source post IDs via SQL filtered by Polylang language taxonomy, then check each via `pll_get_post_translations()`. Zero inconsistencies between the dashboard count and the actual list.

---

## 🐛 Debugging

The **Debug & Logs** page (admin menu) shows:
- Timestamped log of all translation jobs and API calls
- System info (PHP, WordPress, RankMath, WP-Cron status)
- **Polylang link repair tool** — detects and fixes missing `pll_translations` links that cause incorrect stats

---

## 📄 License

MIT — free to use, modify and distribute.

---

## 🙋 Author

Built as a custom internal tool for a French media site requiring automated FR → EN + ES translation at scale.
