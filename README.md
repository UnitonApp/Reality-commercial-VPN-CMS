# 🔒 1984 VPN — Self-Hosted Commercial VPN Infrastructure

> *"Big Brother is watching. We're watching back."*
> A complete, production-ready VPN business infrastructure built by [@DeFiTON](https://github.com/DeFiTON)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Marzban](https://img.shields.io/badge/Panel-Marzban-green.svg)](https://github.com/Gozargah/Marzban)
[![Protocol](https://img.shields.io/badge/Protocol-VLESS%2BReality-orange.svg)](https://github.com/XTLS/Xray-core)
[![Author](https://img.shields.io/badge/Author-DeFiTON-purple.svg)](https://gusev.biz)

---

## 👤 About the Author

This project is built and maintained by **Sviatoslav Gusev** — entrepreneur, developer, and founder of multiple tech products.

| | |
|--|--|
| 🌐 Website | [gusev.biz](https://gusev.biz) |
| 💬 Telegram | [@defiton](https://t.me/defiton) |
| 🐙 GitHub | [@DeFiTON](https://github.com/DeFiTON) |
| 🏢 Company | [Libermall LLC](https://libermall.com) |

### My Projects

| Project | Description |
|---------|-------------|
| [TonChat.AI](https://tonchat.ai) | AI chat on TON blockchain |
| [Tegro.Finance](https://tegro.finance) | DEX on TON |
| [Tegro.Money](https://tegro.money) | Payment system |
| [SMOService](https://smoservice.media) | SMM panel |
| [Libermall](https://libermall.com) | Digital marketplace |

---

## 📖 What is 1984 VPN?

**1984 VPN** is a complete infrastructure stack for launching a commercial VPN service, optimized for the Russian-speaking market where standard protocols (WireGuard, OpenVPN) are blocked by Roskomnadzor's DPI systems.

### Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Protocol | VLESS+Reality | Only protocol reliably bypassing Russian DPI |
| Panel | Marzban | Best open-source VPN management panel |
| Hosting | Hetzner Helsinki | Low latency to Russia, good price/performance |
| Distribution | Telegram Bot | Standard for Russian VPN market |
| Payments | Telegram Stars + TON | Works without Russian bank cards |

---

## 🏗️ Architecture

```
User (iOS/Android/Windows)
         │
         │ VLESS+Reality (port 2053)
         ▼
┌─────────────────────┐
│  EDGE NODE          │  ← Hetzner Helsinki
│  XRay Core          │  ← "Dirty transit", no logs
│  No user data       │  ← Expendable if seized
└──────────┬──────────┘
           │ Marzban Node Protocol (port 62050)
           ▼
┌─────────────────────┐
│  MASTER SERVER      │  ← Hetzner Helsinki
│  Marzban Panel      │  ← panel.1984vpn.com
│  Caddy (SSL proxy)  │
│  SQLite Database    │
└──────────┬──────────┘
           │ REST API
           ▼
┌─────────────────────┐
│  TELEGRAM BOT       │  ← Railway.app
│  Customer Sales     │
│  Payment Processing │
│  Key Delivery       │
└─────────────────────┘
```

---

## 💰 Ecosystem Integration

This project integrates with my existing product ecosystem:

### Payment Processing
Balance top-up is powered by **[Tegro.Money](https://tegro.money)** — a payment system supporting TON, USDT, and other cryptocurrencies.

API Documentation: [tegro.money/docs/en/](https://tegro.money/docs/en/)

```python
# Example: Create payment via Tegro.Money
import requests

response = requests.post("https://tegro.money/api/v1/createOrder", json={
    "shop_id": "YOUR_SHOP_ID",
    "amount": 299,
    "currency": "RUB",
    "order_id": "vpn_sub_12345"
})
```

### Key Distribution
VPN activation codes are distributed through **[Libermall Marketplace](https://libermall.com/)** — a digital goods marketplace where partners can buy and resell VPN keys.

---

## ⚡ Quick Setup

### 1. Clone & Setup

```bash
git clone https://github.com/DeFiTON/1984vpn.git
cd 1984vpn
```

### 2. Deploy Master Server (Hetzner CX22, Ubuntu 24.04)

```bash
# Install Marzban
sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban.sh)" @ install

# Create admin
marzban cli admin create --sudo

# Setup Caddy SSL proxy
apt install -y caddy
cat > /etc/caddy/Caddyfile << 'EOF'
panel.yourdomain.com {
    reverse_proxy localhost:8000
}
EOF
systemctl restart caddy
```

### 3. Configure Environment

```bash
cat >> /opt/marzban/.env << 'EOF'
TELEGRAM_API_TOKEN = "your_admin_bot_token"
TELEGRAM_ADMIN_ID = your_telegram_id
NODE_CLIENT_CERT_FILE = "/var/lib/marzban/certs/client.pem"
NODE_CLIENT_KEY_FILE = "/var/lib/marzban/certs/client.key"
EOF

# Generate node SSL certs
mkdir -p /var/lib/marzban/certs
openssl req -x509 -newkey rsa:4096 \
  -keyout /var/lib/marzban/certs/client.key \
  -out /var/lib/marzban/certs/client.pem \
  -days 3650 -nodes -subj "/CN=marzban-client"

marzban restart
```

### 4. Deploy Edge Node

```bash
# On edge server
sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban-node.sh)" @ install

# IMPORTANT: Use certificate from panel → Node Settings → Download certificate
# Save to: /var/lib/marzban-node/ssl_client_cert.pem

cat > /opt/marzban-node/docker-compose.yml << 'EOF'
services:
  marzban-node:
    image: gozargah/marzban-node:latest
    restart: always
    network_mode: host
    environment:
      SSL_CERT_FILE: "/var/lib/marzban-node/ssl_cert.pem"
      SSL_KEY_FILE: "/var/lib/marzban-node/ssl_key.pem"
      SSL_CLIENT_CERT_FILE: "/var/lib/marzban-node/ssl_client_cert.pem"
      SERVICE_PROTOCOL: "rest"
      SERVICE_PORT: "62050"
      XRAY_API_PORT: "62051"
    volumes:
      - /var/lib/marzban-node:/var/lib/marzban-node
EOF

ufw --force enable && ufw allow 22 && ufw allow 62050 && ufw allow 62051 && ufw allow 2053 && ufw reload
marzban-node restart
```

### 5. Configure VLESS+Reality

```bash
# Generate keypair on master
docker exec marzban-marzban-1 xray x25519
# Save Private key and Public key
```

Paste the XRay config from `configs/xray-config.json` into Marzban Core Settings, insert your private key.

---

## 📱 Client Apps for Users

| Platform | Recommended App |
|----------|----------------|
| iOS | [V2Box](https://apps.apple.com/app/v2box-v2ray-client/id6446814690) or Streisand |
| Android | [V2RayNG](https://play.google.com/store/apps/details?id=com.v2ray.ang) |
| Windows | [v2rayN](https://github.com/2dust/v2rayN) or Hiddify |
| macOS | FoXray or V2Box |

---

## 🗺️ Roadmap

- [x] Marzban panel deployment
- [x] VLESS+Reality protocol
- [x] Edge node (dirty transit)
- [x] SSL via Caddy
- [x] Admin Telegram bot
- [ ] Customer Telegram bot (Railway + aiogram 3)
- [ ] Telegram Stars payment
- [ ] Tegro.Money payment integration
- [ ] 2-level referral program
- [ ] Promo code system (Libermall distribution)
- [ ] White-label (partner bot tokens)
- [ ] Landing page (1984vpn.com)
- [ ] Mobile app
- [ ] Browser extension

---

## 📂 Repository Structure

```
1984vpn/
├── configs/
│   ├── Caddyfile                 # Caddy reverse proxy
│   ├── xray-config.json          # VLESS+Reality config
│   ├── docker-compose.node.yml   # Edge node compose
│   └── marzban.env               # Environment template
├── docs/
│   ├── deployment.md             # Full deployment guide
│   ├── node-setup.md             # Edge node setup
│   └── bot-setup.md              # Telegram bot setup
├── bot/                          # Customer bot (WIP)
│   └── README.md
├── LICENSE
└── README.md
```

---

## 🔑 Key Learnings

Documented here for future reference:

1. **WireGuard is blocked in Russia** — use VLESS+Reality only
2. **Port 443 conflicts with Caddy** — use port 2053 for VLESS
3. **Node SSL cert** — must use `ssl_client_cert.pem` from panel's "Download certificate" button, NOT auto-generated certs
4. **Three-server architecture** — Master / Bot / Edge, never combine them
5. **`SERVICE_PROTOCOL: "rest"`** — required in node docker-compose for proper connection

---

## 📄 License

MIT — see [LICENSE](LICENSE)

---

*Built with ❤️ by [Sviatoslav Gusev](https://gusev.biz) | [@defiton](https://t.me/defiton)*
