# WhatsApp Payment Reminder Automation

A self-hosted payment reminder and renewal alert system for internet service billing, built with **n8n**, **Evolution API**, **PostgreSQL**, and **Redis**, and deployed as a Render Blueprint via GitHub. It sends automated WhatsApp reminders to unpaid customers and notifies the service owner when payments are completed—without relying on paid third-party messaging APIs.

---

## Tech Stack & Components Used

* **n8n (`n8nio/n8n:latest`):** Workflow automation engine that schedules checks, reads customer/payment records, filters unpaid vs. paid accounts, and triggers HTTP requests.
* **Evolution API (`evoapicloud/evolution-api:latest`):** Self-hosted WhatsApp gateway connected to a linked WhatsApp session instance (`isp-billing-bot` / `isp-billing-bot-pro`) to send WhatsApp text messages via HTTP POST requests (`POST /message/sendText/{instance}`).
* **PostgreSQL (`postgres_db`):** Relational database used for persistent storage by n8n and Evolution API.
* **Redis / Key Value (`redis-cache`):** In-memory cache and session state store used by Evolution API (`CACHE_REDIS_ENABLED=true`).
* **Render & GitHub (`render.yaml`):** Infrastructure-as-code Blueprint configuration synced with GitHub for automated cloud deployments (`n8n-app`, `whatsapp-evolution`, `redis-cache`, and `postgres_db`).
* **Data Source (Google Sheets / Billing Sheet):** Tracks customer details such as Customer Name, Mobile Number, Amount, Plan Number, Reminder Date, and Payment Status (`Pending` / `Paid`).

---

## How It Works

```text
[ Scheduled Trigger (e.g., Daily at 10:00 AM / Periodic Check) ]
                         │
                         ▼
         [ Fetch Customer & Payment Records ]
                         │
           ┌─────────────┴─────────────┐
           ▼                           ▼
 [ Unpaid: Status == "Pending" ]   [ Paid: Status == "Paid" ]
           │                           │
           ▼                           ▼
 [ Check Due Window:           ]   [ Send Renewal Alert to Owner ]
   2 days before, 1 day before,  ]   via Evolution API
   or on the reminder date       ]
           │
           ▼
 [ Send Customer Reminder via Evolution API ]
   POST /message/sendText/{instance}
