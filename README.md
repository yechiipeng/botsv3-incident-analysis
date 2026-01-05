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
