# ⚡ Mercury — Domain Guardian

<div align="center">

![HTML](https://img.shields.io/badge/HTML-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![PHP](https://img.shields.io/badge/PHP-777BB4?style=for-the-badge&logo=php&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Version](https://img.shields.io/badge/version-5.0.0-brightgreen?style=for-the-badge)
![Self-hosted](https://img.shields.io/badge/self--hosted-no_server_needed-blue?style=for-the-badge)

**Open-source uptime, DNS, SSL and latency monitor. One HTML file. Zero dependencies.**

*Mercury, the Winged Messenger God — watching over your domains, flawlessly, serverlessly.*

</div>

---

## 🌐 Live Demo

**[demo.mercury.sh](https://demo.mercury.sh)** — top 100 world domains monitored in real time.

---

## 👁 What is Mercury?

Mercury is a **self-hosted infrastructure dashboard** that monitors uptime, DNS health, SSL certificates, and mail security (DMARC/SPF) for any list of domains — entirely in the browser, with no backend required for basic use.

- 🔍 **Live DNS checks** via Cloudflare DoH (HTTPS, no CORS issues)
- 🔐 **PIN-protected** dashboard (SHA-256 hashed, 3-tier persistence)
- 🌓 **Light / Dark mode** (light by default)
- ⚡ **Progressive scan** — rows light up one batch at a time
- 📱 **Mobile-first** — native numeric PIN keyboard on touch devices
- 🔄 **Per-row refresh** — re-scan any single domain with ↺
- ⏱️ **Auto-refresh** every 3 minutes with live countdown
- 🔔 **Email alerts** via Resend API — downtime, SSL expiry, DMARC/SPF issues
- 📊 **Cross-device uptime** — server-side `uptime.json` shared across all browsers
- 📋 **Export CSV** — download a timestamped snapshot any time
- ➕ **Add domains live** — type any domain, checked immediately
- 📁 **`domains.list`** — edit a plain text file to manage your watchlist

---

## 🎬 What it monitors

| Record | What it reveals |
|---|---|
| `A` | Is the domain resolving? Round-trip latency? |
| `NS` | Nameserver provider (Cloudflare, AWS, SiteGround…) |
| `MX` | Mail provider (Google, ProtonMail, Microsoft…) |
| `TXT` | SPF record (`v=spf1 … ~all`) |
| `_dmarc TXT` | DMARC policy (`reject` / `quarantine` / `none` / `missing`) |

---

## 🛠️ What's in the box

| File | Purpose |
|---|---|
| `index.html` | Full application — HTML shell linking `app.css` and `app.js` |
| `app.css` | All styles |
| `app.js` | All JavaScript — DNS checks, PIN, notifications, uptime |
| `domains.list` | Watchlist — one domain per line, `#` for comments |
| `domains.stats` | CSV snapshot updated after every check |
| `domains.json` | Written by `update-stats.php` — feeds SSL expiry on load |
| `ssl-check.php` | Server-side SSL checker with batch mode `?domains=` |
| `config-write.php` | PHP config endpoint — PIN, theme, notification settings |
| `ase_config.json` | Auto-created settings store (PIN hash, notifications) |
| `uptime-write.php` | Server-side uptime accumulation endpoint |
| `notify.php` | Resend email sender — AES-256-GCM encrypted key storage |
| `update-stats.php` | Server-side cron — DNS + SSL checks, writes stats |
| `webhook.do` | Headless endpoint for external cron services |
| `.htaccess` | Apache config — no-cache headers, protected files |
| `INSTALL.md` | Full setup guide |

---

## 📦 Quick Start

```bash
# 1. Clone or download
git clone https://github.com/paulfxyz/mercury-sh.git
cd mercury-sh

# 2. Upload to your web server
# scp -r . user@yourhost:/public_html/

# 3. Visit https://yourdomain.com/
# Default PIN: 123456 — change it immediately
```

No npm, no Composer, no build step. Three files do everything: `index.html`, `app.css`, `app.js`.

---

## 🔑 Default PIN

Default PIN: **`123456`**

On first login, you're automatically prompted to set a personal PIN. Change via **More ⋮ → Change PIN** at any time.

### Three-tier persistence

| Tier | Storage | Scope |
|---|---|---|
| 1 | `ase_config.json` (server) | All browsers + devices |
| 2 | `ase_pin` cookie | Current browser |
| 3 | Hardcoded in `index.html` | Deployment default |

---

## 🔔 Email Notifications (Resend)

Configure via **More ⋮ → Notifications**. Uses [Resend](https://resend.com) (free tier: 100 emails/day).

Every alert includes a full domain health digest:
- ✅ Uptime status + latency
- 🔐 SSL expiry (days remaining, colour-coded)
- 📧 DMARC policy + SPF record
- 🌐 Nameserver + mail provider

**Smart cooldown system:**
- Manual Refresh → 5-minute cooldown (immediate confirmation)
- Auto-refresh → 24-hour cooldown (no inbox spam)
- API key encrypted AES-256-GCM server-side — never plaintext

---

## 🧠 How it works under the hood

### DNS-over-HTTPS (DoH)
Browsers block raw DNS sockets. Mercury uses [Cloudflare's DoH API](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-https/) over HTTPS — works everywhere, no CORS issues. Five parallel queries per domain: `A`, `NS`, `MX`, `TXT`, `_dmarc.TXT`.

### Progressive batch scanning
Checks run in batches of 5 with 300ms pauses — avoids looking like a DoH flood. Table re-renders after each batch. Total time for 100 domains: ~6–8 seconds.

### SSL certificate checking — three-tier strategy
1. `ssl-check.php?domains=...` — batch PHP, one request for all domains
2. `crt.sh` per-domain — certificate transparency logs (fallback for static hosts)
3. `domains.json` from cron — seeded on page load

### Uptime persistence
`uptime.json` via `uptime-write.php` — server-side accumulation shared across all devices. Cookie fallback if PHP unavailable.

### SHA-256 caching bug (and fix)
The original SHA-256 implementation cached prime tables on the function object. Corrupted on subsequent calls. Fix: **stateless implementation** that recomputes primes fresh every call.

### Modal architecture — `overflow:hidden` + `position:sticky` conflict
`position:sticky` silently fails when any ancestor has `overflow:hidden`. Fix: flex-column modal architecture where header and footer are `flex-shrink:0` (always visible). No `overflow:hidden` anywhere.

### Header dropdown — CSS stacking context escape
The sticky header creates a stacking context. Dropdown menus inside it are capped at the header's z-index in the root context. Fix: `position:fixed` + `getBoundingClientRect()` — menu escapes the stacking context entirely.

### Email notifications — dual cooldown system
Two cooldown tables: `NOTIFY_COOLDOWN_MANUAL` (5 min) for manual Refresh, `NOTIFY_COOLDOWN_AUTO` (24h) for auto-refresh. `_manualRefresh` flag set by `triggerRefresh()`, passed through `checkAll()` to `sendHealthReport()`. Cooldowns persisted to `ase_config.json` via `_notifySaveState()`.

### Config persistence
`config-write.php` + `ase_config.json` — atomic writes (temp + rename + LOCK_EX). Three-tier PIN persistence survives incognito and cross-device.

---

## 📝 Changelog

> Full changelog: **[CHANGELOG.md](./CHANGELOG.md)**

### 🔖 v5.0.0 — 2026-03-25
- 🌍 **rebrand:** The All Seeing Eye → Mercury. Repository renamed to `mercury-sh`
- 🌍 **feat:** BUILTIN top-100 world domains (from top-50)
- 📱 **fix:** Mobile PIN — no duplicate dots, auto-focus fixed, Change PIN modal keyboard
- 🔔 **fix:** Manual Refresh triggers email (dual cooldown: 5min manual / 24h auto)
- 💾 **feat:** Notification state persists via `ase_config.json` across page reloads
- 🚀 **feat:** `demo.mercury.sh` — live public demo with top 100 domains

### 🔖 v4.1.0 — 2026-03-25
- 📱 Mobile PIN UX overhaul — no duplicate dots, auto-focus, centred input

### 🔖 v4.0.0 — 2026-03-23
- 🔔 Smart notification cooldowns + server-side persistence

### 🔖 v3.3.x — 2026-03-23
- 🔔 Full notification coverage: cron + browser, digest email, deduplication

### 🔖 v3.0.0 — 2026-03-22
- 📱 Mobile-first overhaul: native PIN keyboard, rebuilt modal system

### 🔖 v2.1.0 — 2026-03-22
- 🔐 PIN persistence via `config-write.php` + `ase_config.json`

---

## ⬇️ Download

👉 **[Download latest ZIP](https://github.com/paulfxyz/mercury-sh/archive/refs/heads/main.zip)**

---

## 🤝 Contributing

Pull requests welcome! Ideas: ping history graphs, multi-user support, Slack/Telegram alerts, dark-mode dashboard improvements.

1. Fork the repo
2. Create branch: `git checkout -b feature/my-improvement`
3. Commit: `git commit -m 'Add feature'`
4. Push: `git push origin feature/my-improvement`
5. Open a Pull Request

---

## 📜 License

MIT License — free to use, modify, and distribute.

---

## 👤 Author

Built with ❤️ — designed and built in collaboration with **[Perplexity Computer](https://www.perplexity.ai/computer)**.

- 🌐 **[mercury.sh](https://mercury.sh)**
- 🐙 **[github.com/paulfxyz/mercury-sh](https://github.com/paulfxyz/mercury-sh)**
- 🔴 **[demo.mercury.sh](https://demo.mercury.sh)**

---

⭐ **If this saved you time, drop a star — it helps others find it!** ⭐
