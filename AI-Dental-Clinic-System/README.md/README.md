<div align="center">
<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=28&pause=1000&color=00D9FF&center=true&vCenter=true&width=700&lines=%F0%9F%A6%B7+AI+Dental+Clinic+System;Intelligent+Appointment+Automation;WhatsApp+%2B+AI+%2B+n8n+%2B+Supabase" alt="Typing SVG" />
<br/>
<img src="https://img.shields.io/badge/Status-Production%20Ready-00D9FF?style=for-the-badge&logo=checkmarx&logoColor=white"/>
<img src="https://img.shields.io/badge/Platform-n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white"/>
<img src="https://img.shields.io/badge/Database-Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white"/>
<img src="https://img.shields.io/badge/AI-OpenRouter-FF6B35?style=for-the-badge&logo=openai&logoColor=white"/>
<img src="https://img.shields.io/badge/Sync-Notion-000000?style=for-the-badge&logo=notion&logoColor=white"/>
<br/><br/>
 
<img src="https://media.giphy.com/media/qgQUggAC3Pfv687qPC/giphy.gif" width="460" alt="AI System Animation"/>
<br/>
> **A production-grade AI receptionist that handles clinic appointments end-to-end via WhatsApp — with zero human intervention.**
 
</div>
---
 
## 📌 What Is This?
 
The **AI Dental Clinic System** is a fully automated, intelligent clinic management system built on **n8n**. It acts as a smart AI receptionist that talks to patients via **WhatsApp**, understands their requests using AI, and handles the full appointment lifecycle — all synced in real time between **Supabase** (the database) and **Notion** (the staff dashboard).
 
No phone calls. No manual scheduling. No back-and-forth. Just a patient sending a WhatsApp message and getting their appointment confirmed automatically.
 
---
 
## 🏗️ System Architecture
 
```
WhatsApp Message
      │
      ▼
┌─────────────────────┐
│   n8n Main Router   │  ← Webhook trigger + customer tracking
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Intent Classifier  │  ← AI Agent (OpenRouter)
│  booking / modify / │     Confidence threshold: 90%
│  general / unclear  │
└─────────┬───────────┘
          │
    ┌─────┴──────────────────┐
    ▼         ▼              ▼
 Booking   Update         General
  Agent    Agent          Agent
    │         │              │
    ▼         ▼              ▼
Supabase  Supabase       Supabase
    │         │
    ▼         ▼
 Notion   Notion
  Sync     Sync
    │
    ▼
WhatsApp Response ← AI-formatted reply sent back to patient
```
 
---
 
## 🧠 AI Agents Breakdown
 
### 1. 🔀 Intent Classifier Agent
**File:** `ai-main-router.json`
 
The first AI brain in the system. Reads every incoming WhatsApp message and classifies it into one of 4 intents:
 
| Intent | Description | Example |
|---|---|---|
| `booking` | New appointment or checking available slots | "I want to book tomorrow at 3 PM" |
| `modify` | Change or reschedule existing appointment | "Move my appointment to Friday" |
| `general` | Clinic info, services, personal appointment lookup | "What are your working hours?" |
| `unclear` | Low confidence — triggers clarification message | Ambiguous or noisy input |
 
> Confidence threshold is **90%** — anything below triggers an automatic clarification request to the patient.
 
---
 
### 2. 📅 Booking Agent
**Files:** `booking-get-available-slots.json` + `booking-create-appointment.json`
 
Handles the full booking flow in two stages:
 
**Stage 1 — Get Available Slots**
- Checks `clinic_exceptions` table for holidays or clinic closures on that date
- Checks `clinic_schedule` table to verify if the day is active (not a day off)
- Fetches all existing appointments for that date from Supabase
- Runs a JavaScript algorithm to calculate all **free time gaps** between bookings
- Returns available slots to the AI agent
**Stage 2 — Create Appointment**
- Parses the customer's booking data (name, phone, email, date, time, service)
- Validates the requested date against exceptions and schedule
- Uses an **AI sub-agent** to fuzzy-match the service name (e.g., "teeth cleaning" → "تنظيف الأسنان")
- Calculates `end_time` based on service duration fetched from Supabase
- Validates requested time is within clinic working hours
- Checks for **slot conflicts** — ensures no double booking
- Creates the appointment in **Supabase** and syncs it to **Notion** in real time
---
 
### 3. ✏️ Update Agent
**Files:** `booking-update-simple.json` + `booking-update-critical.json`
 
Handles two types of updates with different logic:
 
**Simple Update** — name or phone number changes only
- Loops over all requested field changes
- Updates Supabase and syncs the change to the Notion dashboard
**Critical Update** — appointment rescheduling or service change
- Verifies the patient exists in the system using their phone number
- Checks the new requested date against exceptions and clinic schedule
- Uses an AI sub-agent to validate the requested service exists
- Recalculates end time based on new service duration
- Validates the new time slot is within working hours
- Checks for conflicts before committing the update
- Updates both **Supabase** and **Notion** atomically
---
 
### 4. 💬 General Info Agent
**File:** `ai-main-router.json` (General branch)
 
Answers any clinic-related question by querying:
- `clinic_information` table — location, working hours, general details
- `services` table — service list, descriptions, pricing
Also handles personal appointment lookups — if a patient asks "when is my appointment?", the agent asks for their phone number and queries `appointments` directly.
 
---
 
## 📋 Workflow Map
 
| Workflow | Type | Purpose |
|---|---|---|
| `ai-main-router` | Main | Entry point, intent routing, customer tracking |
| `booking-get-available-slots` | Booking | Compute free time slots for a given date |
| `booking-create-appointment` | Booking | Full appointment creation with validation |
| `booking-update-simple` | Update | Update name or phone number |
| `booking-update-critical` | Update | Reschedule appointment or change service |
| `Patient Appointment Check` | Query | Look up personal appointment by phone |
| `sync-bookings-from-notion` | Sync | Notion → Supabase (appointment changes) |
| `sync-clinic-info-from-notion` | Sync | Notion → Supabase (clinic info updates) |
| `sync-clinic-schedule-from-notion` | Sync | Notion → Supabase (schedule changes) |
| `sync-exceptions-from-notion` | Sync | Notion → Supabase (holidays/exceptions) |
| `sync-services-from-notion` | Sync | Notion → Supabase (services/pricing) |
 
---
 
## 🛢️ Database Schema (Supabase)
 
```
appointments
  ├── id
  ├── customer_name
  ├── customer_phone
  ├── customer_email
  ├── appointment_date
  ├── start_time
  ├── end_time
  ├── Service
  └── status
 
clinic_schedule
  ├── notion_id
  ├── day_of_week
  ├── start_time
  ├── end_time
  └── active (bool)
 
clinic_exceptions
  ├── notion_id
  ├── exception_date
  ├── reason
  └── closed (bool)
 
services
  ├── notion_id
  ├── service_name
  ├── price
  ├── duration_minutes
  ├── description
  └── active (bool)
 
clinic_information
  ├── notion_id
  ├── info_key
  └── info_value
 
customers
  ├── customer_name
  ├── phone_number
  └── last_seen
```
 
---
 
## 🔄 Notion ↔ Supabase Sync
 
Staff manage everything from **Notion** — no technical skills required. Any change in Notion triggers a real-time sync to Supabase:
 
```
Notion Dashboard (Staff)
       │
       │  polling trigger (every 1–3 min)
       ▼
  n8n Sync Workflow
       │
       ├── Record exists in Supabase? → UPDATE
       └── Record is new?            → CREATE
```
 
**What staff can manage from Notion:**
- Clinic working hours per day
- Days off and public holidays
- Service catalog (name, price, duration)
- Clinic general information
- Appointment status updates
---
 
## ⚙️ Tech Stack
 
<div align="center">
| Layer | Technology |
|---|---|
| Workflow Engine | [n8n](https://n8n.io) |
| AI Models | OpenRouter (GPT-4o, GPT-5.1) |
| Database | [Supabase](https://supabase.com) (PostgreSQL) |
| Admin Dashboard | [Notion](https://notion.so) |
| Communication | WhatsApp Business API |
| Memory | PostgreSQL Chat Memory (per user session) |
| Scripting | JavaScript (Node.js inside n8n) |
 
</div>
---
 
## 🧩 Key Engineering Decisions
 
**1. Confidence-gated routing** — The intent classifier only routes a message if confidence > 90%. Below that, the system asks the patient to clarify. This prevents wrong routing.
 
**2. AI-powered service matching** — Instead of exact-match lookups, a dedicated AI sub-agent fuzzy-matches service names. So "teeth whitening", "تبييض الأسنان", and "whitening" all correctly resolve to the same service record.
 
**3. Dual-write architecture** — Every write happens in Supabase first, then immediately syncs to Notion. This keeps Supabase as the source of truth while giving staff a human-readable interface.
 
**4. Per-user conversation memory** — Each patient's conversation history is stored in PostgreSQL using n8n's Postgres Chat Memory node, keyed by their WhatsApp number. This makes multi-turn conversations work naturally.
 
**5. Conflict detection before booking** — The system checks for exact slot conflicts using `start_time + end_time + appointment_date` before committing any booking or update. This prevents double-booking even under concurrent requests.
 
---
 
## 🚀 System Visuals
 
### Main Router
![Main Router](./assets/main-ai-router.png)
 
### Booking Workflows
 
<table>
  <tr>
    <td align="center"><b>Available Slots</b></td>
    <td align="center"><b>Create Appointment</b></td>
  </tr>
  <tr>
    <td><a href="./assets/booking-get-available-slots.png"><img src="./assets/booking-get-available-slots.png" width="420"/></a></td>
    <td><a href="./assets/booking-create-appointment.png"><img src="./assets/booking-create-appointment.png" width="420"/></a></td>
  </tr>
</table>
### Update Workflows
 
<table>
  <tr>
    <td align="center"><b>Simple Update</b></td>
    <td align="center"><b>Critical Update</b></td>
  </tr>
  <tr>
    <td><a href="./assets/booking-update-simple.png"><img src="./assets/booking-update-simple.png" width="420"/></a></td>
    <td><a href="./assets/booking-update-critical.png"><img src="./assets/booking-update-critical.png" width="420"/></a></td>
  </tr>
</table>
### Notion Sync Layer
![Sync](./assets/sync-notion-clinic-update.png)
 
### Live WhatsApp Conversations
 
<table>
  <tr>
    <td align="center"><b>Conversation 1</b></td>
    <td align="center"><b>Conversation 2</b></td>
  </tr>
  <tr>
    <td><a href="./assets/ai-agent-whatsapp-conversation-1.png"><img src="./assets/ai-agent-whatsapp-conversation-1.png" width="320"/></a></td>
    <td><a href="./assets/ai-agent-whatsapp-conversation-2.png"><img src="./assets/ai-agent-whatsapp-conversation-2.png" width="320"/></a></td>
  </tr>
</table>
---
 
## ⚠️ Current Limitations
 
- Uses demo/test data — not connected to a live clinic
- No patient authentication layer (e.g., OTP confirmation)
- No appointment reminder system (SMS/WhatsApp reminders 24h before)
- Prompt layer can be further hardened for edge cases in production
---
 
## 🔧 Production Readiness
 
The architecture is production-ready. To go live, only three layers need to be swapped:
 
```
1. Data layer     → Replace demo data with real clinic data
2. Auth layer     → Add OTP or confirmation code system
3. Deploy layer   → Host n8n on a VPS or cloud instance
```
 
Everything else — routing logic, conflict detection, sync layer, AI agents — is already built for production scale.
 
---
 
<div align="center">
<img src="https://media.giphy.com/media/LnQjpWaON8nhr21vNW/giphy.gif" width="60"/>
**Built with intention.**
 
<br/><br/>
 
<a href="https://www.linkedin.com/in/omar-emad-99a919326/">
  <img src="https://img.shields.io/badge/Omar%20Emad-LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"/>
</a>
&nbsp;&nbsp;
<a href="https://github.com/gxxzvb1005-creator">
  <img src="https://img.shields.io/badge/GitHub-Profile-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/>
</a>
</div>
