# onboarding_bcs

Automate onboarding process.

## Daily overdue Microsoft To-Do email reminder

You can do this with **Power Automate** + **Microsoft Graph** in one scheduled cloud flow.

### What this flow does
- Runs once per day (for example, 6:00 PM).
- Reads all your Microsoft To-Do lists and tasks.
- Filters tasks that are overdue and not completed.
- Sends a single email summary with those outstanding items.

## Prerequisites
- A Microsoft 365 account with access to:
  - Microsoft To-Do
  - Power Automate
  - Outlook (for email sending)
- Permission to use a **custom connector** in Power Automate (if your tenant allows it).

## Build steps

### 1) Register an app in Azure
1. Go to **Microsoft Entra admin center** → **App registrations** → **New registration**.
2. Name it `TodoOverdueDigest`.
3. Create a client secret and copy it.
4. Add Microsoft Graph **Application** permissions:
   - `Tasks.Read`
   - `User.Read.All` (if needed in your tenant)
5. Grant admin consent.

> If you prefer delegated permissions, use delegated auth and run the flow as your account. Application permissions are usually easier for unattended daily runs.

### 2) Create a Power Automate cloud flow
1. In Power Automate, create a **Scheduled cloud flow**.
2. Set frequency to **1 day** at your preferred end-of-day time.

### 3) Add HTTP calls to Graph
Use an action that can call Graph (HTTP with Microsoft Entra ID, or a custom connector).

1. **Get To-Do lists**
   - `GET https://graph.microsoft.com/v1.0/me/todo/lists`
2. **Loop through each list** and get tasks
   - `GET https://graph.microsoft.com/v1.0/me/todo/lists/{listId}/tasks`
3. For each task, keep only those where:
   - `status != completed`
   - `dueDateTime` exists
   - `dueDateTime/dateTime < utcNow()`

### 4) Build the email body
Inside the flow, create an array of lines like:
- `- [List Name] Task Title (Due: 2026-05-09)`

If array is empty, send:
- `No overdue tasks today 🎉`

Otherwise send:
- Subject: `Daily Overdue To-Do Items`
- Body: newline-joined list.

### 5) Send the email
Use **Office 365 Outlook – Send an email (V2)**:
- To: your address
- Subject: `Daily Overdue To-Do Items - @{formatDateTime(utcNow(),'yyyy-MM-dd')}`
- Body: generated summary

## Optional enhancements
- Exclude specific lists (for example, "Shopping").
- Include tasks due in next 24 hours as a second section.
- Post the same summary to Teams.
- Add a deep link to each task using `webUrl` (if available in your response payload).

## Troubleshooting
- **401/403 from Graph**: check app permissions and admin consent.
- **Empty results**: ensure tasks have due dates and are in the same mailbox/account as the flow identity.
- **Time zone issues**: normalize with `convertTimeZone()` before comparing end-of-day cutoffs.
