# 🗺️ 1984 VPN — Product Roadmap

All planned features, ideas, and future directions discussed during development.

---

## ✅ Phase 0: Infrastructure (DONE)

- [x] Domain `1984vpn.com` registered on Cloudflare
- [x] Master server: Hetzner Helsinki CX22 — Marzban + Caddy
- [x] Edge node: Hetzner Helsinki CX22 — XRay (dirty transit, no logs)
- [x] VLESS+Reality protocol on port 2053
- [x] `panel.1984vpn.com` accessible via HTTPS
- [x] Admin Telegram bot `@VPN1984PrivacyBot` connected to Marzban
- [x] SSH access via ed25519 key in Termius

---

## 🚧 Phase 1: Customer Bot MVP (Next)

Core sales bot for the first paying users.

- [ ] Create customer bot via @BotFather
- [ ] Deploy on Railway (aiogram 3, Python)
- [ ] Connect to Marzban REST API
- [ ] `/start` — welcome message + onboarding
- [ ] Free trial: subscribe to Telegram channel → get N free days
  - Users subscribe to channel, bot verifies via `getChatMember`
  - 1 channel subscription = 1 free day (configurable)
- [ ] Purchase flow: select plan → pay → receive key
- [ ] Subscription plans: 1 month / 3 months / 6 months / 12 months
- [ ] Auto-create Marzban user on payment
- [ ] Send subscription link + QR code to user
- [ ] `/myaccount` — show subscription status, expiry, traffic used
- [ ] Renewal reminders (3 days before expiry)
- [ ] Client app instructions (V2Box iOS, V2RayNG Android)

---

## 💳 Phase 2: Payments

- [ ] **Telegram Stars** — native Telegram payment, no KYC needed
  - Simplest to implement, works for Russian users
  - ~1.4–1.8 ₽ per Star
- [ ] **Tegro.Money** — crypto payments (TON, USDT, USDC, BTC)
  - [API Docs](https://tegro.money/docs/en/)
  - No commission for 1984 VPN ecosystem
  - Ideal for users without Russian bank cards
- [ ] **Test payment 50₽** — verified payment for trial week
  - Filters out bots and RKN scanners (learned from Magnum VPN competitor)
  - Converts better than free trial (users who paid once tend to stay)
- [ ] Auto-renewal with notification
- [ ] Payment history in bot
- [ ] Invoice generation

---

## 🔗 Phase 3: Referral Program

2-level referral system.

- [ ] Level 1: **30% from direct referral** payments (lifetime)
- [ ] Level 2: **10% from referral's referrals** payments (lifetime)
- [ ] Referral link: `t.me/YourBot?start=ref_USERNAME`
- [ ] `/referrals` command — stats, earnings, withdraw
- [ ] Payout to Telegram Stars or TON wallet
- [ ] Referral leaderboard (top inviters get bonuses)

---

## 🎟️ Phase 4: Promo Codes & Marketplace Distribution

Sell VPN access through third-party channels.

- [ ] Promo code system in bot: `/activate CODE`
- [ ] Admin panel for generating codes (single-use / multi-use)
- [ ] **[Libermall.com](https://libermall.com/)** distribution
  - List VPN activation codes on marketplace
  - Buyers get code → activate in bot
  - Passive distribution channel
- [ ] Partner promo codes (for bloggers, affiliates)
- [ ] Codes for specific plans (1 month, 3 months, etc.)
- [ ] Bulk code generation for B2B (corporate clients)

---

## 🏷️ Phase 5: White-Label / Partner Program

Allow partners to sell VPN under their own brand.

- [ ] Partner account in admin panel
- [ ] Partner adds their **own Telegram Bot token** in settings
- [ ] All users who come through partner's bot automatically become their referrals
- [ ] Partner gets revenue share (configurable %)
- [ ] Partner can set custom welcome message, bot name
- [ ] This creates a **pseudo-white-label** without separate infrastructure
- [ ] Multi-brand from one Marzban installation:
  - `@VPN1984PrivacyBot` — main brand
  - `@TegroVPNbot` — Tegro ecosystem brand
  - `@Partner1Bot` — partner brand
  - All use the same Hetzner servers and Marzban panel

---

## 🌍 Phase 6: Infrastructure Scaling

- [ ] Add edge nodes in other locations:
  - 🇩🇪 Nuremberg / Falkenstein (Germany)
  - 🇳🇱 Amsterdam (Netherlands)
  - 🇸🇬 Singapore (for Asia users)
- [ ] Node health monitoring
- [ ] Auto-failover between nodes
- [ ] Traffic balancing across nodes
- [ ] Separate node for YouTube (optimized routing)
- [ ] **Cloudflare 1.1.1.1 DNS** integrated into VPN client config
  - Prescript in XRay: route DNS through 1.1.1.1
  - Users get both VPN + fast DNS automatically

---

## 📱 Phase 7: Client Applications

- [ ] **Landing page** — `1984vpn.com`
  - Simple one-pager
  - Links to bot, client apps, instructions
  - Connection guide for iOS/Android/Windows/macOS
- [ ] **Browser extension** (Chrome/Firefox)
  - Connects to user's subscription link
  - One-click enable/disable
  - Shows connection status
- [ ] **Mobile app** (iOS + Android)
  - Native VPN client with 1984 VPN branding
  - Based on XRay-core SDK
  - In-app subscription management
  - No separate client app needed (currently users need V2Box/V2RayNG)

---

## 🧩 Phase 8: Ecosystem Integration

Integration with existing product ecosystem.

- [ ] **Tegro VPN** — fork 1984 VPN infrastructure under Tegro brand
  - `tegrovpn.com` or subdomain of `tegro.ru`
  - Target: Tegro Finance / Tegro Money existing users
  - Same servers, different Telegram bot + branding
- [ ] **TonChat.AI widget** on `1984vpn.com` landing
  - AI assistant helps users set up VPN
  - Powered by [TonChat.AI](https://tonchat.ai)
- [ ] **$TGR token** holders get VPN discount
  - Connect TON wallet in bot
  - Verify $TGR balance → apply discount
- [ ] **SMOService** cross-promo
  - SMM panel users get VPN offer
  - VPN users get SMM discount

---

## 🔒 Phase 9: Security & Privacy Enhancements

- [ ] Change admin panel URL from `/dashboard` to random path (security through obscurity)
- [ ] Rename admin login from `your_admin_username` to non-obvious username
- [ ] Rate limiting on Marzban API
- [ ] IP-based abuse detection
- [ ] Rotate edge node IPs periodically
- [ ] Separate technical domain for VPN traffic (not linked to brand)
- [ ] Abuse-resistant registrar for technical domain (Njalla or Porkbun)
- [ ] Log rotation policy
- [ ] Automated backups of Marzban DB to remote storage

---

## 📊 Revenue Model Summary

| Channel | Mechanic | Revenue |
|---------|----------|---------|
| Direct bot sales | Subscription | MRR |
| Referral program | 2-level commission | Viral growth |
| Marketplace codes | Libermall.com | Passive |
| White-label partners | Revenue share | B2B |
| Corporate B2B | Bulk codes | One-time + recurring |
| Tegro VPN brand | Separate funnel | Ecosystem |

### Pricing Benchmark (from market research)

| Competitor | Price/month | Users |
|-----------|------------|-------|
| Magnum VPN | 299 ₽ | 138k |
| Need VPN | ~310 ₽ ($3.20) | 108k |
| VlessWB | 155 ₽ | — |
| ADD VPN | от 120 ₽ | — |
| **1984 VPN** | **TBD** | 0 → ∞ |

---

## 📝 Technical Debt & Known Issues

- [ ] Xray core version WARNING (24.12.31 vs 25.3.6) — update core
- [ ] `IMPORTANT: running without SSL certs` warning in logs — cosmetic, Caddy handles SSL externally
- [ ] UFW force-enabled but SSH rule added after — verify SSH is always allowed first
- [ ] No automated Marzban DB backup
- [ ] No monitoring/alerting (server down = manual discovery)

---

## 💡 Ideas Parking Lot

Ideas mentioned but not yet scoped:

- **Subscription gifting** — send VPN access as gift (like Need VPN's "Подарить VPN" feature)
- **Family plan** — one subscription, multiple devices, multiple users
- **Corporate plan** — team management, usage analytics
- **Telegram Mini App** — in-app store instead of bot commands (like Gram VPN)
- **Loyalty system** — VPN Coins, bonus days, gamification
- **YouTube Premium routing** — special server optimized for YouTube without ads
- **Tor bridge** — access .onion sites through VPN (like Gram VPN)
- **Kill switch** — block all traffic if VPN drops
- **Split tunneling** — route only specific apps through VPN

---

*Last updated: March 2026*
*Infrastructure built by [@DeFiTON](https://github.com/DeFiTON)*
