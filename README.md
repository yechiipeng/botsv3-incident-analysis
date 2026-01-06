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