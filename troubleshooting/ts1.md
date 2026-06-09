if(
  empty(coalesce(
    body('Execute_Retry_Agent')?['responses'][0],
    body('Execute_Retry_Agent')?['response.0'],
    null
  )),
  '{"classification":"Incident","subject_summary":"manual review needed","affected_system":"Unknown","affected_users":"Unknown","incident_number":"Unknown","bmc_helix_alert":false,"severity":"P2","probable_cause":"Retry agent returned no output - review manually","confidence":"Low","resolution_steps":["Manual review required"],"knowledge_sources_used":[],"similar_past_incidents":[],"gaps_flagged":["Retry agent returned empty response"],"rca_summary":"Retry agent failed - manual review required","corrective_actions":[]}',
  replace(replace(
    coalesce(
      body('Execute_Retry_Agent')?['responses'][0],
      body('Execute_Retry_Agent')?['response.0']
    ),
    concat(uriComponentToString('%60%60%60'),'json'),''),
    uriComponentToString('%60%60%60'),'')
)
