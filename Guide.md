Below is a wall-to-wall battle plan to take you from “zero” to a fully-working, production-grade AI networking agent that does the outreach, follow-ups, and relationship tracking for a new-grad SWE.
It’s split into seven phases (A → G).  Treat each phase as a milestone you can ship in 1-4 days.  Copy this into Notion/Jira and tick the boxes.
────────────────────────
PHASE A – REQUIREMENTS & GO/NO-GO (Day 0)
Define the exact outcome
• Goal: Land 5–10 coffee chats or referrals per month.
• Target personas:
– Alumni SWEs at tier-1 companies (FAANG, Unicorns).
– Hiring managers / team leads.
• Channels: LinkedIn (primary), Twitter (secondary), cold email (tertiary).
Compliance check
• LinkedIn ToS: no scraping of private data; use only public profiles + 1st-degree connections.
• Rate limits: 100 connection requests & 150 DMs/week.
Decide on human-in-the-loop (HITL) level
• V1: AI drafts → you click “Send”.
• V2 (later): Auto-send after approval queue.
Deliverable: One-page spec + green light.
────────────────────────
PHASE B – DATA STRATEGY & LEAD SOURCING (Days 1–2)
Build a lead schema
id | full_name | headline | company | school | mutual_group | profile_url | email | last_contact | status
Populate leads
Tier 1 (fastest) – CSV import from:
• LinkedIn Sales Navigator export (filter: “Past school = X”, “Title contains Software Engineer”).
Tier 2 (scalable) – APIs
• Proxycurl “Person Search” endpoint (Python snippet below).
• GitHub API: search users by location & language.
• Twitter API: followers of target companies’ eng accounts.
De-dupe & enrich
• Use pandas.drop_duplicates(subset=['profile_url']).
• Add missing emails with hunter.io or apollo.io.
Store in Postgres table leads or Airtable base.
Deliverable: ≥200 cleaned leads.
────────────────────────
PHASE C – PERSONALIZATION ENGINE (Days 2–3)
Message templates
a. Connection request (300 chars)
b. First DM (after accept)
c. Follow-up (after 3 days silence)
d. Thank-you after call
Prompt library (GPT-4)
• System prompt: “You are a polite new-grad SWE reaching out to {role} at {company}. Keep it under 50 words, reference 1 shared detail.”
• Variables: {first_name}, {school}, {project}, {tech_stack}.
Generate in batches
Python

Copy
import openai, pandas as pd
df = pd.read_csv("leads.csv")
for idx, row in df.iterrows():
    prompt = f"""
    Write a 50-word LinkedIn DM to {row.first_name}, {row.headline} who studied at {row.school}.
    Mention their recent post about {row.recent_post_topic} and politely ask for a 15-min career chat.
    """
    msg = openai.ChatCompletion.create(
        model="gpt-4", messages=[{"role":"user","content":prompt}]
    ).choices[0].message.content
    df.at[idx, "draft_msg"] = msg
df.to_csv("leads_with_drafts.csv", index=False)
Human review UI
• 50-line Streamlit page to scroll through drafts and click ✅ / ❌ / ✏️.
• Store approved messages in outbox table with send_at timestamp.
Deliverable: 100 approved, personalized messages ready to ship.
────────────────────────
PHASE D – DELIVERY AUTOMATION (Days 3–4)
Option 1 – Semi-auto (recommended for v1)
• Use LinkedIn “Sales Navigator” → “Message” → bulk upload approved messages via CSV (manual click).
Option 2 – Full-auto (Chrome + Playwright)
• Install undetected-chromedriver.
• Script logs in, loops over outbox, sends 10 per day, marks sent=True.
• Rotate proxies / 2Captcha if needed.
Cold email fallback
• Use AWS SES + yagmail to send plain-text from your own domain.
• DKIM & SPF records set up via Route53.
Deliverable: 50 messages sent in first batch.
────────────────────────
PHASE E – RESPONSE PIPELINE & CRM (Days 4–5)
Inbox listener
• Gmail API watch() webhook → when new reply arrives → push to CRM.
GPT-based triage
Prompt:

Copy
Categorize this reply as: 1=interested 2=busy 3=decline 4=needs-follow-up.
Extract any suggested meeting time.
Update lead status in Postgres.
Calendar bot
• If category=1, send Calendly link via DM/email.
• Use Calendly webhook to mark call_scheduled=True.
Dashboard
• Streamlit: funnel view (Sent → Replied → Call Booked → Referral).
• Export CSV for weekly retro.
Deliverable: Live dashboard with metrics.
────────────────────────
PHASE F – CONTINUOUS IMPROVEMENT LOOP (Week 2)
A/B test subject lines & CTAs.
Retarget non-responders after 10 days with new angle (e.g., share your latest project).
Add more lead sources: conference attendee lists, open-source contributors.
Fine-tune GPT prompts weekly based on response sentiment.
Add “warm intro” flow: if mutual connection exists, ask for intro first.
Deliverable: 2× improved reply rate (track via dashboard).
────────────────────────
PHASE G – SCALE & HARDEN (Weeks 3–4)
Containerize
• Dockerfile for Python scripts + cronjob to enqueue daily batch.
• Deploy on AWS Fargate (serverless).
Secrets & config
• Use AWS Secrets Manager for API keys, LinkedIn creds.
Observability
• CloudWatch alarms on failed sends.
• Sentry for Python exceptions.
Compliance
• Add “unsubscribe” link in cold emails.
• Honor LinkedIn 100-request weekly limit with exponential backoff.
Optional ML upgrade
• Fine-tune a lightweight model (LoRA on Llama-2-7B) on your best-performing messages to reduce GPT-4 cost.
Deliverable: Production agent running 24/7 with <5 min daily oversight.
────────────────────────
RUN-BOOK SUMMARY (Checklist)
[ ] Phase A spec approved
[ ] ≥200 leads in DB
[ ] GPT prompts & templates live
[ ] 50 messages sent
[ ] Dashboard tracking replies
[ ] A/B test 2 variants
[ ] Production deploy + alarms
────────────────────────
CODE REPO LAYOUT

Copy
networking-agent/
├── README.md
├── requirements.txt
├── src/
│   ├── lead_gen.py        # Proxycurl + dedup
│   ├── personalize.py     # GPT-4 message drafts
│   ├── review_ui.py       # Streamlit
│   ├── sender.py          # Playwright or SES
│   ├── inbox_listener.py  # Gmail webhook
│   └── dashboard.py       # Streamlit metrics
├── sql/
│   └── schema.sql
├── infra/
│   ├── Dockerfile
│   └── cloudformation.yaml
└── tests/
    └── test_sender.py

