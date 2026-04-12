# Telegram Bot Setup

## Admin Bot (Marzban Built-in)

The built-in Marzban Telegram bot serves as your admin panel.

### Setup

1. Create a bot via [@BotFather](https://t.me/BotFather): `/newbot`
2. Copy the token
3. Add to `/opt/marzban/.env`:
   ```env
   TELEGRAM_API_TOKEN = "your_token"
   TELEGRAM_ADMIN_ID = your_telegram_id
   ```
4. `marzban restart`
5. Send `/start` to your bot

### Available Commands

| Command | Description |
|---------|-------------|
| `/start` | Open admin panel |
| `/user username` | Get/edit user |
| System Info | Server stats |
| Users | List all users |
| Create User | Add new user manually |
| Restart Xray | Restart VPN core |

---

## Customer Bot (Railway — Main Product)

The customer-facing bot handles sales, payments, and key delivery.

### Features Roadmap

- `/start` — onboarding flow
- Free trial (subscribe to channel → get N free days)
- Purchase subscription (Telegram Stars / Tegro.Money)
- Auto-create user in Marzban via API
- Deliver subscription link / QR code
- 2-level referral program
- Promo code activation
- Partner white-label (custom bot token)

### Tech Stack

- **Language:** Python (aiogram 3)
- **Hosting:** Railway
- **Database:** SQLite on Railway Volume
- **Payments:** Telegram Stars + [Tegro.Money](https://tegro.money/docs/en/)
- **Distribution:** [Libermall Marketplace](https://libermall.com/) promo codes

### Marzban API Integration

Base URL: `https://panel.yourdomain.com/api`

Key endpoints:
```
POST /admin/token          — get auth token
POST /user                 — create user
GET  /user/{username}      — get user info
DELETE /user/{username}    — delete user
GET  /user/{username}/subscription — get subscription link
```

### Payment Flow

```
User clicks "Buy" 
  → Select plan (1/3/6/12 months)
  → Choose payment (Stars / TON / USDT)
  → Payment confirmed
  → Bot calls Marzban API → create user
  → Bot sends subscription link + QR
  → User imports in V2Box/V2RayNG
```

### Deployment on Railway

```bash
# Procfile
web: python bot.py
```

Environment variables on Railway:
```
BOT_TOKEN=customer_bot_token
MARZBAN_URL=https://panel.yourdomain.com
MARZBAN_USERNAME=admin_username
MARZBAN_PASSWORD=admin_password
```
