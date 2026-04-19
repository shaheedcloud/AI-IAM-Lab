# Phase 1 — Identity Foundation

## What I built
- 5 users: Sarah Mitchell, Marcus Johnson, David Chen, 
Priya Patel, James Okafor
- 3 groups: HR-Team, IT-Team, Finance-Team
- Group assignments matching each user's department
- MFA enabled on admin account (Shaheed Ibrahim)
- James Okafor disabled to simulate offboarding

## What I learned
- A user is an identity object. A group is a management 
container. A role gives administrative capability.
- Assigning access to groups instead of individuals 
makes it scalable and auditable.
- Disabling preserves the full audit trail. Deleting 
destroys it.
- MFA protects accounts even when passwords are stolen.
- Least privilege means every user gets only what they 
need for their job. Nothing more.

## What broke
- Per-user MFA button was not visible inside individual 
user profiles. It only appears in the All Users list view.
- MFA was already enforced on the Global Administrator 
account by Microsoft automatically.

## How I fixed it
- Navigated to the All Users list to find the Per-user 
MFA button in the toolbar.
- Confirmed admin MFA status was already enabled and 
moved on.

## Screenshots
See /phase-1-identity/screenshots/ folder
