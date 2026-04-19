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

## Workflow 1 — Manual Trigger → Edit Fields → Gmail

### Purpose
Learn how data flows through a workflow. Understand 
field mapping. Send your first automated notification.

### What was built
- Manual Trigger node — starts the workflow manually 
during testing. Called "When clicking Execute workflow" 
in this version of n8n.
- Edit Fields node (formerly called Set node) — creates 
three structured data fields: name, email, department
- Gmail node — sends an email using the field data 
from the previous node via expressions like 
`{{ $json.name }}` and `{{ $json.department }}`

### Setting up Gmail credentials — what actually happened

Connecting Gmail to n8n on a self-hosted local instance 
is not a one-click process. It requires creating a 
Google Cloud OAuth credential manually. Here is exactly 
what was done:

**Step 1 — Created a Google Cloud project**
- Went to console.cloud.google.com
- Created a new project named `n8n-iam-lab`

**Step 2 — Enabled the Gmail API**
- Searched for Gmail API in the library
- Clicked Enable

**Step 3 — Configured OAuth consent screen**
- Went to Google Auth Platform → Get started
- Set app name to `n8n-iam-lab`
- Set audience to External
- Added shayahim@gmail.com as a test user
- This step is required before creating OAuth credentials

**Step 4 — Created OAuth Client ID**
- Went to Clients → Create client
- Application type: Web application
- Name: n8n-gmail
- Added authorized redirect URI exactly:
  `http://localhost:5678/rest/oauth2-credential/callback`
- Copied Client ID and Client Secret

**Step 5 — Connected in n8n**
- Pasted Client ID and Client Secret into n8n 
Gmail credential fields
- Clicked Sign in with Google
- First attempt failed with Error 403: access_denied 
because the Gmail account was not added as a test user
- Fixed by adding the account in Google Auth Platform 
→ Audience → Test users
- Second attempt succeeded — "Account connected" confirmed

**Step 6 — Enabled Gmail API**
- First workflow execution failed with "Forbidden" error
- Error message showed Gmail API was not enabled
- Went to Google Cloud Console → Library → Gmail API 
→ Enable
- Re-ran the workflow — succeeded

### What the workflow produced
- Subject: `n8n test - Test User`
- Body: `New user detected. Department: IT - 
Email: testuser@example.com`
- Email arrived in inbox within seconds of execution

---

## Break tests performed

### Break test 1 — Lowercase department value
Changed department field value from `IT` to `it`.
Ran the workflow.

Result: Email sent successfully but body showed 
"Department: it" — lowercase passed through silently 
with no warning or error.

Why this matters: When IF node branching checks for 
"IT" and receives "it" — it will route to the wrong 
path. Case sensitivity is invisible and dangerous.

See screenshot: `07-break-test-lowercase.png`

### Break test 2 — Changed name value
Changed name field from `Test User` to `Test Usher`.
Ran the workflow.

Result: Subject line showed "n8n test - Test Usher" — 
the expression pulled exactly what was in the field 
with no validation.

Why this matters: n8n does not validate your data. 
Whatever enters a field is exactly what flows through. 
Garbage in means garbage out. Input validation must 
be built deliberately.

See screenshot: `08-break-test-name-TestUsher.png`

---

## What broke and how it was fixed

| Problem | Cause | Fix |
|---|---|---|
| Gmail credential Error 403 | Gmail account not added as test user in Google Cloud | Added account in Google Auth Platform → Audience → Test users |
| Gmail node Forbidden error | Gmail API not enabled in Google Cloud project | Enabled Gmail API in Google Cloud Console Library |
| Client ID popup closed before copying | Navigated away too quickly | Retrieved Client ID from Clients page. Reset secret to get new Client Secret value |
| Graph Explorer showing wrong tenant | Signed in with personal Microsoft account not Entra org account | Documented as known limitation. Moved to n8n HTTP calls instead |

---

## What I learned

- Every node has input and output. Inspect both every 
time before moving to the next node.
- Field names and values are exact. A typo or wrong 
case silently breaks things downstream.
- Self-hosted n8n requires manual OAuth setup for 
Google services. This is a one-time process per 
credential type.
- The Google Cloud Console requires Gmail API to be 
explicitly enabled even after OAuth credentials are 
created. Two separate steps.
- Credentials are stored once in n8n and reused across 
all workflows. They are never hardcoded in the workflow.
- The trigger determines everything — manual for 
testing, webhook for real intake, schedule for 
time-based runs.
- HTTP 200 means the call succeeded. Always check the 
status code first before reading UI warnings.
- Break tests reveal assumptions. Running the workflow 
with wrong data shows exactly where validation is 
missing before it causes problems in production.

---

## Screenshots in this folder

| File | What it shows |
|---|---|
| 00-n8n-ready.png | n8n running at localhost:5678 ready for first workflow |
| 01-manual-trigger-added.png | Empty canvas with Manual Trigger node added |
| 02-set-node-output.png | Edit Fields node showing three fields in OUTPUT panel |
| 03-gmail-credential-connected.png | Gmail OAuth credential showing Account connected |
| 04-gmail-node-config.png | Gmail node configured with To, Subject and Message expressions |
| 05-gmail-send-success.png | Gmail node OUTPUT showing message ID and SENT label |
| 06-email-received.png | Actual email received in inbox with correct field values |
| 07-break-test-lowercase.png | Email body showing lowercase department value passed through |
| 08-break-test-name-TestUsher.png | Subject showing wrong name value passed through unchanged |
