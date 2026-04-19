# Phase 3 — n8n Fundamentals

## What n8n is and why it matters

n8n is the workflow automation engine that sits between 
all the other tools. It receives data, makes decisions, 
calls APIs, sends notifications, and logs results — 
all without manual intervention.

Every workflow follows the same pattern:
Trigger → Transform → Decide → Act → Log

Learn this pattern once and every workflow you ever 
build will follow it.

---

## Core concepts

| Concept | What it is |
|---|---|
| Trigger | What starts the workflow |
| Node | One step in the workflow |
| Execution | One complete run of the workflow |
| Input/Output | Data entering and leaving each node |
| Branching | IF node routes data down different paths |
| Credential | Stored auth material reused across workflows |

---

## The most important habit in n8n

After every single node runs — click on its execution 
result and inspect the output data. Ask:
- What fields came in?
- What fields left?
- What changed?
- Does the next node see what I think it does?

If you skip this habit n8n stays confusing.
If you build it n8n becomes transparent.

---

## Workflow 1 — Manual Trigger → Set → Gmail

### Purpose
Learn how data flows through a workflow. Understand 
field mapping. Send your first automated notification.

### What was built
- Manual Trigger node — the test button
- Set node — creates structured data fields
- Gmail node — sends an email using that data

### Screenshots
See screenshots folder

---

## Workflow 2 — Webhook → Branch by department → Email

### Purpose
Learn how real intake works. Understand branching 
logic. Route different inputs to different actions.

### What was built
- Webhook trigger — receives POST request with JSON
- IF node — checks department field value
- Two paths — different email sent per department
- Tested using hoppscotch.io (free browser tool)

### Screenshots
See screenshots folder

---

## What broke and how it was fixed

| Problem | Cause | Fix |
|---|---|---|
| IF node always false | Value comparison case-sensitive | Matched exact case of department string |
| Gmail node fails | OAuth not configured | Connected Gmail account through n8n credentials |
| Fields not resolving | Previous node did not run in same execution | Re-ran full workflow from trigger |
| Webhook not receiving | Wrong test URL used | Copied fresh URL from webhook node each session |

---

## What I learned

- Every node has input and output. Inspect both every time.
- Field names are exact. A typo silently breaks things.
- Branching is just an IF statement — one condition, 
two paths.
- Credentials are stored once in n8n and reused. 
They are never hardcoded in the workflow itself.
- The trigger determines everything — manual for 
testing, webhook for real requests, schedule for 
time-based runs.
- n8n execution logs show every run. Failed runs 
stay in the log so you can debug them.

---

## Screenshots in this folder

| File | What it shows |
|---|---|
| 01-first-workflow-canvas.png | Manual trigger + Set + Gmail nodes connected |
| 02-set-node-output.png | Set node execution showing field data |
| 03-gmail-node-config.png | Gmail node configured with field expressions |
| 04-email-received.png | Test email received in inbox |
| 05-webhook-workflow.png | Webhook + IF + two email paths |
| 06-hoppscotch-test.png | Test POST request sent from hoppscotch.io |
| 07-branch-true-path.png | Workflow executing the true/HR path |
| 08-break-test-typo.png | Field expression broken by intentional typo |
