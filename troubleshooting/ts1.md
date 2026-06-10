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

concat('Step ', item()?['step_number'], ': ', item()?['action'], ' (Source: ', item()?['source_title'], ')')


Flow run failed. Action 'Compose_Noise_Triage' failed: Unable to process template language expressions in action 'Compose_Noise_Triage' inputs at line '0' and column '0': 'The template language expression 'replace(replace(body('Execute_Noise_Detector_Agent')?['responses'][0], concat(uriComponentToString('%60%60%60'), 'json'), ''), uriComponentToString('%60%60%60'), '')' cannot be evaluated because array index '0' cannot be selected from empty array. Please see https://aka.ms/logicexpressions for usage details.'.

length(body('Execute_Noise_Detector_Agent')?['responses'])


{"is_noise":false,"confidence":"Low","primary_reason":"agent failed after 3 attempts - passing to RCA for safety","matched_patterns":["max retry fallback"]}
