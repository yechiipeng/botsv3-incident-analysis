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
- Evidence captured (path): evidence/01_installation/osquery_not_found_for_download.png
- Next step: Proceed to data validation and sourcetype verification
