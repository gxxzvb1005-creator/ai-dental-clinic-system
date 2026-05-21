# AI Dental Clinic System

## Overview

AI Dental Clinic System is a workflow-based automation system designed to simulate and manage real clinic operations using AI agents, n8n workflows, Supabase, and WhatsApp integration.

The system is built as a production-ready architecture prototype that can be extended into a fully deployed real-world product with minimal changes in the data layer.

---

## Core Idea

The system acts as an AI-powered clinic receptionist that handles:

- Appointment booking
- Schedule management
- Patient inquiries
- Data updates
- Real-time synchronization with staff tools (Notion)

---

## System Architecture

WhatsApp → n8n Webhook → AI Main Router → Intent Classification → Specialized Workflows → Supabase → Response

---

## AI Main Router

The Main Router is responsible for:

- Detecting user intent (booking / update / general / unclear)
- Assigning confidence score
- Routing requests to correct workflow
- Asking clarification when confidence < 90

---

## Booking System

Handles full appointment lifecycle:

- Fetch available slots
- Validate clinic schedule constraints
- Prevent overlapping bookings
- Create confirmed appointments

This ensures strict and reliable scheduling logic.

---

## Update System

### Simple Update
- Patient name
- Phone number

### Critical Update
- Appointment rescheduling
- Service modification
- Duration recalculation
- Conflict detection

Critical updates require full validation to ensure schedule integrity.

---

## Information Layer

Connected to Supabase:

- Clinic services (price, duration)
- Clinic general information (location, working hours)
- Used as source of truth for AI responses

---

## External Sync (Notion)

- Staff use Notion dashboard for updates
- Changes are synced in real-time to Supabase
- Enables non-technical management of clinic data

---

## AI Behavior Model

- Intent-based routing
- Confidence threshold system
- Context-aware decision making

---

## Limitations

- Uses demo data instead of production data
- No authentication layer implemented
- No confirmation code system yet
- Prompt layer can be further optimized for production robustness

---

## Production Readiness Note

The system is fully designed with production-grade architecture.

Only data, authentication, and deployment layers are required to convert it into a live production system.

---

## Tech Stack

- n8n (workflow automation)
- OpenAI (AI agents)
- Supabase (database)
- Notion (admin dashboard)
- WhatsApp API (communication layer)

---

## System Visuals

All workflows and real interactions are documented below.

### Main Router
![Main Router](./assets/main-ai-router.png)

### Booking - Available Slots
![Slots](./assets/booking-get-available-slots.png)

### Booking - Create Appointment
![Booking](./assets/booking-create-appointment.png)

### Simple Update
![Simple Update](./assets/booking-update-simple.png)

### Critical Update
![Critical Update](./assets/booking-update-critical.png)

### Sync Layer
![Sync](./assets/sync-notion-clinic-update.png)

### AI Chat Example
![Chat](./assets/ai-agent-whatsapp-conversation-1.png)