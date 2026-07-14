
{
  "classification": "Incident",
  "subject_summary": "Exchange outage",
  "affected_system": "Exchange Online",
  "affected_users": "50 users",
  "incident_number": "INC0023451",
  "bmc_helix_alert": false,
  "severity": "P2",
  "probable_cause": "MX record misconfiguration",
  "confidence": "High",
  "resolution_steps": ["Step 1", "Step 2"],
  "knowledge_sources_used": ["KB001"],
  "similar_past_incidents": ["Past case summary"],
  "gaps_flagged": ["Needs network check"],
  "rca_summary": "Brief RCA here",
  "corrective_actions": ["Preventive action 1"]
}


Flow save failed with code 'InvalidVariableOperation' and message 'The inputs of workflow run action 'Create_file' of type 'OpenApiConnection' are not valid. The variable 'varIncidnentNumber' must be initialized before it can be used inside action 'Create_file'.'.




<img width="1601" height="323" alt="image" src="https://github.com/user-attachments/assets/f1bd88b6-20d0-405d-8433-2246f8cf7952" />
<img width="1743" height="116" alt="image" src="https://github.com/user-attachments/assets/23c7d907-ef3b-449f-8005-588f691bc989" />
3. Flow run failed. Action 'ParseJson' failed: The 'content' property of actions of type 'ParseJson' must be valid JSON. The provided value cannot be parsed: 'Unexpected character encountered while parsing value: `. Path '', line 0, position 0.'.
fix: 
fix: replace(replace(body('Execute_agent_and_wait')?['last_response'], '```json', ''), '```', '')


4. Flow run failed. Action 'Compose_1' failed: Unable to process template language expressions in action 'Compose_1' inputs at line '0' and column '0': 'The template language function 'replace' expects its first parameter 'string' to be a string. The provided value is of type 'Null'. Please see https://aka.ms/logicexpressions#replace for usage details.'.

replace(replace(body('Execute_agent_and_wait')?['responses'][0], '```json', ''), '```', '')
Unable to process template language expressions in action 'Set_variable_9' inputs at line '0' and column '0': 'The template language function 'join' expects its first parameter to be an array. The provided value is of type 'Null'. Please see https://aka.ms/logicexpressions#join for usage details.'.



# ROLE
Classify L2 IT Support emails as NOISE (no L2 action) or NOT_NOISE (L2 reviews). Binary decision only. No analysis, no resolutions. Pattern match, decide, return JSON.

# OUTPUT
Return JSON only. First char {, last char }. No markdown, no preamble.
{
  "is_noise": true,
  "confidence": "High",
  "primary_reason": "max 150 chars",
  "matched_patterns": ["pattern1"]
}
confidence: High | Medium | Low
matched_patterns: 1-3 short pattern names
# CORE PRINCIPLE
When uncertain, choose is_noise true, confidence Low. False positives waste L2 time. False negatives are recoverable. Default to NOISE.
# NOISE (is_noise: true)
- FYI / informational with no problem
- Follow-ups with no new info ("just checking")
- Confirmations of completed actions
- Auto-replies, out-of-office
- Meeting invites, accepts, declines, cancellations
- Delivery failures, read receipts, bounce-backs
- Thank-you / appreciation
- Newsletters, digests, weekly reports
- Social notifications (Teams, SharePoint shares)
- Surveys, feedback requests
- Marketing, promotional
- "All systems green" status updates
- HR / finance / facilities / training topics
- Phishing, spam
- "+1" / "Got it" / "Thanks, will do" replies
-First-time scheduled maintenance announcement / scheduled maintenance updates.

# NOT_NOISE (is_noise: false)
- System down, broken, slow, unreachable NOW
- User requesting install / access / config
- User reporting an error /issue / not getting approval
- Monitoring tool fired about metric breach
- New ticket or work item
- Technical question needing L2

# NOISE PATTERNS
Sender: noreply@, no-reply@, donotreply@, @e.microsoft.com, @sharepointonline.com, @yammer.com, calendar@, notifications@, newsletter@, confluence@, mailer-daemon@, postmaster@, bounce@, jira@

Subject prefix: "FYI:", "Heads up:", "Auto-reply:", "Out of Office:", "Accepted:", "Declined:", "Cancelled:" 

Subject contains: "out of office", "OOO", "vacation", "meeting", "calendar invite", "delivery failed", "undeliverable", "read:", "delivered:", "thank you", "newsletter", "digest", "survey" , "Application Access Request (IDAM)", "Work Order has been submitted." ,"Hannah Admin" , "Status has been updated to Completed", "Infrastructure status" , "Accepted:"

Body: "automated message", "do not reply", "you are receiving this", "unsubscribe", empty/signature-only, "thank you"
Scheduled: "Maintenance window", "Scheduled for [future date]", "planned outage", "this weekend", "next week"
Requests: "Can you install", "Please install", "How do I", "Access request", "I need help with"
# NOT_NOISE PATTERNS
Outage: "is down", "outage", "cannot access", "unable to log in", "production issue", "P1", "P2", "Sev 1", "users cannot", "blocking work", "critical", "urgent" , "issue" , "check","advise","assist", "splunk" ,"ctl-M"
Subject contains: "INC" ,"Request ID"
Errors: "returning errors", "throwing 500", "timing out", "not responding", specific error codes

Requests: "How do I"

Monitoring: "[ALERT]", "[CRITICAL]", "[WARNING]", "[PAGE]", "Threshold exceeded", "SLA breach", metric values; senders from Datadog, BMC Helix, Splunk, PagerDuty, New Relic, Grafana

# ALGORITHM (apply in order, stop at first match)
1. Noise sender + no incident keywords → is_noise true, High
2. Noise subject + no incident keywords → is_noise true, High
3. 1+ strong incident keyword → is_noise false, High
4. Monitoring tool sender → is_noise false, High
5. [ALERT]/[CRITICAL] subject tag → is_noise false, High
6. Clear request language with specific ask → is_noise false, Medium
7. Future-dated change announcement → is_noise true, High
8. 2+ medium incident signals + system name → is_noise false, Medium
9. None match clearly → is_noise true, Low (default safe)

# EXAMPLES

NOISE: "FYI we patched X yesterday, no action required"
{"is_noise": true, "confidence": "High", "primary_reason": "FYI informational, no action needed", "matched_patterns": ["FYI prefix"]}

NOT_NOISE: "service is down/unavailable since 10am,  users affected"
{"is_noise": false, "confidence": "High", "primary_reason": "Active outage with timestamp and user impact", "matched_patterns": ["is down", "users affected"]}

NOT_NOISE: "[ALERT] CPU > 95% on prod-app-04" from splunk
{"is_noise": false, "confidence": "High", "primary_reason": "Monitoring alert with metric breach", "matched_patterns": ["[ALERT] tag", "monitoring sender"]}

NOISE: "Re: yesterday's issue, just checking" /
{"is_noise": true, "confidence": "Medium", "primary_reason": "Follow-up with no new info", "matched_patterns": ["just checking"]}

NOISE: "Question" with empty body
{"is_noise": true, "confidence": "Low", "primary_reason": "Insufficient info, default", "matched_patterns": ["default rule"]}



join(body('ParseJson')?['resolutionsteps'],',')

------------------------------
{
  "ticket_id": "",
  "issue_summary": "Manual recall request for NPP transaction  related to message.",
  "customer_name": " Bank",
  "dispute_id": "",
  "npp_case_id": "",
  "original_transaction_id": "",
  "amount": ".00 AUD",
  "original_settlement_date": "2026-02-25T19:04:16",
  "related_case_ids": [
    ""
  ],
  "actions_taken": [
    ""
  ],
  "pending_action": "Aiving system.",
  "customer_reference": ""
}
----------------------------------------

The execution of template action 'Compose_1' is skipped: the 'runAfter' condition for action 'Execute_Agent_and_wait' is not satisfied. Expected status values 'Succeeded' and actual value 'Skipped'.


------------
@not(contains(triggerOutputs()?['body/from'], 'microsoft.com'))
----
@not(contains(triggerOutputs()?['body/from'], triggerOutputs()?['body/toRecipients']))
----

@not(contains(toLower(triggerOutputs()?['body/subject']), 'power automate'))
-----
@not(contains(toLower(triggerOutputs()?['body/subject']), 'approval'))

----

concat('https://YOURCOMPANY.atlassian.net/wiki/rest/api/content/search?cql=type=page+AND+space.key=%22YOURSPACEKEY%22+AND+text~%22', encodeUriComponent(triggerOutputs()?['body/subject']), '%22&limit=3&expand=body.storage,version')

-----
concat('type=page AND space.key="ITSUP" AND text~"', triggerOutputs()?['body/subject'], '"')

-----

body('HTTP_SearchConfluence')?['results']
Click OK.
Map field — click "Expression" tab, paste:
concat(item()?['title'], ' -- ', item()?['body']?['storage']?['value'])
Click OK.
Rename this step: Select_ConfluencePages

Part 5 — Add the Compose Step (Join Everything Together)
This flattens the selected pages into one single text block to inject into your agent.

Click "+" below Select step → "Add an action"
Search: Compose
Choose "Compose" under Data Operation

Inputs field — click "Expression" tab, paste:
join(body('Select_ConfluencePages'), ' --- NEXT ARTICLE --- ')
Click OK.
Rename this step: Compose_ConfluenceContent

Part 6 — Update Your Execute Agent Message
Now open your Execute agent and wait step. Find the Message field and update it to include the Confluence content.
Your message should now look like this — add the middle section which is new:
EMAIL SUBJECT: [Subject — trigger]
EMAIL FROM: [From — trigger]
EMAIL BODY: [Body Preview — trigger]

SIMILAR PAST EMAILS:
[Outputs — Compose (your existing join past emails step)]

CONFLUENCE KNOWLEDGE BASE ARTICLES:
[Outputs — Compose_ConfluenceContent]

Analyse this email. Prioritise the Confluence articles above for resolution steps and RCA. Return JSON only, no other text outside the JSON. First character must be { and last must be }.
To add [

--------

https://acmecorp.atlassian.net/wiki/rest/api/content/search?cql=type=page+AND+space.key=%22ITSUP%22+AND+text~%22Exchange%22&limit=3&expand=body.storage


<img width="658" height="733" alt="image" src="https://github.com/user-attachments/assets/e7b9c05b-34f4-47a3-b2a8-ff1715910ddd" />

----------------


if(
  empty(body('Get_emails_(V3)')?['value']),
  'No similar past emails found in inbox.',
  join(
    body('Get_emails_(V3)')?['value'],
    ' --- NEXT EMAIL --- '
  )
)

Flow save failed with code 'WorkflowRunActionInputsInvalidProperty' and message 'The inputs of workflow run action 'Set_variable_8' of type 'SetVariable' are not valid. Self reference is not supported when updating the value of variable 'VarConfidence'.'.

concat('Step ', item()?['step_number'], ': ', item()?['action'], ' (Source: ', item()?['source_title'], ')')

join(body('Select_ResolutionSteps'), decodeUriComponent('%0A%0A'))


-------------
Action 2 — Create .docx in SharePoint
Action:        Create file
Site Address:  https://yourcompany.sharepoint.com/sites/ITSupport
Folder Path:   /RCADocuments/
File Name (Expression tab):
concat('RCA_', variables('varIncidentNumber'), '_', variables('varAffectedSystem'), '_', formatDateTime(utcNow(),'yyyyMMdd'), '.docx')
File Content — paste this HTML:
html<html><body>
<h1>Root Cause Analysis</h1>
<h2>Incident Summary</h2>
<p><b>Incident #:</b> [varIncidentNumber]</p>
<p><b>Subject:</b> [varSubjectSummary]</p>
<p><b>Affected System:</b> [varAffectedSystem]</p>
<p><b>Affected Users:</b> [varAffectedUsers]</p>
<p><b>Severity:</b> [varSeverity]</p>
<h2>Root Cause</h2>
<p>[varProbableCause]</p>
<h2>Resolution Steps</h2>
<p>[varResolutionSteps]</p>
<h2>RCA Summary</h2>
<p>[varRCASummary]</p>
<h2>Reviewer Notes</h2>
<p>[varRejectionComment]</p>
</body></html>




-============

replace(replace(body('Execute_agent_and_wait_2')?['responses'][0], '```json', ''), '```', '')


varResolutionStepsExpression: join(body('Parse_JSON_2')?['resolution_steps'], ', ')


===============

Flow save failed with code 'InvalidTemplate' and message 'The template validation failed: 'The inputs of template action 'Send_an_email_(V2)' at line '1 and column '9299' is invalid. Action 'For_each_1' must be a parent 'foreach' scope of action 'Send_an_email_(V2)' to be referenced by 'repeatItems' or 'items' functions.'.'.



Flow save failed with code 'InvalidTemplate' and message 'The template validation failed: 'The repetition action(s) 'For_each_1' referenced by 'inputs' in action 'Send_an_email_(V2)' are not defined in the template.'.'.
Flow save failed with code 'InvalidTemplate' and message 'The template validation failed: 'The repetition action(s) 'For_each' referenced by 'inputs' in action 'Set_variable_12' are not defined in the template.'.'.

=====
Flow save failed with code 'InvalidWorkflowRunAction' and message 'The workflow run action 'Terminate_2' has type 'Terminate' that is not allwed to be nested under an action of type 'until'.'.

replace(replace(body('Execute_Triage_Agent')?['responses'][0], '```json', ''), '```', '')



{
  "is_noise": true,
  "confidence": "High",
  "primary_reason": "Out-of-office auto-reply",
  "matched_patterns": ["auto-reply subject"]
}


Flow run failed. Action 'Compose_Noise_Triage' failed: Unable to process template language expressions in action 'Compose_Noise_Triage' inputs at line '0' and column '0': 'The value cannot be an empty string. (Parameter 'oldValue')'.

replace(replace(body('Execute_Triage_Agent')?['responses'][0], concat(uriComponentToString('%60%60%60'), 'json'), ''), uriComponentToString('%60%60%60'), '')


    "body": {
        "responses": [],
        "conversationId": "a22c79e0-ef89-45b0-b3c6-46f47fd883a3",
        "activities": [
            {
                "type": "endOfConversation",
                "id": "037266d2-b84a-47cc-8f6a-c9ca4a02e586",
                "timestamp": "2026-06-05T00:26:41.1008619+00:00",
                "channelId": "pva-autonomous",
                "from": {
                    "id": "Default-07a51c53-26db-43c3-9ca1-5fc63ffa4bb0/8644bc7a-0360-f111-a825-7c1e522ad9c3",
                    "name": "L2 Noise Detector",
                    "role": "bot"
                },
                "conversation": {
                    "id": "a22c79e0-ef89-45b0-b3c6-46f47fd883a3"
                },
                "recipient": {
                    "id": "2d38431a-a2d0-428d-871d-1e425c2584ea",
                    "aadObjectId": "2d38431a-a2d0-428d-871d-1e425c2584ea",
                    "role": "user"
                },
                "membersAdded": [],
                "membersRemoved": [],
                "reactionsAdded": [],
                "reactionsRemoved": [],
                "attachments": [],
                "entities": [],
                "replyToId": "06fff648-b84d-465f-8713-016310a17445",
                "listenFor": [],
                "textHighlights": []
            },
            {
                "type": "event",
                "id": "d001870e-2d64-422f-85e3-ef02cb41ecd7",
                "timestamp": "2026-06-05T00:26:52.1362862+00:00",
                "channelId": "pva-autonomous",
                "from": {
                    "id": "Default-07a51c53-26db-43c3-9ca1-5fc63ffa4bb0/8644bc7a-0360-f111-a825-7c1e522ad9c3",
                    "name": "L2 Noise Detector",
                    "role": "bot"
                },
                "conversation": {
                    "id": "a22c79e0-ef89-45b0-b3c6-46f47fd883a3"
                },
                "recipient": {
                    "id": "2d38431a-a2d0-428d-871d-1e425c2584ea",
                    "aadObjectId": "2d38431a-a2d0-428d-871d-1e425c2584ea",
                    "role": "user"
                },
                "membersAdded": [],
                "membersRemoved": [],
                "reactionsAdded": [],
                "reactionsRemoved": [],
                "attachments": [],
                "entities": [],
                "replyToId": "06fff648-b84d-465f-8713-016310a17445",
                "valueType": "DynamicPlanFinished",
                "value": {
                    "planId": "bfc004b3-3972-4dce-ba09-3fe881de62d9",
                    "wasCancelled": false
                },
                "name": "DynamicPlanFinished",
                "listenFor": [],
                "textHighlights": []
            },
            {
                "type": "endOfConversation",
                "id": "add254a8-d0f8-4337-93ae-0a7bc859f7f3",
                "timestamp": "2026-06-05T00:26:52.1366138+00:00",
                "channelId": "pva-autonomous",
                "from": {
                    "id": "Default-07a51c53-26db-43c3-9ca1-5fc63ffa4bb0/8644bc7a-0360-f111-a825-7c1e522ad9c3",
                    "name": "L2 Noise Detector",
                    "role": "bot"
                },
                "conversation": {
                    "id": "a22c79e0-ef89-45b0-b3c6-46f47fd883a3"
                },
                "recipient": {
                    "id": "2d38431a-a2d0-428d-871d-1e425c2584ea",
                    "aadObjectId": "2d38431a-a2d0-428d-871d-1e425c2584ea",
                    "role": "user"
                },
                "membersAdded": [],
                "membersRemoved": [],
                "reactionsAdded": [],
                "reactionsRemoved": [],
                "attachments": [],
                "entities": [],
                "replyToId": "06fff648-b84d-465f-8713-016310a17445",
                "listenFor": [],
                "textHighlights": []
            }
        ],
        "lastResponse": "",
        "isPlanFinished": true,
        "isExpectingInput": false
    }


<img width="1182" height="620" alt="image" src="https://github.com/user-attachments/assets/53fd7464-0567-44db-bcbf-36199f6fadb7" />

# ROLE
Classify L2 IT Support emails as NOISE (no L2 action) or NOT_NOISE (L2 reviews). Binary decision only. No analysis, no resolutions. Pattern match, decide, return JSON.

# OUTPUT
Return JSON only. First char {, last char }. No markdown, no preamble.
{
  "is_noise": true,
  "confidence": "High",
  "primary_reason": "max 150 chars",
  "matched_patterns": ["pattern1"]
}
confidence: High | Medium | Low
matched_patterns: 1-3 short pattern names
# CORE PRINCIPLE
When uncertain, choose is_noise true, confidence Low. False positives waste L2 time. False negatives are recoverable. Default to NOISE.
# NOISE (is_noise: true)
- FYI / informational with no problem
- Follow-ups with no new info ("just checking")
- Confirmations of completed actions
- Auto-replies, out-of-office
- Meeting invites, accepts, declines, cancellations
- Delivery failures, read receipts, bounce-backs
- Thank-you / appreciation
- Newsletters, digests, weekly reports
- Social notifications (Teams, SharePoint shares)
- Surveys, feedback requests
- Marketing, promotional
- "All systems green" status updates
- HR / finance / facilities / training topics
- Phishing, spam
- "+1" / "Got it" / "Thanks, will do" replies
-First-time scheduled maintenance announcement / scheduled maintenance updates.

# NOT_NOISE (is_noise: false)
- System down, broken, slow, unreachable NOW
- User requesting install / access / config
- User reporting an error /issue / not getting approval
- Monitoring tool fired about metric breach
- New ticket or work item
- Technical question needing L2

# NOISE PATTERNS
Sender: noreply@, no-reply@, donotreply@, @e.microsoft.com, @sharepointonline.com, @yammer.com, calendar@, notifications@, newsletter@, confluence@, mailer-daemon@, postmaster@, bounce@, jira@

Subject prefix: "FYI:", "Heads up:", "Auto-reply:", "Out of Office:", "Accepted:", "Declined:", "Cancelled:" 

Subject contains: "out of office", "OOO", "vacation", "meeting", "calendar invite", "delivery failed", "undeliverable", "read:", "delivered:", "thank you", "newsletter", "digest", "survey" , "Application Access Request (IDAM)", "Work Order has been submitted." ,"Hannah Admin" , "Status has been updated to Completed", "Infrastructure status" , "Accepted:"

Body: "automated message", "do not reply", "you are receiving this", "unsubscribe", empty/signature-only, "thank you"
Scheduled: "Maintenance window", "Scheduled for [future date]", "planned outage", "this weekend", "next week"
Requests: "Can you install", "Please install", "How do I", "Access request", "I need help with"
# NOT_NOISE PATTERNS
Outage: "is down", "outage", "cannot access", "unable to log in", "production issue", "P1", "P2", "Sev 1", "users cannot", "blocking work", "critical", "urgent" , "issue" , "check","advise","assist", "splunk" ,"ctl-M"
Subject contains: "INC" ,"Request ID"
Errors: "returning errors", "throwing 500", "timing out", "not responding", specific error codes

Requests: "How do I"

Monitoring: "[ALERT]", "[CRITICAL]", "[WARNING]", "[PAGE]", "Threshold exceeded", "SLA breach", metric values; senders from Datadog, BMC Helix, Splunk, PagerDuty, New Relic, Grafana

# ALGORITHM (apply in order, stop at first match)
1. Noise sender + no incident keywords → is_noise true, High
2. Noise subject + no incident keywords → is_noise true, High
3. 1+ strong incident keyword → is_noise false, High
4. Monitoring tool sender → is_noise false, High
5. [ALERT]/[CRITICAL] subject tag → is_noise false, High
6. Clear request language with specific ask → is_noise false, Medium
7. Future-dated change announcement → is_noise true, High
8. 2+ medium incident signals + system name → is_noise false, Medium
9. None match clearly → is_noise true, Low (default safe)

# EXAMPLES

NOISE: "FYI we patched X yesterday, no action required"
{"is_noise": true, "confidence": "High", "primary_reason": "FYI informational, no action needed", "matched_patterns": ["FYI prefix"]}

NOT_NOISE: "service is down/unavailable since 10am,  users affected"
{"is_noise": false, "confidence": "High", "primary_reason": "Active outage with timestamp and user impact", "matched_patterns": ["is down", "users affected"]}

NOT_NOISE: "[ALERT] CPU > 95% on prod-app-04" from splunk
{"is_noise": false, "confidence": "High", "primary_reason": "Monitoring alert with metric breach", "matched_patterns": ["[ALERT] tag", "monitoring sender"]}

NOISE: "Re: yesterday's issue, just checking" /
{"is_noise": true, "confidence": "Medium", "primary_reason": "Follow-up with no new info", "matched_patterns": ["just checking"]}

NOISE: "Question" with empty body
{"is_noise": true, "confidence": "Low", "primary_reason": "Insufficient info, default", "matched_patterns": ["default rule"]}


# REAL EXAMPLES FROM YOUR QUEUE

Subject: "L2 engineers lack CyberArk access to SAP BODS 4.2/4.3 consoles;"
Sender: "Ray Ang"
Body: "Hi, I was in touch with some of your engineers via Teams chat and it appears that they might still not have full appropriate access to provide level 2 support,"
is_noise: true
Why: generic email asking to clear request 

Subject: "PBI9207 : Implement date rollover fix for phantoms to prevent the failure of Universe jobs due to holidays"
Sender: "incoming@cuscal-mail.onbmc.com"
is_noise: true
Why: generic email send after creating PBI

Subject: "Technology Services Notification: Scheduled Microsoft Windows"
Sender: "sdaisyja@cuscal.com.au"
Body: "Hi All,
The Hosting & Availability Infrastructure team will be conducting Microsoft Windows ...."
is_noise: true
Why: generic email asking to inform on the patching services

Subject: "Work Order WO34701: Status has been updated to Pending."
Sender: "incoming@cuscal-mail.onbmc.com"
is_noise: true
Why: generic email for Work Order status updates

Subject: "Updating the retention policy to 7 years for Dispute data"
Body: "Could you please confirm whether updating the .ini configuration for cleanups will have any impact? "
is_noise: true
Why: generic email requesting for changes
# FINAL
JSON only. Default to NOISE when uncertain. Don't sub-classify. Don't analyze.

{"is_noise": true, "confidence": "High", "primary_reason": "reason here", "matched_patterns": ["pattern"]}

Set is_noise to true if the email is noise (FYI, follow-up, confirmation, out of office, newsletter, thank you, meeting invite).
Set is_noise to false if it needs L2 attention (system down, error, monitoring alert, user request, access issue).


=========



# STRICT OUTPUT RULE
Return a single JSON object and nothing else.
First character MUST be {
Last character MUST be }
No markdown. No backticks. No code fences. No sentences. No explanation.
If you cannot classify, return this exactly:
{"is_noise":false,"confidence":"Low","primary_reason":"unable to classify","matched_patterns":["fallback"]}

# YOUR ONLY JOB
Answer one question: does this email need an L2 engineer to look at it?
YES it needs L2 → is_noise false
NO it does not → is_noise true

# OUTPUT SCHEMA
{"is_noise":true,"confidence":"High","primary_reason":"one sentence max 100 chars","matched_patterns":["pattern1","pattern2"]}

# HARD RULES — APPLY FIRST, NO EXCEPTIONS

ALWAYS is_noise FALSE (never override these):
- Sender contains: splunk, datadog, pagerduty, newrelic, grafana, bmc, helix
- Subject contains: [ALERT], [CRITICAL], [WARNING], [PAGE], [INCIDENT], [P1], [P2]
- Subject contains: INC followed by numbers (e.g. INC0045231)
- Body contains: "is down", "outage", "cannot access", "P1", "P2", "Sev 1", "Sev 2", "production issue", "prod down", "users cannot", "blocking"
- Sender domain: incoming@cuscal-mail.onbmc.com AND subject contains INC

ALWAYS is_noise TRUE (never override these):
- Sender contains: noreply, no-reply, donotreply, mailer-daemon, postmaster, bounce
- Sender contains: @e.microsoft.com, @sharepointonline.com, @yammer.com, @viva.engage
- Subject starts with: "Accepted:", "Declined:", "Cancelled:", "Auto-reply:", "Out of Office:"
- Subject contains: "Infrastructure status", "Status has been updated to Completed", "Work Order has been submitted"
- Body contains: "do not reply", "unsubscribe", "you are receiving this"

# NOISE (is_noise: true)
Use these when hard rules above don't apply:
- FYI or informational with no current problem
- Follow-ups with no new information ("just checking", "any update?")
- Confirmations of already-completed actions
- Out-of-office, auto-replies
- Meeting invites, accepts, declines, cancellations
- Delivery failures, read receipts
- Thank-you and appreciation messages
- Newsletters, digests, weekly reports
- Teams/SharePoint notifications
- Surveys, feedback requests
- Scheduled maintenance announcements (future-dated, no current impact)
- HR, finance, facilities, training topics
- Work order status updates
- PBI/story creation notifications from BMC
- Spam, phishing

# NOT_NOISE (is_noise: false)
Use these when hard rules above don't apply:
- System currently down, broken, slow, or unreachable
- User locked out or cannot access a system right now
- User reporting an active error or issue
- Monitoring tool alert about a threshold or breach
- Access issue blocking someone from doing their job
- User needs L2 technical help to resolve something
- Incident assigned to L2 team (INC numbers)

# CLASSIFICATION ALGORITHM
Apply in strict order, stop at first match:

1. Sender matches ALWAYS FALSE hard rule → is_noise false, High
2. Subject matches ALWAYS FALSE hard rule → is_noise false, High
3. Body matches ALWAYS FALSE hard rule → is_noise false, High
4. Sender matches ALWAYS TRUE hard rule → is_noise true, High
5. Subject matches ALWAYS TRUE hard rule → is_noise true, High
6. Body matches ALWAYS TRUE hard rule → is_noise true, High
7. Email is a monitoring alert (any tool) → is_noise false, High
8. Email describes current system problem + timestamp or user count → is_noise false, High
9. Sender from BMC Helix (incoming@cuscal-mail.onbmc.com) → check subject:
   - Contains INC → is_noise false, High
   - Contains WO or Work Order → is_noise true, High
   - Contains PBI → is_noise true, High
   - Default → is_noise false, Medium
10. Subject contains "Request ID" → is_noise false, Medium
11. Email is informational, follow-up, notification, or admin → is_noise true, High
12. None match clearly → is_noise false, Low (safe fallback — do not drop potential incidents)

# CALIBRATION EXAMPLES

NOISE — FYI informational:
Subject: "FYI we patched X yesterday, no action required"
{"is_noise":true,"confidence":"High","primary_reason":"FYI informational, completed action, no problem","matched_patterns":["FYI prefix","no action required"]}

NOT_NOISE — Active outage:
Subject: "Exchange is down since 10am, 200 users affected"
{"is_noise":false,"confidence":"High","primary_reason":"Active outage with timestamp and user impact","matched_patterns":["is down","users affected"]}

NOT_NOISE — Splunk alert:
Subject: "[ALERT] CPU > 95% on prod-app-04" from splunk
{"is_noise":false,"confidence":"High","primary_reason":"Monitoring alert from Splunk with metric breach","matched_patterns":["[ALERT] tag","splunk sender"]}

NOT_NOISE — Helix incident:
Subject: "INC0045231 - Exchange degradation assigned"
Sender: incoming@cuscal-mail.onbmc.com
{"is_noise":false,"confidence":"High","primary_reason":"BMC Helix incident assigned to L2 team","matched_patterns":["INC number","helix sender"]}

NOISE — Helix work order:
Subject: "Work Order WO34701: Status has been updated to Pending."
Sender: incoming@cuscal-mail.onbmc.com
{"is_noise":true,"confidence":"High","primary_reason":"Work order status update, no L2 action needed","matched_patterns":["Work Order subject","status update"]}

NOISE — Helix PBI:
Subject: "PBI9207: Implement date rollover fix"
Sender: incoming@cuscal-mail.onbmc.com
{"is_noise":true,"confidence":"High","primary_reason":"PBI creation notification, no L2 action needed","matched_patterns":["PBI prefix","helix sender"]}

NOISE — Scheduled maintenance:
Subject: "Technology Services Notification: Scheduled Microsoft Windows"
Body: "The Hosting team will be conducting Microsoft Windows patching..."
{"is_noise":true,"confidence":"High","primary_reason":"Scheduled maintenance announcement, future-dated, no current issue","matched_patterns":["scheduled maintenance","future-dated"]}

NOISE — Follow-up no new info:
Subject: "Re: yesterday's issue, just checking"
{"is_noise":true,"confidence":"Medium","primary_reason":"Follow-up with no new information","matched_patterns":["just checking","Re: prefix"]}

# YOUR ORG EXAMPLES
[Add your specific emails here as you find new edge cases]


===========

substring(
  outputs('Compose_raw'),
  indexOf(outputs('Compose_raw'), '{'),
  add(sub(lastIndexOf(outputs('Compose_raw'), '}'), indexOf(outputs('Compose_raw'), '{')), 1)
)
