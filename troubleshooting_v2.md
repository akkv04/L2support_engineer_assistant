Flow run failed. Action 'Get_email_(V2)' failed: The specified object was not found in the store., The process failed to get the correct properties. clientRequestId: bbe5dfed-7a3e-494b-8a18-50ac370ff2a2 serviceRequestId: 7f342394-8e2c-4044-a3a8-6b27c581b475


<img width="883" height="805" alt="image" src="https://github.com/user-attachments/assets/257d0ddf-4b32-4e70-b3e2-d22ebb9bedd8" />

replace(replace(replace(triggerOutputs()?['body/subject'], 'RE: ', ''), 'FW: ', ''), 'FWD: ', '')
Then in Get emails V3, change the Search Query to use this cleaned subject:
subject:"[Outputs from Compose_CleanSubject]"


<!DOCTYPE html>
<html>
<head>
<style>
  body { font-family: Calibri, Arial, sans-serif; margin: 40px; color: #333; }
  h1 { color: #1a5276; border-bottom: 2px solid #1a5276; padding-bottom: 8px; }
  h2 { color: #2874a6; margin-top: 30px; }
  table { border-collapse: collapse; width: 100%; margin-bottom: 20px; }
  td { padding: 8px 12px; border: 1px solid #ddd; }
  td:first-child { font-weight: bold; width: 200px; background-color: #f2f3f4; }
  .footer { margin-top: 40px; font-size: 12px; color: #888; }
</style>
</head>
<body>

<h1>Root Cause Analysis Document</h1>

<h2>Incident Summary</h2>
<table>
  <tr><td>Incident #</td><td>[varIncidentNumber]</td></tr>
  <tr><td>Subject</td><td>[varSubjectSummary]</td></tr>
  <tr><td>Affected System</td><td>[varAffectedSystem]</td></tr>
  <tr><td>Affected Users</td><td>[varAffectedUsers]</td></tr>
  <tr><td>Severity</td><td>[varSeverity]</td></tr>
  <tr><td>Confidence</td><td>[varConfidence]</td></tr>
</table>

<h2>Root Cause</h2>
<p>[varProbableCause]</p>

<h2>Resolution Steps</h2>
<p>[varResolutionSteps]</p>

<h2>RCA Summary</h2>
<p>[varRCASummary]</p>

<h2>Reviewer Notes</h2>
<p>[varRejectionComment]</p>

<h2>Timeline</h2>
<table>
  <tr><td>Email Received</td><td>[Received Time from trigger]</td></tr>
  <tr><td>RCA Created</td><td>[utcNow() expression]</td></tr>
</table>

<div class="footer">
  Generated automatically by L2 Support — Email to RCA Flow
</div>

</body>
</html>




<img width="931" height="828" alt="image" src="https://github.com/user-attachments/assets/a7d62db3-86ee-4739-88ed-1bcf465fb339" />

Flow save failed with code 'InvalidTemplate' and message 'The template validation failed: 'The repetition action(s) 'For_each_1' referenced by 'inputs' in action 'Create_file1' are not defined in the template.'.'.


outputs('Start_and_wait_for_an_approval')?['body/responses'][0]?['comments']


The input parameter(s) of action 'Set_varRejectionComment' contain an invalid reference to 'Start and wait for an approval'. Correct to include a valid reference to 'Start and wait for an approval' for the input parameter(s) of action 'Set_varRejectionComment'.


<img width="1621" height="820" alt="image" src="https://github.com/user-attachments/assets/31b15d9f-83a7-4b96-8933-24a9a537692f" />



