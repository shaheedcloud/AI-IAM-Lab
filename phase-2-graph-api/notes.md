# Phase 2 — Microsoft Graph API

## What is Microsoft Graph and why it matters

Microsoft Graph is the API layer that lets you control 
Microsoft Entra ID programmatically. Everything you can 
do manually in the Entra portal — creating users, 
assigning groups, disabling accounts — Graph API lets 
you do through code and automation.

Without Graph, your IAM work stays manual forever.
With Graph, n8n can control Entra automatically.

This is the bridge between identity management and 
automation.

---

## The mental model

Every API system follows this same pattern:

| Piece | What it is | In this lab |
|---|---|---|
| Identity | Who the caller is | iam-lab-automation app |
| Permission | What they can do | User.Read.All, GroupMember.ReadWrite.All |
| Token | Proof of authentication | OAuth2 access token from Microsoft |
| Endpoint | What you are calling | graph.microsoft.com |
| Response | What comes back | JSON data about users, groups |

Learn this pattern once here and you will never be 
confused by API authentication again. It repeats in 
OpenAI, Google, Slack, and every other API you will 
ever use.

---

## Step 1 — App Registration

An app registration is the identity of your automation.
Instead of logging in as a human, your workflow logs 
in as an app. This is how machine-to-machine access 
works in enterprise environments.

**What was created:**
- App name: `iam-lab-automation`
- Account type: Single tenant (this organization only)
- State: Activated

**Three values noted from the overview page:**
- Application (client) ID — identifies the app
- Directory (tenant) ID — identifies the organization
- Object ID — the app's identity object in Entra

See screenshot: `01-app-registration-overview.png`

**Why this matters:**
A real company would create a separate app registration 
for every automated system. Each one gets only the 
permissions it needs. If one is compromised, the 
damage is limited to its permission scope only.

---

## Step 2 — API Permissions

Permissions define what the app is allowed to do.
You request them, but they do not work until an admin 
grants consent. This two-step process is intentional 
— it prevents apps from silently gaining access.

**Permissions added:**
- `User.Read.All` — Application type — allows the app 
to read all user profiles in the directory
- `GroupMember.ReadWrite.All` — Application type — 
allows the app to add and remove group members

**Before consent:**
Both permissions were listed but showed "Not granted 
for Default Directory" with yellow warning icons.
The app existed but had zero power.

See screenshot: `02-api-permissions-not-granted.png`

**After admin consent:**
Both permissions turned green — "Granted for Default 
Directory." The app is now authorized to act.

See screenshot: `03-api-permissions-granted.png`

**Why this matters:**
Application permissions are powerful. They act without 
a signed-in user. This means the automation can run 
at 3am with no human present and still make changes 
to your directory. That power requires admin consent 
deliberately — it should never be automatic.

---

## Step 3 — Client Secret

A client secret is the app's password. It proves the 
app's identity when requesting an access token from 
Microsoft.

**What was created:**
- Description: `iam-lab-secret`
- Expiry: 24 months (expires April 2028)

**Critical security rule learned:**
The secret value is shown exactly once. The moment 
you leave the page it is hidden forever. If lost, 
you must create a new one. This is by design — 
Microsoft never stores your secret in readable form.

See screenshot: `04-client-secret.png`
(Secret value blurred — never commit real secrets 
to a public repository)

**Why this matters:**
In production environments, secrets are stored in 
secure vaults like Azure Key Vault, not in text files 
or code. For this lab, the secret is stored locally 
only and never pushed to GitHub.

---

## Step 4 — First Live Graph API Call

Using Graph Explorer at graph.microsoft.com to test 
the connection before touching n8n.

**Read action tested:**
`GET https://graph.microsoft.com/v1.0/users`

This returns a JSON list of all users in the tenant.

See screenshot: `05-graph-explorer-users.png`

**Why read before write:**
Read actions are safe to test. They confirm that:
- The app registration is working
- The token acquisition is working  
- The permissions are sufficient
- The request syntax is correct

Only after a successful read do you attempt a write.

**Write action tested:**
Adding a user to a group via Graph API.

`POST https://graph.microsoft.com/v1.0/groups/
{groupId}/members/$ref`

See screenshot: `06-graph-write-group-assignment.png`

---

## What broke and how it was fixed

| Error | Cause | Fix |
|---|---|---|
| 401 Unauthorized | Wrong or expired token | Regenerate token with correct credentials |
| 403 Forbidden | Permission not granted | Grant admin consent in Entra portal |
| 404 Not Found | Wrong user or group ID | Copy IDs directly from Entra portal |

---

## What I learned

- An app registration is a machine identity. It is 
not a human login. It exists so automation can 
authenticate without a person being present.
- Permissions must be both requested AND consented. 
Requesting alone does nothing.
- Application permissions are more powerful than 
delegated permissions and must be granted carefully.
- Tokens expire. Every API call must include a valid 
token or it will be rejected with a 401 error.
- Read before write. Always validate your setup with 
a safe read action before attempting changes.
- The app registration pattern repeats in every API 
system. Learn it once, use it everywhere.

---

## Screenshots in this folder

| File | What it shows |
|---|---|
| 01-app-registration-overview.png | App registration created with client ID and tenant ID visible |
| 02-api-permissions-not-granted.png | Permissions added but not yet consented |
| 03-api-permissions-granted.png | Admin consent granted, all permissions active |
| 04-client-secret.png | Client secret created (value blurred) |
| 05-graph-explorer-users.png | First successful Graph API read call |
| 06-graph-write-group-assignment.png | Successful write call adding user to group |

## Troubleshooting note — Graph Explorer tenant issue

Graph Explorer defaulted to personal Microsoft account 
context because the Entra tenant was created with a 
Gmail-based admin account. This caused a display error 
in Graph Explorer even though the API returned HTTP 200.

Lesson learned: HTTP 200 means the call succeeded. 
Always check the status code first. UI warnings can 
be misleading.

Resolution: Moved directly to testing Graph API calls 
through n8n HTTP Request nodes using app registration 
credentials — which is the correct production approach 
anyway. Graph Explorer is only a convenience tool.
