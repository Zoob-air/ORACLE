#### Cek Backup
```
set linesize 250 pages 100 trimspool on numwidth 14
col STARTDDATE format a25
col ENDTIME format a25
col STATUS format a15
col OUTPUT_BYTES_DISPLAY format a15
col TIME_TAKEN_DISPLAY format a25
select to_char(start_time,'dd-mm-yyyy hh24:mi:ss') startddate,
to_char(end_time,'dd-mm-yyyy hh24:mi:ss') endtime,
status,INPUT_TYPE,OUTPUT_BYTES_DISPLAY,TIME_TAKEN_DISPLAY
from    V$RMAN_BACKUP_JOB_DETAILS
where start_time like '%SEP%';
```
