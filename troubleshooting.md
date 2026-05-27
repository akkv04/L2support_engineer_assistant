
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
