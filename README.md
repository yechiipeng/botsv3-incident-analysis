# botsv3-incident-analysis
## 0.2 Dataset Integrity Verification

Prior to ingestion into software and analysis of the data, the integrity of the BOTSv3 dataset was verified to ensure that the dataset used in this investigation matches the original distribution provided by the dataset maintainer.

**Verification details**

- Dataset file: `botsv3_data_set (1).tgz`
- Integrity check method: MD5 checksum
- Expected MD5 hash: `d7ccca99a01cff070dff3c139cdc10eb`
- Observed MD5 hash: `d7ccca99a01cff070dff3c139cdc10eb`
- Verification tool: PowerShell `Get-FileHash`
- Evidence: `evidence/00_preflight/md5_verification.png`

The matching checksum confirms that the dataset was not corrupted or altered prior to ingestion.

### 1.2 App stack verification

The required BOTSv3 app stack was verified to ensure correct field extraction and data normalization across endpoint, cloud, network, and email telemetry, consistent with a real SOC deployment.

The following applications were originally not present but were eventually installed:
- Splunk Add-on for Microsoft Sysmon
- Splunk Add-on for Microsoft Office 365
- Splunk Stream
- Splunk Common Information Model (CIM)

The Splunk App for OSquery was not present in the environment and was not required for analysis, as the BOTSv3 dataset does not include OSquery telemetry. Endpoint visibility is provided via Sysmon-based data sources.

Evidence:
- Installed app list: `evidence/01_installation/apps_list_sysmon,apps_list_office365,apps_list_stream,apps_list_cim,apps_not_installed`
- OSquery absence verification: `evidence/01_installation/osquery_not_found_for_download.png`
Additional security and cloud add-ons were present in the environment but were not overly required for BOTSv3 analysis and are therefore not discussed further.
## 2. Dataset Validation

### 2.1 Index identification
Event count analysis confirmed that the BOTSv3 dataset was successfully ingested into the `botsv3_clean` index.

Evidence:
- `evidence/02_validation/index_discovery.png`

### 2.2 Sourcetype distribution
Sourcetype distribution confirms ingestion of endpoint, cloud, email, and network telemetry required for full kill chain analysis.

Evidence:
- `evidence/02_validation/sourcetype_distribution.png`

### 2.3 Time coverage validation
Time coverage analysis confirms that the dataset spans a meaningful attack window suitable for longitudinal incident analysis.

Evidence:
- `evidence/02_validation/time_coverage.png`

### Ingestion correction note
Initial ingestion resulted in a single forced sourcetype; the dataset was subsequently re-ingested using recursive folder monitoring with automatic sourcetype detection to ensure correct parsing.
Evicence:
- `evidence/02_validation/proof_of_proper_ingestion.png`
## Baseline dataset orientation

A baseline orientation review was performed prior to question-driven investigation to understand dataset scope and typical activity patterns within the `botsv3_clean` index. This included high-level profiling of dataset volume, sourcetype composition, and the most active hosts/users/destinations to support anomaly-driven analysis.

Evidence:
- Dataset overview: `evidence/02_baseline/dataset_overview.png`
- Event volume over time: `evidence/02_baseline/event_volume_timetable.png``evidence/02_baseline/event_volume_timechart.png`
- Top sourcetypes: `evidence/02_baseline/top_sourcetypes.png`
- Top users: `evidence/02_baseline/top_users.png`
- Top destinations: `evidence/02_baseline/top_destinations.png`

## Q1 — OneDrive malicious link upload (User Agent)

### Objective
Identify potentially malicious file upload activity within OneDrive and extract client attribution data (user agent) to support investigation and response.

### Method
Microsoft 365 Management Activity logs were queried for OneDrive file-related operations. Analysis was narrowed to Windows shortcut (`.lnk`) file uploads due to their known abuse in phishing and malware delivery. User identity, source IP, and user agent were extracted for attribution.

### SPL
- `spl/q1_onedrive_useragent.spl`

### Evidence
- OneDrive `.lnk` upload event with user agent visible:  
  `evidence/03_q1/onedrive_upload_useragent.png`

### Key finding
A `.lnk` file upload was observed in OneDrive activity logs. The associated user agent identifies the client context used to perform the upload.

**UserAgent:** `Mozilla/5.0 (X11; U; Linux i686; ko-KP; rv: 19.1br) Gecko/20130508 Fedora/1.9.1-2.5.rs3.0 NaenaraBrowser/3.5b4`
This user agent is highly anomalous, referencing NaenaraBrowser and a ko-KP locale, which is inconsistent with normal enterprise OneDrive usage and indicative of potentially attacker-controlled access.

### Why this matters
- OneDrive logs provide authoritative visibility into cloud file uploads with strong attribution.
- `.lnk` files can execute embedded commands and are frequently used as phishing or malware delivery artefacts.

### SOC relevance and response
- Revoke active user sessions and refresh tokens
- Review Conditional Access posture (MFA, device compliance, sign-in risk)
- Investigate file sharing/download activity and correlate with endpoint execution telemetry

### ATT&CK alignment
- **TA0001 – Initial Access**
- **T1566.001 – Phishing: Spearphishing Attachment**
- **T1204.002 – User Execution: Malicious File**

## Q2 — Macro-enabled email attachment

### Objective
Identify and confirm delivery of a macro-enabled malicious attachment via email.

### Method
SMTP telemetry (`stream:smtp`) was analysed for attachment artefacts associated with security alerts. The original malicious attachment was removed by email security controls and replaced with an alert artefact (`Malware Alert Text.txt`). The original attachment filename was extracted from the alert content using pattern matching.

### Artefacts
- SPL: `spl/q2_macro_attachment.spl`
- Evidence: `evidence/03_q2/macro_attachment.png`

### Macro-enabled extensions
- `.xlsm`

### Delivery phase relevance
Macro-enabled Office documents are commonly used in phishing to deliver malware and require user interaction to execute.

### Answer: attachment filename
**Frothly-Brewery-Financial-Planning-FY2019-Draft.xlsm**

### Confirmed malware
- **Family:** W97M.Empstage
- **Detection:** Email security alert

### SOC response
- Block sender and attachment hash across email security controls
- Identify all recipients and confirm no execution occurred
- Correlate with endpoint telemetry for macro execution
- Harden macro policies (block macros from internet sources)
