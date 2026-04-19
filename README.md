# AI-IAM-Lab

A practical, hands-on lab where I built an AI-powered 
Identity and Access Management automation system from 
zero using Microsoft Entra ID, n8n, and OpenAI.

This is not a tutorial follow-along. Every workflow was 
built, broken, fixed, and documented by hand.

---

## What this project does

- Automates user onboarding and offboarding using 
Microsoft Entra ID and Microsoft Graph API
- Uses n8n to orchestrate identity workflows without 
manual intervention
- Adds an AI layer (OpenAI) to evaluate access requests 
and score suspicious login events
- Logs every action to a Google Sheets audit trail
- Includes a human approval workflow for escalated 
decisions

---

## Stack

| Tool | Purpose |
|---|---|
| Microsoft Entra ID | Identity source of truth |
| Microsoft Graph API | Programmatic control of Entra |
| n8n (self-hosted) | Workflow automation engine |
| OpenAI API | AI-assisted decision making |
| Google Sheets | Audit logging |
| Gmail | Alerts and notifications |

---

## Phases

### Phase 1 — Identity Foundation
Built and managed users, groups, roles, MFA, and 
account lifecycle in Microsoft Entra ID manually.
Learned identity lifecycle, least privilege, and why 
disable matters more than delete.

### Phase 2 — Microsoft Graph API
Created an app registration, configured permissions, 
acquired tokens, and made live API calls to control 
Entra ID programmatically.

### Phase 3 — n8n Fundamentals
Built first workflows in n8n. Learned triggers, nodes, 
data flow, branching, and how to debug executions.

### Phase 4 — Onboarding Automation
Built a full new-hire onboarding workflow: webhook 
intake, department routing, Graph API group assignment, 
welcome email, and audit logging.

### Phase 5 — AI Access Decisions
Connected OpenAI to evaluate access requests. AI 
returns APPROVE, DENY, or ESCALATE with a reason. 
Workflow branches and acts accordingly.

### Phase 6 — Suspicious Login Handler
Built a security workflow that receives login events, 
scores them with AI, disables accounts on high risk, 
alerts the team, and logs everything.

### Phase 7 — Enterprise Refinement
Added error handling, retry logic, approval workflows, 
offboarding automation, and weekly reporting.

---

## What I learned

- IAM is not just creating users. It is designing 
controlled, auditable access at scale.
- Graph API is the control plane for everything 
Microsoft identity related.
- n8n makes complex multi-step automation accessible 
without writing full code.
- AI works best in automation when it is bounded by 
clear policy and structured outputs.
- Every automated action must leave evidence. Logging 
is not optional.

---

## Project status

In progress — building phase by phase.

---

*Built by Shaheed Hayatuddin | shaheedcloud*
