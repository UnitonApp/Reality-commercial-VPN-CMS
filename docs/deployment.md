# Deployment Guide

## Prerequisites

- 2× Hetzner VPS (CX22, Ubuntu 24.04, Helsinki region)
- Domain registered on Cloudflare (e.g. `1984vpn.com`)
- SSH key pair generated on your machine
- Telegram Bot token from [@BotFather](https://t.me/BotFather)

---

## Step 1: Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "1984vpn"
cat ~/.ssh/id_ed25519.pub
```

Add the public key to both Hetzner servers during creation.

---

## Step 2: Create Hetzner Servers

Create two servers with these settings:
- **Type:** CX22 (2 vCPU, 4GB RAM)
- **OS:** Ubuntu 24.04
- **Location:** Helsinki (eu-central)
- **Names:** `master-1984vpn` and `edge1-1984vpn`

---

## Step 3: Configure DNS on Cloudflare

Add these A records pointing to your master server IP:

| Type | Name | Value | Proxy |
|------|------|-------|-------|
| A | `@` | `MASTER_IP` | ✅ Proxied |
| A | `panel` | `MASTER_IP` | ✅ Proxied |
| A | `www` | `MASTER_IP` | ✅ Proxied |

---

## Step 4: Install Marzban on Master

```bash
ssh root@MASTER_IP

sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban.sh)" @ install

# Create admin user
marzban cli admin create --sudo
# Username: your_admin_name
# Password: strong_password
# Telegram ID: your_telegram_id
```

---

## Step 5: Install & Configure Caddy

```bash
apt install -y caddy

cat > /etc/caddy/Caddyfile << 'EOF'
panel.yourdomain.com {
    reverse_proxy localhost:8000
}
EOF

systemctl restart caddy
```

---

## Step 6: Configure Marzban Environment

```bash
cat >> /opt/marzban/.env << 'EOF'
TELEGRAM_API_TOKEN = "your_bot_token"
TELEGRAM_ADMIN_ID = your_telegram_id
NODE_CLIENT_CERT_FILE = "/var/lib/marzban/certs/client.pem"
NODE_CLIENT_KEY_FILE = "/var/lib/marzban/certs/client.key"
EOF

# Generate SSL certs for node connection
mkdir -p /var/lib/marzban/certs
openssl req -x509 -newkey rsa:4096 \
  -keyout /var/lib/marzban/certs/client.key \
  -out /var/lib/marzban/certs/client.pem \
  -days 3650 -nodes -subj "/CN=marzban-client"

marzban restart
```

---

## Step 7: Setup Edge Node

```bash
ssh root@EDGE_IP

sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban-node.sh)" @ install
```

When prompted for the Client Certificate, paste the contents of your certificate from Marzban panel → Node Settings → **Download certificate**.

Save it at `/var/lib/marzban-node/ssl_client_cert.pem`.

Configure docker-compose.yml:
```bash
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

marzban-node restart
```

Open firewall ports:
```bash
ufw --force enable
ufw allow 22
ufw allow 80
ufw allow 443
ufw allow 2053
ufw allow 62050
ufw allow 62051
ufw reload
```

---

## Step 8: Connect Node to Panel

1. Open `https://panel.yourdomain.com/dashboard`
2. Go to Settings → Node Settings
3. Click **Add New Marzban Node**
4. Fill in:
   - **Name:** `edge1-helsinki`
   - **Address:** `EDGE_IP`
   - **Port:** `62050`
   - **API Port:** `62051`
5. Check **Add this node as a new host for every inbound**
6. Click **Add Node**

---

## Step 9: Configure VLESS+Reality

Generate private key:
```bash
docker exec marzban-marzban-1 xray x25519
```

In Marzban panel → Core Settings, replace the configuration with the template from `configs/xray-config.json`, inserting your generated private key.

Click **Save** → **Restart Core**.

---

## Step 10: Test

1. Create a test user in Marzban panel
2. Click the QR code icon
3. Scan with V2RayNG (Android) or V2Box (iOS)
4. Enable VPN and verify your IP changed

---

## Troubleshooting

### Node connection fails
- Verify `ssl_client_cert.pem` contains the certificate from panel's **Download certificate** button
- Check ports 62050/62051 are open on edge node
- Restart both: `marzban restart` and `marzban-node restart`

### Port 443 conflict with Caddy
- Use port 2053 for VLESS instead of 443
- Caddy occupies 443 for SSL termination

### Xray fails to start
- Check Core Settings JSON is valid
- Port 443 conflict — change to 2053

---

## Scaling

To add more edge nodes, repeat Step 7-8 for each new server. Marzban supports unlimited nodes.

Recommended locations for Russian users:
- 🇫🇮 Helsinki (lowest latency)
- 🇩🇪 Frankfurt / Nuremberg
- 🇳🇱 Amsterdam
