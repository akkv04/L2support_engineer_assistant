# Complete Guide: Copilot Studio Agent - Email to Jira Epic Automation

## Overview
This guide will help you create an autonomous agent in Copilot Studio that:
1. Monitors for emails from a specific person with a specific subject
2. Automatically creates a Jira Epic
3. Uses email content as the epic description
4. Pre-populates other epic fields with values you define

---

## Prerequisites Checklist
Before you start, ensure you have:

- [ ] **Copilot Studio access** - Go to copilot.cloud.microsoft.com (or copilot.microsoft.com)
- [ ] **Jira Cloud instance** - (Must be Jira Cloud, not on-premise)
- [ ] **Jira API Token** - Generate this from your Jira profile settings
- [ ] **Outlook/Microsoft 365 account** - For email monitoring
- [ ] **Power Automate Premium license** - Your tenant admin should have this
- [ ] **Jira Project Key** - (Example: SUPPORT, BUGS, etc.)
- [ ] **Jira Epic Issue Type ID** - You'll need to find this (usually "10001" or similar)
- [ ] **List of Jira field IDs** - For custom fields you want to populate

---

## PART 1: Preparation Phase (Do This First!)

### Step 1a: Get Your Jira API Token

1. Log into your **Jira Cloud instance**
2. Go to: Profile Icon (top-right) → Settings
3. Click on **Security** (left sidebar)
4. Click **Create and manage API tokens**
5. Click **Create API token**
6. **Name it**: "Copilot Studio Automation"
7. **Copy the token** and save it somewhere safe (you'll need this later)

**Important**: The token will only be shown once. Save it immediately!

---

### Step 1b: Identify Your Jira Epic Issue Type ID

1. Go to your **Jira Cloud** instance
2. Navigate to: **Project Settings** → **Issue Types** (or **Manage Issues**)
3. Look for "**Epic**" in the list
4. Click on Epic and find the **ID** (usually looks like `10001` or `10050`)
5. **Write down this ID** - you'll need it later

**If you can't find it in the UI**, use this workaround:
- Go to: `https://yourjira.atlassian.net/rest/api/3/issuetype`
- Look for the issue type with `"name": "Epic"`
- Copy its `"id"` value

---

### Step 1c: Gather Jira Field IDs (Important!)

When creating Jira issues/epics via API, you need field IDs. Common ones:

| Field Name | Standard ID | Notes |
|---|---|---|
| Summary | `summary` | Required - Epic name |
| Description | `description` | Epic description |
| Project | `project` | Required - Your project key |
| Issue Type | `issuetype` | Required - Epic type ID |
| Priority | `priority` | (Optional) Must use ID like 1, 2, 3... |
| Labels | `labels` | (Optional) Array of labels |
| Components | `components` | (Optional) Array of component IDs |
| Custom Fields | `customfield_XXXXX` | Custom fields - ask your admin |

**To find custom field IDs**:
1. Go to **Project Settings** → **Custom fields**
2. Hover over the field name
3. The URL will show something like `customfield_10050`
4. Write down the ID

---

### Step 1d: Find the Email Sender's Email Address

1. Know the **exact email address** of the person who will be sending you emails
2. Example: `manager@company.com`
3. The email **subject line** you want to trigger on
4. Example: `"Create Epic: "`

---

## PART 2: Create the Copilot Studio Agent

### Step 2a: Create a New Agent

1. Go to **copilot.cloud.microsoft.com**
2. Click **+ New agent** (top-left)
3. Enter **Agent Name**: `Email to Jira Epic Automation` (or your preferred name)
4. Choose **Create a custom agent** 
5. Click **Create**
6. Wait for the agent to load

---

### Step 2b: Enable Generative Orchestration (Required for Email Triggers)

1. In your newly created agent, go to **Agent Overview** (top-left)
2. Scroll down to **Generative Orchestration**
3. Toggle it **ON** (Enable it)
4. Click **Save**

**What is this?** This allows your agent to respond to automatic triggers (like emails) rather than just user messages.

---

## PART 3: Create the Email Trigger Flow

### Step 3a: Navigate to Triggers

1. In your Copilot Studio agent, go to **Agent Overview**
2. Look for the **Triggers** section
3. Click **+ Add Trigger** or **+ Create new trigger**

---

### Step 3b: Create Power Automate Flow for Email Monitoring

1. Click **+ Add Trigger**
2. Select **Cloud flows** → **Automated cloud flow**
3. In Power Automate, select: **When a new email arrives** (Outlook trigger)
4. Click **Create**

You're now in Power Automate. Keep going!

---

### Step 3c: Configure the Email Trigger

In the **"When a new email arrives"** action, configure:

1. **Folder**: Select **Inbox** (or the folder where you receive these emails)
2. **From**: Type the email address from Step 1d
   - Example: `manager@company.com`
3. **Subject Filter**: Type part of the subject line
   - Example: `Create Epic:` (leave off the colon if you want to match variations)
4. **Include Attachments**: Toggle **On** (if you expect attachments)

Click **Save** once configured.

---

### Step 3d: Add a Parse Email Body Action

1. Click **+ New step**
2. Search for **Parse email body**
3. Select **Parse email body** (from the HTML to text connector)
4. In the **Content** field, click the dynamic content button
5. Select **Body** from "When a new email arrives"
6. Click **Save**

**Why?** This extracts the text content from the email for use in later steps.

---

## PART 4: Create the Jira Connection

### Step 4a: Add Jira Connector Connection

1. In Power Automate flow, click **+ New step**
2. Search for **Jira**
3. Select **Jira Cloud** connector
4. Click **Create**

---

### Step 4b: Authenticate with Jira

1. A popup will ask you to **sign in**
2. Enter your **Jira email address** and password
3. **Close** the popup once authenticated

---

## PART 5: Create the Jira Epic

### Step 5a: Add "Create a new issue" Action

1. In Power Automate, click **+ New step**
2. Search for and select **Jira** → **Create a new issue (V3)**

---

### Step 5b: Fill in Required Fields

Now you'll see fields in the "Create a new issue (V3)" action. Fill in:

**REQUIRED FIELDS:**

1. **Site**: Select your Jira Cloud instance from dropdown
2. **Project key**: Select your project (e.g., SUPPORT, BUGS)
3. **Issue Type**: You need to type this as JSON. Keep reading!

**For Issue Type (Epic)**, you have two options:

**Option A - Simple Version:**
- Click in the **Issue Type** field
- Click **Switch to input entire array**
- Paste this JSON structure:
```json
{
  "id": "YOUR_EPIC_ID_HERE"
}
```
Replace `YOUR_EPIC_ID_HERE` with the ID from Step 1b (usually 10001 or 10050)

Example:
```json
{
  "id": "10001"
}
```

---

### Step 5c: Add Summary (Epic Name)

1. Find the **Summary** field
2. Click the dynamic content button
3. Select **Subject** from "When a new email arrives"
4. This will use the email subject as your epic name

Example: If email subject is "Create Epic: New Feature X", the epic summary will be "Create Epic: New Feature X"

---

### Step 5d: Add Description (Email Content)

1. Find the **Description** field
2. Click the dynamic content button
3. Select **Content** from "Parse email body" step
4. This will use the email body as your epic description

---

### Step 5e: Add Optional Fields (Priority, Labels, etc.)

**To add more fields:**

1. Look for the **Item** field (this is the advanced JSON field)
2. Click **Show advanced options** or **Switch to input entire array**
3. Build a JSON payload like this:

```json
{
  "fields": {
    "summary": "Email subject here",
    "description": "Email body content here",
    "issuetype": {
      "id": "10001"
    },
    "project": {
      "key": "YOUR_PROJECT_KEY"
    },
    "priority": {
      "id": "2"
    },
    "labels": ["automated", "email-triggered"],
    "components": [
      {
        "id": "10050"
      }
    ]
  }
}
```

**Priority ID Reference** (Adjust based on your Jira):
- 1 = Highest
- 2 = High
- 3 = Medium
- 4 = Low
- 5 = Lowest

**What to replace:**
- `YOUR_PROJECT_KEY`: Your actual project key
- `10001`: Your epic issue type ID
- Component IDs: From your Jira settings
- Field values: According to your setup

---

### Step 5f: Save Your Power Automate Flow

1. Give your flow a name: `Trigger Copilot - Email to Jira Epic`
2. Click **Save** (top-right)

---

## PART 6: Connect Flow to Copilot Studio Agent

### Step 6a: Go Back to Copilot Studio

1. Return to your **Copilot Studio agent** tab
2. Go to **Agent Overview** → **Triggers**
3. You should see your Power Automate flow listed

---

### Step 6b: Select and Enable the Trigger

1. Find your flow: `Trigger Copilot - Email to Jira Epic`
2. Click on it to select it
3. Toggle it **ON** (Activate the trigger)
4. Click **Save**

---

## PART 7: Create the Topic (How Agent Responds)

### Step 7a: Create a New Topic

1. In Copilot Studio, go to **Topics** (left sidebar)
2. Click **+ New topic**
3. Name it: `Handle Email to Jira Epic Creation`
4. Under **Trigger phrases**, add:
   - "Create Jira epic from email"
   - "Process email to epic"

---

### Step 7b: Add Confirmation Message

1. In the topic editor, add a **Send a message** action
2. Type: `✅ Epic created successfully from your email: @{Email Subject}`
3. Replace `Email Subject` with the dynamic content from your email

---

### Step 7c: Save the Topic

1. Click **Save topic** (top-right)

---

## PART 8: Publish Your Agent

### Step 8a: Publish the Agent

1. Go back to **Agent Overview**
2. Click **Publish** (top-right)
3. Select **Publish to web** or **Publish to Teams**
4. Follow the prompts

---

### Step 8b: Test Your Automation

1. Send yourself a test email from the configured sender
2. Use the subject line: `Create Epic: Test Subject`
3. Add your issue description in the email body
4. Wait 2-5 minutes for the trigger to fire
5. Check your Jira project for the newly created epic

---

## TROUBLESHOOTING COMMON ISSUES

### Issue: "Email trigger not firing"
**Solution:**
- Check that Generative Orchestration is **enabled** in Agent Overview
- Verify the email is from the exact address you specified
- Check your Outlook junk/spam folder (email might be filtered)
- Wait 5+ minutes (triggers have a delay)

### Issue: "Jira returns error: Issue fields are empty"
**Solution:**
- Make sure you have **all required fields** in your JSON
- Verify your **Project key** is correct
- Check that your **Epic Issue Type ID** is correct
- Ensure your **Jira API token** is valid (not expired)

### Issue: "Can't find Epic issue type in Jira dropdown"
**Solution:**
- Use the **manual JSON approach** instead
- Copy the epic ID directly (from Step 1b)
- Paste it in the JSON format: `{"id": "10001"}`

### Issue: "Custom fields not showing up"
**Solution:**
- Custom fields must use the format: `customfield_XXXXX`
- Get the exact field ID from your Jira admin
- Add them to your JSON payload manually

---

## ADVANCED TIPS

### Add Email Attachments to Epic
1. In your Power Automate flow, add an **Attachments** section
2. Use the `Attachments` from "When a new email arrives"
3. In Jira description, mention attachment details

### Link to Specific Epic (Parent)
Add this to your JSON:
```json
"customfield_10000": {
  "id": "10001"
}
```
(Replace with your actual epic link field ID)

### Add Watchers
Add this to your JSON:
```json
"watchers": [
  {
    "name": "user@company.com"
  }
]
```

### Conditional Creation (Only create if subject contains certain words)
Add a **Condition** step in Power Automate:
1. Click **+ New step** → **Condition**
2. Set condition: `@contains(triggerBody()?['subject'], 'Create Epic')`
3. Only if true, run the Jira creation action

---

## QUICK REFERENCE CHECKLIST

Before testing, verify:
- [ ] Generative Orchestration is **ON**
- [ ] Email trigger is configured with correct email address
- [ ] Email subject filter is set correctly
- [ ] Jira API token is valid
- [ ] Project key is correct
- [ ] Epic Issue Type ID is correct
- [ ] Summary and Description fields are mapped
- [ ] All required fields are included in JSON
- [ ] Power Automate flow is **activated**
- [ ] Copilot Studio agent is **published**
- [ ] Trigger is enabled in Agent Overview

---

## Next Steps

Once this is working:
1. **Add more fields** - Priority, assignee, components, labels
2. **Create multiple triggers** - For different email subjects
3. **Add approval workflow** - Require approval before epic creation
4. **Notify team** - Send Teams/Slack message when epic is created
5. **Track metrics** - Log creation details for reporting

---

## Support Resources

- **Copilot Studio Docs**: https://learn.microsoft.com/copilot-studio
- **Power Automate Docs**: https://learn.microsoft.com/power-automate
- **Jira API Docs**: https://developer.atlassian.com/cloud/jira/platform/rest/v3
- **Community Help**: power.microsoft.com/community

