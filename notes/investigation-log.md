# BOTSv3 Investigation Log

## Case metadata
- Analyst: 
- Date started:
- Dataset: BOTSv3 (Frothly Brewing Co.)
- Platform: Splunk Enterprise
- Goal: Answer Q1..Qn with defensible SPL + evidence

## Log entries
### Entry 2
- Date/Time: 2026-01-05
- Objective: Verify integrity of BOTSv3 dataset prior to ingestion
- Actions taken: Calculated MD5 hash of botsv3_data.tar.gz using PowerShell and compared with the hash provided by the author earlier
- SPL used: N/A
- Findings: Observed MD5 matches expected value published by dataset provider
- Evidence captured (path): evidence/00_preflight/md5_verification.png
- Next step: Proceed to dataset ingestion and validation

### Entry 3
- Date/Time: 2026-01-06
- Objective: Verify required BOTSv3 app stack
- Actions taken: Reviewed installed Splunk apps and searched for required components
- Findings: The OSquery app was not present; dataset does not include OSquery telemetry; Sysmon may provide endpoint coverage instead
- Evidence captured (path): evidence/01_installation/
- Next step: Proceed to data validation and sourcetype verification

### Entry 4
- Date/Time: 2026-01-06
- Objective: Validate BOTSv3 dataset completeness and correct parsing
- Actions taken: Re-ingested dataset using recursive folder monitoring with automatic sourcetype detection; validated index, sourcetypes, and time coverage
- Findings: Multiple sourcetypes present; dataset spans full attack window; ingestion issues resolved
- Evidence captured (path): evidence/02_validation/
- Next step: Establish baseline
### Entry 5
- Date/Time: 2026-01-06
- Objective: Establish baseline understanding of dataset prior to investigation
- Actions taken: Profiled dataset scope, event volume over time, and top hosts/users/destinations within botsv3_fixed
- Findings: Dataset appears complete and diverse; baseline established for anomaly comparison
- Evidence captured (path): evidence/02_baseline/
- Next step: Begin question-driven incident analysis

### Entry Q1.1
- Date/Time: 2026-01-08
- Objective: Identify available cloud-based telemetry relevant to file storage and delivery
- Trigger:
  Initial review of dataset scope to determine whether cloud storage activity could be analysed as part of potential malware delivery pathways.
- Actions taken:
  Reviewed sourcetype distribution and Microsoft 365 Management Activity logs to identify available workloads.
- Findings:
  OneDrive workload events are present within the `ms:o365:management` sourcetype, indicating visibility into cloud file upload and synchronization activity.
- Next step:
  Review OneDrive operation types to identify file-related actions suitable for further analysis.

### Entry Q1.2
- Date/Time: 2026-01-08
- Objective: Identify OneDrive operations associated with file creation or upload
- Trigger:
  Confirmation that OneDrive telemetry is available prompted further scoping to identify file-related actions.
- Actions taken:
  Enumerated OneDrive operation values within Microsoft 365 Management Activity logs to identify upload or file modification events.
- Findings:
  OneDrive operations related to file upload and file handling are present and suitable for filtering malicious artefact delivery.
- Next step:
  Apply threat-driven filtering to identify high-risk file types associated with malware delivery.

### Entry Q1.3
- Date/Time: 2026-01-08
- Objective: Narrow OneDrive file activity to high-risk artefact types
- Trigger:
  Threat intelligence and prior incident patterns indicate abuse of Windows shortcut (`.lnk`) files for phishing and malware execution.
- Actions taken:
  Applied filename-based filtering to OneDrive upload and file operations to isolate `.lnk` file activity.
- Findings:
  `.lnk` file upload events were identified within OneDrive activity logs, indicating potential malicious staging or delivery.
- Next step:
  Extract client attribution details to support incident assessment and response decisions.


### Entry Q1.4
- Date/Time: 2026-01-08
- Objective: Attribute `.lnk` file upload activity to a specific client context
- Trigger:
  Detection of `.lnk` file upload activity warranted attribution to assess legitimacy and risk.
- Actions taken:
  Extracted user identity, client IP address, and user agent associated with the OneDrive upload event.
- SPL used:
  See `spl/q1_onedrive_useragent.spl`
- Findings:
  The upload event was attributed to a specific user account and client context, with the user agent providing insight into the originating software and access method.Multiple user agents were observed. A standard Windows Edge user agent and a Microsoft backend process were identified as benign. A third user agent referencing NaenaraBrowser (ko-KP locale) was identified as anomalous and assessed as the primary indicator of suspicious activity
- Evidence captured (path):
  evidence/03_q1/onedrive_upload_useragent.png
- Next step:
  Assess security impact and determine appropriate SOC containment and follow-on investigation actions.

### Entry Q1.5
- Date/Time: 2026-01-08
- Objective: Assess risk and define SOC response actions for observed OneDrive activity
- Trigger:
  Attribution of `.lnk` file upload raised concern for potential malicious staging or phishing activity.
- Analyst assessment:
  Upload of a Windows shortcut file to cloud storage represents a credible indicator of malicious intent due to the file type’s ability to trigger command execution.
- Recommended SOC actions:
  - Revoke active user sessions and refresh authentication tokens
  - Review Conditional Access controls (MFA, device compliance, sign-in risk)
  - Investigate subsequent sharing or download activity involving the uploaded file
  - Correlate with endpoint telemetry to determine execution or follow-on payload delivery
- Next step:
  Continue investigation with follow-on questions focusing on execution, lateral movement, or persistence.
