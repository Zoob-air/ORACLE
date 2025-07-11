###**ORA-31634**
###**ORA-31664**


oracle@mvtest]$ expdp migrasi/migrasi@tataruang schemas=ARSIP directory=dmptataruang dumpfile=ARSIP_`date +%d_%b_%y`.dmptataruang logfile=ARSIP_`date +%d_%b_%y`.log parallel=4

Export: Release 19.0.0.0.0 - Production on Fri Jul 11 10:12:02 2025
Version 19.18.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
ORA-31634: job already exists
ORA-31664: unable to construct unique job name when defaulted

**Cause : it has 99 jobs with the NOT RUNNING state.**
_QUERY CHECK JOBS_  : 
```
SELECT owner_name,
job_name,
operation,
job_mode,
state
FROM dba_datapump_jobs;
```
```
SELECT 'DROP table ' || owner_name || '.' || job_name || ';'
FROM DBA_DATAPUMP_JOBS
WHERE STATE = 'NOT RUNNING';
```
```
SQL > DROP table MIGRASI.SYS_EXPORT_SCHEMA_68;
```
