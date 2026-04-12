# Edge Node Setup Guide

This guide covers setting up a Marzban edge node — the "dirty transit" server that passes user traffic without storing any user data.

## Why a Separate Edge Node?

- **Security:** If edge node is seized/compromised, no user data is exposed
- **Privacy:** Node doesn't know who users are, only passes encrypted traffic
- **Scalability:** Add more nodes in different locations without touching master
- **Expendable:** Can be destroyed and recreated in minutes

---

## Requirements

- Hetzner CX22, Ubuntu 24.04
- Same SSH key as master (or new key added)
- Master server already running with Marzban panel

---

## Step 1: SSH Into Edge Node

```bash
ssh root@EDGE_IP
```

---

## Step 2: Install Marzban Node

```bash
sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban-node.sh)" @ install
```

When prompted:
```
Please paste the content of the Client Certificate, press ENTER on a new line when finished:
```

→ Paste the certificate from Marzban panel → Node Settings → **Download certificate** button

```
Do you want to use REST protocol? (Y/n): y    ← MUST be y
Enter the SERVICE_PORT (default 62050):        ← press Enter
Enter the XRAY_API_PORT (default 62051):       ← press Enter
```

---

## Step 3: Update docker-compose.yml

The installer creates the file but may not have all required settings. Overwrite with correct config:

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
```

> ⚠️ **Critical:** `SERVICE_PROTOCOL: "rest"` is required. Without it, connections fail with `RemoteDisconnected` error.

---

## Step 4: Place Correct SSL Certificate

The most common source of connection errors is the wrong certificate.

```bash
# The certificate MUST come from Marzban panel → Node Settings → Download certificate button
# NOT from openssl self-signed generation
# File name MUST be ssl_client_cert.pem

cat > /var/lib/marzban-node/ssl_client_cert.pem << 'EOF'
-----BEGIN CERTIFICATE-----
[paste certificate from panel here]
-----END CERTIFICATE-----
EOF
```

Verify file exists:
```bash
ls -la /var/lib/marzban-node/
# Should show: ssl_client_cert.pem, ssl_cert.pem, ssl_key.pem
```

---

## Step 5: Open Firewall Ports

```bash
# Critical: use --force to avoid unicode prompt issue in Termius
ufw --force enable
ufw allow 22      # SSH — always first!
ufw allow 62050   # Marzban node service port
ufw allow 62051   # XRay API port
ufw allow 2053    # VLESS+Reality (or whatever port you set)
ufw allow 443     # Optional: if using port 443 for VLESS (conflicts with Caddy on master)
ufw reload
```

---

## Step 6: Start Node

```bash
marzban-node restart
```

Verify it's running:
```bash
marzban-node status
# Expected: Status: Up, marzban-node: running

ss -tlnp | grep 62050
# Expected: LISTEN 0 2048 0.0.0.0:62050
```

---

## Step 7: Connect Node to Marzban Panel

1. Open `https://panel.yourdomain.com/dashboard`
2. Click Settings (gear icon) → Node Settings
3. Click **Add New Marzban Node**
4. Fill in:
   - **Name:** `edge1-helsinki` (or descriptive name)
   - **Address:** `EDGE_NODE_IP`
   - **Port:** `62050`
   - **API Port:** `62051`
5. ✅ Check **Add this node as a new host for every inbound**
6. Click **Add Node**

Expected result: Node shows `Connected` status with Xray version.

---

## Troubleshooting Node Connection

### Error: `TLSV1_ALERT_UNKNOWN_CA`
→ Wrong certificate in `ssl_client_cert.pem`
→ Must use certificate from panel's **Download certificate** button

### Error: `Connection aborted, RemoteDisconnected`
→ `SERVICE_PROTOCOL` not set to `"rest"` in docker-compose.yml
→ Fix: add `SERVICE_PROTOCOL: "rest"` and restart

### Error: `Unable to connect to node` (in Marzban logs)
→ Firewall blocking port 62050
→ Fix: `ufw allow 62050 && ufw reload`

### Node installs but docker-compose.yml is missing
→ Previous broken install left traces
→ Fix: `rm -rf /opt/marzban-node` then reinstall

---

## Verifying Node Works

From master server:
```bash
curl -v --insecure https://EDGE_IP:62050
# Should see TLS handshake and HTTP 405 response (normal, means node is responding)
```

In Marzban logs on master:
```bash
marzban logs | grep -i "edge\|node"
# Should show: Connected to "edge1-helsinki" node, xray run on v25.x.x
```

---

## Adding More Nodes

Repeat this entire guide for each new server. Common locations for Russian audience:

| Location | Expected latency (from Russia) |
|----------|-------------------------------|
| Helsinki, FI | 30–60ms |
| Frankfurt, DE | 50–80ms |
| Nuremberg, DE | 50–80ms |
| Amsterdam, NL | 60–90ms |
| Singapore, SG | 150–200ms (for Asian diaspora) |
