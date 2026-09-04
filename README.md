# Concussion Recovery Pacer

A daily check-in assistant for concussion recovery built in n8n. Patient submits symptoms each day via Google Sheet. The pipeline checks for red flags, matches CDC HEADS UP return-to-activity guidelines, and delivers a personalized pacing recommendation to their inbox — in under 60 seconds. No app. No clinician required for routine days.

## What it does

1. **Patient fills Google Sheet** — symptom score (0-10), screen time, sleep, exercise attempted, notes. One row per day.
2. **Red Flag Checker fires first** — IF node checks symptom_score ≥ 8. If triggered: immediate emergency alert email with 911 and Concussion Alliance contacts. Pipeline stops. No LLM involved in safety decisions.
3. **Normal path: GPT-4o-mini agent** — receives patient data + full CDC HEADS UP 5-stage protocol inline. Determines the correct return-to-activity stage based on day number and symptom score. Outputs structured JSON.
4. **Personalized email delivered** — color-coded by protocol stage, with today's recommendation, what's next, and a protocol citation.

## Architecture

```
Google Sheets Trigger (Concussion_Checkin tab)
    ↓
Input Validator
    ↓
Red Flag Checker (IF node — symptom_score ≥ 8)
    ├── True  → Emergency Alert (Gmail) → Log Check-In
    └── False → Concussion Pacer Agent (GPT-4o-mini)
                    ↓
                Build Recovery Email
                    ↓
                Deliver Daily Summary (Gmail)
                    ↓
                Log Check-In
```

## CDC HEADS UP Return-to-Activity Stages

| Stage | Days | Score | Activity | Description |
|---|---|---|---|---|
| 1 | 1-2 | 7-10 | Complete Rest | No screens, no exercise, no school |
| 2 | 3-5 | 4-6 | Light Aerobic | Walking 20-30 min, stop if symptoms |
| 3 | 6-8 | 2-4 | Sport-Specific | Non-contact drills, screen time 2hr max |
| 4 | 9-12 | 0-2 | Non-Contact Drills | Full school, resistance training, need clearance |
| 5 | 13+ | 0 | Full Return | Written medical clearance required |

Source: CDC HEADS UP (public domain) + Buffalo Concussion Protocol

## Google Sheet Schema

Tab name: `Concussion_Checkin`

| Column | Description | Example |
|---|---|---|
| patient_id | Unique identifier | patient_001 |
| patient_email | Where daily email is delivered | patient@email.com |
| day_number | Recovery day number | 4 |
| symptom_score | 0 (none) to 10 (severe) | 3 |
| screen_hrs | Hours of screen time today | 1.5 |
| sleep_hrs | Hours of sleep last night | 8 |
| exercise_attempted | What was tried | light walking |
| vomiting | yes / no | no |
| vision_changes | yes / no | no |
| notes | Free text | headache mostly gone |

## Sample data (copy-paste CSV)

```csv
patient_id,patient_email,day_number,symptom_score,screen_hrs,sleep_hrs,exercise_attempted,vomiting,vision_changes,notes
patient_001,your@email.com,1,8,0,9,no,no,no,severe headache and sensitivity to light since injury
patient_002,your@email.com,4,3,1.5,8,light walking,no,no,headache mostly gone feeling much better today
patient_003,your@email.com,7,2,2,7.5,jogging 20 min,no,no,tried light jog felt okay mild pressure behind eyes
```

Row 1 triggers emergency alert (score 8). Rows 2-3 trigger normal pacing path.

## Setup

### Prerequisites
- n8n cloud account
- OpenAI API key (for GPT-4o-mini + text-embedding-3-small)
- Google account (Sheets trigger + Gmail delivery)

### Step 1 — Import workflows
1. n8n → Workflows → Import from file
2. Import `concussion_pacer_main_workflow.json`
3. Import `concussion_pacer_seed_kb_workflow.json`

### Step 2 — Set credentials
- OpenAI API key → Credentials → OpenAI
- Google Sheets → Credentials → Google Sheets Trigger OAuth2
- Gmail → Credentials → Gmail OAuth2

### Step 3 — Create Google Sheet
Create a sheet with tab `Concussion_Checkin` and the columns above. Copy Sheet ID from URL.

### Step 4 — Seed the KB
Open Seed KB workflow → Test Workflow. Wait for green checkmark. Re-run before every demo session (in-memory store resets on n8n restart).

### Step 5 — Activate and test
Toggle workflow active. Add a test row to the sheet. Within 60 seconds check inbox.

## n8n Workflow URLs
- Main: https://udayshankar.app.n8n.cloud/workflow/ulYYEbb3ZW380DpE
- Seed KB: https://udayshankar.app.n8n.cloud/workflow/KN4xxwTCPT3j2lea
