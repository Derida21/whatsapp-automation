# WhatsApp AI Automation

A Docker-based development setup for building WhatsApp AI automation using **WAHA**, **n8n**, and **Cloudflare Tunnel**.

This project provides a local environment where WhatsApp messages can be received through WAHA, processed by n8n workflows and AI Agents, and exposed through a public HTTPS endpoint using Cloudflare Tunnel.

## Tech Stack

| Component                        | Purpose                                                    |
| -------------------------------- | ---------------------------------------------------------- |
| [n8n](https://n8n.io)            | Workflow automation and AI Agent orchestration             |
| [WAHA](https://waha.devlike.pro) | Unofficial WhatsApp HTTP API based on WhatsApp Web         |
| Cloudflare Tunnel                | Exposes local n8n webhooks through a public HTTPS endpoint |
| Docker                           | Container runtime for all services                         |
| Custom Domain                    | Public HTTPS endpoint for n8n                              |

## Architecture

```text
                         Internet
                            │
                            │ HTTPS
                            ▼
                 YOUR_SUBDOMAIN.YOUR_DOMAIN
                            │
                            ▼
                   Cloudflare Tunnel
                            │
                            │ HTTP
                            ▼
                      cloudflared
                            │
                            ▼
                    ┌───────────────┐
                    │      n8n      │
                    │    :5678      │
                    └───────┬───────┘
                            │
                            │ HTTP
                            ▼
                    ┌───────────────┐
                    │     WAHA      │
                    │    :3000      │
                    └───────────────┘
```

The public connection uses HTTPS, while Cloudflare Tunnel communicates with the n8n container through the internal Docker network using HTTP.

---

# Prerequisites

Before starting, make sure you have:

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
* A Cloudflare account
* A domain managed by Cloudflare
* A WhatsApp number for testing
* Basic familiarity with Docker and terminal commands

> **Recommendation:** Use a dedicated WhatsApp number for testing instead of your primary daily-driver number.

---

# Configuration ⚙️

## 1. Create a Cloudflare Tunnel

Create a **Named Tunnel** from the Cloudflare Dashboard.

After creating the tunnel:

1. Open the tunnel configuration.
2. Go to **Public Hostnames**.
3. Add a hostname for n8n.
4. Configure the service as:

```text
Type: HTTP
URL: n8n:5678
```

For example:

```text
Hostname:
YOUR_SUBDOMAIN.YOUR_DOMAIN
```

### Important

Use:

```text
HTTP → n8n:5678
```

Do **not** use:

```text
HTTPS → n8n:5678
```

The n8n container serves HTTP internally. HTTPS is handled by Cloudflare on the public side.

The public URL remains:

```text
https://YOUR_SUBDOMAIN.YOUR_DOMAIN
```

---

# 2. Clone the Repository

```bash
git clone <THIS_REPOSITORY_URL>
cd whatsapp-ai-automation
```

---

# 3. Create the Environment File

Copy the example environment file:

```bash
cp .env.example .env
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

---

# 4. Configure `.env`

Open `.env` and configure the required variables:

```env
# n8n
N8N_DATA_PATH=YOUR_ROOT_DIRECTORY/n8n-data
N8N_HOST=YOUR_SUBDOMAIN.YOUR_DOMAIN

# Cloudflare Tunnel
CLOUDFLARE_TUNNEL_TOKEN=YOUR_CLOUDFLARE_TUNNEL_TOKEN

# WAHA
WAHA_DATA_PATH=YOUR_ROOT_DIRECTORY/waha-data
WAHA_API_KEY=YOUR_API_KEY
WAHA_DASHBOARD_USERNAME=YOUR_USERNAME
WAHA_DASHBOARD_PASSWORD=YOUR_PASSWORD

# Timezone
GENERIC_TIMEZONE=Asia/Jakarta
TZ=Asia/Jakarta
```

Example Windows paths:

```env
N8N_DATA_PATH=C:/Development/n8n_data
WAHA_DATA_PATH=C:/Development/waha_data
```

Example public hostname:

```env
N8N_HOST=n8n.example.com
```

> **Security:** Never commit `.env` to Git. It contains private credentials and your Cloudflare Tunnel token.

---

# 5. Start the Stack

Start all services:

```bash
docker compose up -d
```

Check the container status:

```bash
docker compose ps
```

You should see three services:

```text
n8n
waha
cloudflared
```

running.

---

# 6. Access n8n

Open your public Cloudflare URL:

```text
https://YOUR_SUBDOMAIN.YOUR_DOMAIN
```

Complete the n8n owner account setup.

Because n8n is configured with the public HTTPS URL, OAuth integrations such as Google credentials can use the public callback URL.

The callback URL follows this format:

```text
https://YOUR_SUBDOMAIN.YOUR_DOMAIN/rest/oauth2-credential/callback
```

---

# WAHA Setup

## 1. Open WAHA Dashboard

WAHA is available locally at:

```text
http://localhost:3000
```

Log in using the credentials configured in `.env`.

---

## 2. Configure the API Key

Use the API key configured in:

```env
WAHA_API_KEY=YOUR_API_KEY
```

---

## 3. Create a WhatsApp Session

From the WAHA dashboard:

1. Create a new session.
2. Start the session.
3. Scan the QR code using WhatsApp.
4. Wait until the session is connected.

> Use a dedicated WhatsApp number for testing when possible.

---

# Connect WAHA to n8n

## 1. Create a WAHA Trigger Workflow

Open n8n and create a new workflow.

Add the **WAHA Trigger** node.

The node provides webhook URLs for test and production modes.

Copy the appropriate webhook URL.

---

## 2. Configure WAHA Webhooks

In the WAHA dashboard:

1. Open your WhatsApp session.
2. Open **Configuration**.
3. Go to **Webhooks**.
4. Add the n8n webhook URL.
5. Save the configuration.

The message flow is:

```text
WhatsApp
    │
    ▼
   WAHA
    │
    │ Webhook
    ▼
Cloudflare Tunnel
    │
    ▼
   n8n
    │
    ▼
AI Agent / Automation
```

---

# Configure WAHA Credential in n8n

For WAHA action nodes, configure the credential using:

```text
Host:
http://waha:3000

API Key:
YOUR_API_KEY
```

### Why `waha:3000`?

All services are connected to the same Docker network:

```text
local_net
```

Docker allows containers to communicate using their service names.

Therefore:

```text
n8n → http://waha:3000
```

works internally.

Do not use:

```text
http://localhost:3000
```

from inside the n8n container because `localhost` refers to the n8n container itself.

---

# Project Structure

```text
whatsapp-ai-automation/
│
├── docker-compose.yml
├── .env.example
├── .env
├── .gitignore
├── README.md
│
├── n8n-data/
│   └── # n8n persistent data
│
└── waha-data/
    └── # WAHA WhatsApp session data
```

The actual data directories are determined by the paths configured in `.env`.

Example:

```text
YOUR_ROOT_DIRECTORY/
│
├── n8n-data/
└── waha-data/
```

---

# Useful Docker Commands

### Start

```bash
docker compose up -d
```

### Stop

```bash
docker compose down
```

### Restart

```bash
docker compose restart
```

### Check status

```bash
docker compose ps
```

### View all logs

```bash
docker compose logs -f
```

### View n8n logs

```bash
docker logs -f n8n
```

### View WAHA logs

```bash
docker logs -f waha
```

### View Cloudflare Tunnel logs

```bash
docker logs -f cloudflared
```

---

# Troubleshooting

## n8n Cannot Be Accessed Through the Domain

Check the Cloudflare Tunnel logs:

```bash
docker logs cloudflared
```

The origin service should be:

```text
http://n8n:5678
```

Not:

```text
https://n8n:5678
```

---

## Google OAuth Callback Does Not Work

Make sure the following configuration matches your Cloudflare hostname:

```env
N8N_HOST=YOUR_SUBDOMAIN.YOUR_DOMAIN
N8N_PROTOCOL=https
WEBHOOK_URL=https://YOUR_SUBDOMAIN.YOUR_DOMAIN/
N8N_EDITOR_BASE_URL=https://YOUR_SUBDOMAIN.YOUR_DOMAIN/
```

The OAuth callback should be:

```text
https://YOUR_SUBDOMAIN.YOUR_DOMAIN/rest/oauth2-credential/callback
```

---

## n8n Cannot Connect to WAHA

Use:

```text
http://waha:3000
```

instead of:

```text
http://localhost:3000
```

Check that both containers are connected to:

```text
local_net
```

---

# Security

* Never commit `.env` to Git.
* Never expose the Cloudflare Tunnel token.
* Use a strong WAHA API key.
* Use a strong WAHA dashboard password.
* Use a dedicated WhatsApp number for testing.
* Keep Docker Desktop updated.
* Keep container images updated.
* Back up n8n and WAHA persistent data.

Example `.gitignore`:

```gitignore
.env
n8n-data/
waha-data/
```

---

# Production Considerations

This repository is primarily designed for **local development, experimentation, and portfolio purposes**.

For production deployments, consider:

* PostgreSQL instead of SQLite for n8n
* Automated backups
* Pinned container versions instead of `latest`
* Persistent storage
* Monitoring and health checks
* Proper secret management
* Resource limits
* Database backup and recovery procedures

For larger workloads, additional infrastructure such as Redis and n8n queue workers may be considered.

---

# License

This project is intended for learning, development, experimentation, and portfolio purposes.
