# 📦 Shipment Tracker Bot — Setup Guide

Automated shipment tracking across multiple forwarders, every 2 hours, with Telegram notifications **only when there's an update**.

---

## Prerequisites

- **Docker Desktop** installed → https://www.docker.com/products/docker-desktop
- A **Telegram account**
- ~5 minutes

---

## Step 1 — Create Your Telegram Bot

1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Give it a name, e.g. `My Shipment Tracker`
4. Give it a username, e.g. `myshipments_bot`
5. BotFather replies with your **Bot Token** — looks like:
   ```
   7123456789:AAHxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   ```
   **Save this token.**

---

## Step 2 — Get Your Telegram Chat ID

1. Search Telegram for **@userinfobot** and start it
2. It immediately replies with your **Chat ID** — a number like `123456789`
3. **Save this Chat ID**

> 💡 If you want notifications in a **group chat**, add your bot to the group, then send a message in the group and visit:
> `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
> Find the `"chat": {"id": ...}` value (will be negative for groups, e.g. `-1001234567890`).

---

## Step 3 — Configure Environment

```bash
cd /Users/bryanteo/Documents/ME/Ling/shipment-tracker-bot
cp .env.example .env
```

Edit `.env` and fill in:

```env
N8N_USER=admin
N8N_PASSWORD=your_strong_password_here
TELEGRAM_CHAT_ID=123456789
```

---

## Step 4 — Start n8n

```bash
docker compose up -d
```

Wait ~20 seconds, then open: **http://localhost:5678**

Log in with the credentials you set in `.env`.

---

## Step 5 — Add Telegram Credentials in n8n

1. Go to **Settings → Credentials → Add Credential**
2. Search for **Telegram**
3. Paste your **Bot Token** from Step 1
4. Click **Save**

---

## Step 6 — Set TELEGRAM_CHAT_ID as n8n Variable

1. Go to **Settings → Variables**
2. Click **+ Add Variable**
3. Name: `TELEGRAM_CHAT_ID`
4. Value: your Chat ID (e.g. `123456789`)
5. Save

---

## Step 7 — Import the Workflow

1. In n8n, go to **Workflows → Import from File**
2. Select `shipment-tracker-workflow.json` from this folder
3. The workflow loads with all nodes pre-configured

---

## Step 8 — Link Telegram Credential

1. Click the **"Send Telegram Notification"** node
2. In the **Credential** dropdown, select the Telegram credential you created
3. Close the node

---

## Step 9 — Test It

1. Click **"Execute Workflow"** (▶ button)
2. You should receive a Telegram message for each shipment
   (first run always fires — it's recording the baseline)
3. Check the execution log to confirm no errors

---

## Step 10 — Activate

Toggle the workflow to **Active** (top-right switch).

It will now run automatically **every 2 hours** and only notify you when something changes. ✅

---

## Adding More Shipments

Edit the **"Shipment List"** node. Add entries to the array:

```json
[
  {
    "id": "DX52001620915",
    "forwarder": "podtrack",
    "label": "Thailand Shipment",
    "trackUrl": "http://47.52.146.142/podtrack/Details.aspx?ID=DX52001620915"
  },
  {
    "id": "MP0092805689",
    "forwarder": "austway",
    "label": "Australia Shipment",
    "trackUrl": "https://track.austway-cargo.com"
  },
  {
    "id": "YOUR_NEW_WAYBILL",
    "forwarder": "podtrack",
    "label": "Another Parcel",
    "trackUrl": "http://47.52.146.142/podtrack/Details.aspx?ID=YOUR_NEW_WAYBILL"
  }
]
```

Supported forwarder values: `"podtrack"` | `"austway"`

---

## Useful Docker Commands

```bash
docker compose logs -f n8n     # live logs
docker compose down             # stop
docker compose restart n8n      # restart
docker compose pull && docker compose up -d  # update n8n
```

---

## Cloud Hosting (Free 24/7)

### Oracle Cloud Always Free (Recommended)
1. Sign up at https://cloud.oracle.com/free (credit card needed for verification, never charged)
2. Create an ARM VM (4 CPU, 24GB RAM — free forever)
3. Install Docker on the VM:
   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER
   ```
4. Copy this project to the VM and follow Steps 3–10 above

---

## How It Works

| Component | Role |
|-----------|------|
| Schedule Trigger | Fires every 2 hours |
| Shipment List → Split | Iterates each shipment individually |
| Switch (Route by Forwarder) | Directs to correct scraper |
| HTTP (Podtrack) | GET HTML page, parsed with regex |
| HTTP (Austway) | GET /api/tms/order-traces/listTracking → JSON |
| Check For Update | Fingerprint comparison using n8n Static Data |
| IF node | Only passes when update detected |
| Telegram node | Sends formatted Markdown message |
| Store Fingerprint | Updates Static Data baseline |

---

## Example Telegram Message

```
🇦🇺 Shipment Update — Austway Cargo (AU)
━━━━━━━━━━━━━━
Waybill: `MP0092805689`
Ref: `8905532425`
Status: *DELIVERING*
🕐 30 Jul 2026, 10:46 AM
📍 Sydney

_Your parcel is in our care. Your parcel has arrived at our Aramex depot._

🔗 Track Shipment
```
