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
Google Cloud OAuth credential manually.

**Step 1 — Created a Google Cloud project**
- Went to console.cloud.google.com
- Created a new project named n8n-iam-lab

**Step 2 — Enabled the Gmail API**
- Searched for Gmail API in the Google Cloud Library
- Clicked Enable
- Important: this is a separate step from OAuth setup.
  Both must be done or the Gmail node will fail.

**Step 3 — Configured OAuth consent screen**
- Went to Google Auth Platform → Get started
- Set app name to n8n-iam-lab
- Set audience to External
- Added shayahim@gmail.com as a test user
- This step is required before creating OAuth credentials
- Without adding your account as a test user, Google 
  returns Error 403: access_denied

**Step 4 — Created OAuth Client ID**
- Went to Clients → Create client
- Application type: Web application
- Name: n8n-gmail
- Added authorized redirect URI:
  http://localhost:5678/rest/oauth2-credential/callback
- Copied Client ID and Client Secret into n8n

**Step 5 — Connected in n8n**
- Pasted Client ID and Client Secret into Gmail 
  credential fields in n8n
- Clicked Sign in with Google
- First attempt: Error 403 access_denied — fixed by 
  adding Gmail account as test user in Google Auth 
  Platform → Audience → Test users
- Second attempt: succeeded — Account connected confirmed

**Step 6 — First execution failed**
- Error: Forbidden — Gmail API not enabled
- Fix: went to Google Cloud Library → searched Gmail 
  API → clicked Enable
- Waited 30 seconds then re-ran — succeeded

### What the completed workflow produced
- Subject: n8n test - Test User
- Body: New user detected. Department: IT - 
  Email: testuser@example.com
- Email arrived in inbox within seconds of execution

### Break tests performed on Workflow 1

**Break test 1 — Lowercase department value**
Changed department field from IT to it (lowercase).
Result: Email sent successfully but body showed 
"Department: it" with no warning or error.
Lesson: n8n passes values through exactly as they are.
Case sensitivity is invisible. An IF node checking for 
"IT" would silently route "it" to the wrong path.
See: 07-break-test-lowercase.png

**Break test 2 — Wrong name value**
Changed name field from Test User to Test Usher.
Result: Subject showed "n8n test - Test Usher" — 
expression pulled the wrong value with no validation.
Lesson: Garbage in means garbage out. n8n does not 
validate your data. Input validation must be built 
deliberately into every workflow.
See: 08-break-test-name-TestUsher.png

---

## Workflow 2 — Webhook → IF → Department Routing

### Purpose
Learn how real intake works using a webhook trigger.
Understand branching logic. Route different inputs 
to different automated actions based on a field value.

### What was built
- Webhook node — receives POST requests at a URL.
  Replaces the manual trigger with real external intake.
  Path set to new-hire. Method set to POST.
  Test URL: http://localhost:5678/webhook-test/new-hire
- IF node — evaluates the department field from the 
  incoming webhook body. Condition: department equals HR.
  Creates two paths: true (HR) and false (all others).
- Gmail node on true path — fires HR onboarding email
- Gmail node on false path — fires non-HR notification

### Key difference from Workflow 1

In Workflow 1, data came from an Edit Fields node.
Field references used: `{{ $json.name }}`

In Workflow 2, data comes from a webhook POST body.
Field references must use: `{{ $json.body.name }}`

The data is nested inside a body object when it arrives 
via webhook. Using `$json.name` instead of 
`$json.body.name` would silently return empty values.

### Critical discovery — expression mode in Gmail node

When filling in the Subject field of the Gmail node,
typing `{{ $json.body.name }}` in field mode treats 
it as plain text and does not resolve it.

The field must be switched to expression mode first
by clicking the expression toggle (fx icon) before 
typing the expression. Once in expression mode the 
preview below the field shows the resolved value.

This was discovered when the first test email arrived 
with the literal text `{{ $json.body.name }}` in the 
subject instead of the actual name. Switched to 
expression mode and re-tested — resolved correctly.

### Testing with Hoppscotch

Used hoppscotch.io (free browser-based API tool) to 
send POST requests to the webhook URL.

**HR test — true path:**
```json
{
  "name": "Sarah Mitchell",
  "email": "sarah.mitchell@company.com",
  "department": "HR"
}
```
Result: Email received — Subject: HR Onboarding - 
Sarah Mitchell. True path fired. False path did not run.
See: 14-workflow2-hr-execution.png and 
15-hr-email-received.png

**IT test — false path:**
```json
{
  "name": "David Chen",
  "email": "david.chen@company.com",
  "department": "IT"
}
```
Result: Email received — Subject: Non-HR Onboarding — 
David Chen. False path fired. True path did not run.
See: 14-workflow2-it-execution.png and 
15-it-email-received.png

---

## Complete troubleshooting log

| Problem | Cause | Fix |
|---|---|---|
| Gmail Error 403 access_denied | Gmail account not added as test user | Added account in Google Auth Platform → Audience → Test users |
| Gmail node Forbidden error | Gmail API not enabled in Google Cloud | Enabled Gmail API in Google Cloud Console Library |
| Client secret popup closed early | Navigated away before copying | Retrieved Client ID from Clients page. Reset secret to get new value. |
| Expression not resolving in subject | Subject field was in field mode not expression mode | Clicked fx toggle to switch to expression mode before typing expression |
| IF node typo — dpartment | Missed the letter e when typing department | Caught before testing. Fixed expression to $json.body.department |
| Graph Explorer showing wrong tenant | Signed in with personal Microsoft account | Known limitation. Documented and moved to n8n HTTP calls instead |

---

## What I learned

- Every node has input and output. Inspect both every 
  time before moving to the next node.
- Field names and values are exact. A typo or wrong 
  case silently breaks downstream routing.
- Webhook data arrives nested inside a body object. 
  Always use $json.body.fieldname not $json.fieldname 
  when the trigger is a webhook.
- Gmail fields must be in expression mode to resolve 
  dynamic expressions. Field mode treats them as text.
- Self-hosted n8n requires manual Google Cloud OAuth 
  setup. This involves two completely separate steps: 
  creating OAuth credentials AND enabling the Gmail API.
- The Google Cloud consent screen requires your own 
  account to be added as a test user when the app is 
  in testing mode.
- Credentials are stored once in n8n and reused across 
  all workflows. Never hardcoded in the workflow itself.
- IF node branching is binary — one condition, two 
  paths. Only the matching path executes. The other 
  path stays grey in the execution view.
- Break tests reveal assumptions before they become 
  production problems.

---

## Workflow files

| File | What it contains |
|---|---|
| workflow-1-basic-notification.json | Manual trigger to Edit Fields to Gmail |
| workflow-2-department-routing.json | Webhook to IF branching to two Gmail paths |

Import either file into any n8n instance to see the 
workflow structure directly.

---

## Screenshots in this folder

| File | What it shows |
|---|---|
| 00-n8n-ready.png | n8n running at localhost:5678 |
| 01-manual-trigger-added.png | Empty canvas with Manual Trigger node |
| 02-set-node-output.png | Edit Fields output showing three fields |
| 03-gmail-credential-connected.png | Gmail OAuth showing Account connected |
| 04-gmail-node-config.png | Gmail node with expressions configured |
| 05-gmail-send-success.png | Gmail OUTPUT showing SENT confirmation |
| 06-email-received.png | Test email in inbox with correct field values |
| 07-break-test-lowercase.png | Lowercase department passed through silently |
| 08-break-test-name-TestUsher.png | Wrong name value in subject line |
| 09-webhook-node-config.png | Webhook node configured POST new-hire path |
| 10-if-node-config.png | IF condition checking department equals HR |
| 11-gmail-true-path-hr.png | HR Gmail node on true path configured |
| 12-gmail-false-path-other.png | Non-HR Gmail node on false path configured |
| 13-workflow2-full-canvas.png | Full canvas showing all four nodes connected |
| 14-workflow2-it-execution.png | Canvas showing false path execution green |
| 15-it-email-received.png | Non-HR email received with correct values |
