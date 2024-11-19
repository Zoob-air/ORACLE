#### Cek Status DB
```
SET LINESIZE 1000
SET PAGESIZE 50
COLUMN database_name FORMAT A8
COLUMN open_mode FORMAT A15
COLUMN database_role FORMAT A8
COLUMN platform_name FORMAT A25
COLUMN host_name FORMAT A15
COLUMN instance_name FORMAT A8
COLUMN status FORMAT A8
COLUMN version FORMAT A10
COLUMN startup_time FORMAT A20

SELECT 
    db.name AS database_name,
    db.open_mode,
    db.database_role,
    db.platform_name,
    inst.host_name,
    inst.instance_name,
    inst.status,
    inst.version,
    TO_CHAR(inst.startup_time, 'DD-MON-YY HH24:MI:SS') AS startup_time
FROM 
    (SELECT 
         name, 
         open_mode, 
         database_role,
         platform_name
     FROM 
         v$database) db,
    (SELECT 
         host_name, 
         instance_name, 
         status, 
         version, 
         startup_time
     FROM 
         v$instance) inst;
```
#### Cek Backup Order By Month
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
#### Cek Backup Order By Month And Year
```
SET LINESIZE 250 PAGES 100 TRIM SPOOL ON NUMWIDTH 14
COL STARTDDATE FORMAT A25
COL ENDTIME FORMAT A25
COL STATUS FORMAT A15
COL OUTPUT_BYTES_DISPLAY FORMAT A25
COL TIME_TAKEN_DISPLAY FORMAT A25

SELECT TO_CHAR(start_time, 'dd-mm-yyyy hh24:mi:ss') STARTDDATE,
       TO_CHAR(end_time, 'dd-mm-yyyy hh24:mi:ss') ENDTIME,
       STATUS,
       INPUT_TYPE,
       OUTPUT_BYTES_DISPLAY,
       TIME_TAKEN_DISPLAY
FROM V$RMAN_BACKUP_JOB_DETAILS
WHERE TO_CHAR(start_time, 'MON') = 'JUL'
  AND EXTRACT(YEAR FROM start_time) = 2024;
```
#### Cek All Backup
```
COL STATUS FORMAT a9
COL hrs FORMAT 999.99

  SELECT SESSION_KEY,
         INPUT_TYPE,
         STATUS,
         TO_CHAR (START_TIME, 'mm/dd/yy hh24:mi')     start_time,
         TO_CHAR (END_TIME, 'mm/dd/yy hh24:mi')       end_time,
         ELAPSED_SECONDS / 3600                       hrs
    FROM V$RMAN_BACKUP_JOB_DETAILS
ORDER BY SESSION_KEY;
```
#### Monitoring User
```
SELECT
    MAX(a.SAMPLE_TIME) AS SAMPLE_TIME,
    MAX(a.SAMPLE_TIME_UTC) AS SAMPLE_TIME_UTC,
    MAX(a.SESSION_ID) AS SESSION_ID,
    MAX(a.SESSION_SERIAL#) AS SESSION_SERIAL#,
    MAX(a.USER_ID) AS USER_ID,
    MAX((SELECT USERNAME FROM DBA_USERS WHERE USER_ID = a.USER_ID)) AS USERNAME,
    MAX(a.MACHINE) AS MACHINE,
    MAX(a.SQL_ID) AS SQL_ID,
    MAX((SELECT DBMS_LOB.SUBSTR(SQL_FULLTEXT, 4000, 1) FROM V$SQL WHERE SQL_ID = a.SQL_ID AND ROWNUM <= 1)) AS SQL_TEXT,
    MAX(a.PROGRAM) AS PROGRAM,
    MAX(a.MODULE) AS MODULE
FROM
    V$ACTIVE_SESSION_HISTORY a
WHERE
    SQL_ID IS NOT NULL AND
    USER_ID != 0 AND
    MODULE NOT IN ('SWJobEngineWorker2x64.exe', 'oracle@dbrepo (TNS V1-V3)', 'DBeaver 22?0?2 ? Metadata', 'DBeaver 21?3?0 ? Metadata') AND
    SQL_ID IN ('fzadcpk5njuu9') AND
    TO_CHAR(a.SAMPLE_TIME_UTC, 'MM-DD-YYYY') = '12-15-2023' -- Replace '12/11/2023' with your desired date
GROUP BY a.SESSION_ID
```
#### Melihat User Yang Sedang Menjalankan Program
```
select a.sid, a.serial#, a.schemaname, a.status, a.osuser, a.machine, a.program, sysdate, a.prev_exec_start, (SYSDATE - PREV_eXEC_START) * 60 * 60 * 24 RUN_IN_SEC,  a.sql_id, b.spid  
from gv$session a, gv$process b  
where a.paddr= b.addr  
and a.schemaname not in ('SYS','SYSTEM','SYSMAN','SYSMON') 
and status ='ACTIVE'   
order by a.prev_exec_start, a.sid;
```
#### Cek User Spesifik SQL_ID
```
SELECT 
    a.SAMPLE_TIME, 
    a.PROGRAM, 
    a.MODULE, 
    a.MACHINE, 
    a.PORT, 
    a.SQL_ID, 
    a.SESSION_ID, 
    a.SESSION_SERIAL#,
    (SELECT MAX(u.USERNAME)  -- Menggunakan MAX() sebagai contoh
     FROM v$session v
     JOIN DBA_USERS u ON v.USERNAME = u.USERNAME
     WHERE v.SID = a.SESSION_ID AND v.SERIAL# = a.SESSION_SERIAL#
    ) AS USERNAME,
    (SELECT MAX(s.SQL_TEXT)  -- Menggunakan MAX() sebagai contoh
     FROM v$sql s
     WHERE s.SQL_ID = a.SQL_ID
    ) AS SQL_TEXT
FROM 
    v$active_session_history a
WHERE 
    a.SQL_ID = '0x24edc8cf'
ORDER BY 
    a.SAMPLE_TIME DESC;
```
#### Monitoring Rman Proses
```
select s.inst_id, a.sid, CLIENT_INFO AS Ch, a.STATUS,
open_time, round(BYTES/1024/1024,2) "SOFAR Mb", round(total_bytes/1024/1024,2)
TotMb, io_Count,
round(BYTES/TOTAL_BYTES*100,2) "% Complete",  a.type, filename
from gv$backup_async_io a, gv$session s
where not a.STATUS in ('UNKNOWN') and s.status='ACTIVE' and a.STATUS <> 'FINISHED'
and a.sid=s.sid order by 6 desc,7;
```
#### Cek Character Set Oracle Database
```
select * from NLS_DATABASE_PARAMETERS where parameter='NLS_CHARACTERSET';
```
#### Cek Invalid Object
```
SELECT owner,
       object_type,
       object_name
FROM   dba_objects
WHERE  status = 'INVALID'
       AND last_ddl_time BETWEEN TO_DATE('04/04/2024', 'dd/mm/yyyy')
                                AND TO_DATE('04/04/2024', 'dd/mm/yyyy')
       AND owner NOT IN ('SYS', 'SYSTEM', 'DBSNMP', 'TRACESVR')
       AND object_type NOT IN ('SEQUENCE')
ORDER BY last_ddl_time DESC;
```
#### Cek Valid Object
```
SELECT owner,
       object_type,
       object_name
FROM   dba_objects
WHERE  status = 'VALID'
       AND last_ddl_time BETWEEN TO_DATE('04/04/2024', 'dd/mm/yyyy')
                                AND TO_DATE('04/04/2024', 'dd/mm/yyyy')
       AND owner NOT IN ('SYS', 'SYSTEM', 'DBSNMP', 'TRACESVR')
       AND object_type NOT IN ('SEQUENCE')
ORDER BY last_ddl_time DESC;
```
#### Cek Datafile Status after restore
```
set numf '999999999999999';
select file#, checkpoint_change# from v$datafile_header order by 2;
```
#### Cek FRA
```
SELECT
    NAME AS "NAME",
    ROUND(SPACE_LIMIT / 1024 / 1024, 2) AS "SIZE_MB",
    ROUND(SPACE_USED / 1024 / 1024, 2) AS "USED_MB",
    ROUND((SPACE_LIMIT - SPACE_USED) / 1024 / 1024, 2) AS "FREE_MB",
    ROUND((SPACE_USED / SPACE_LIMIT) * 100, 2) AS "USED_PCT",
    ROUND(((SPACE_LIMIT - SPACE_USED) / SPACE_LIMIT) * 100, 2) AS "FREE_PCT"
FROM
    V$RECOVERY_FILE_DEST;
```
#### Cek Usage Recovery Destination
```
SELECT
    name,
    round(space_limit / power(1024, 3), 2) AS size_gb,
    round(space_used / power(1024, 3), 2) AS used_gb,
    round(space_limit / power(1024, 3) - space_used / power(1024, 3), 2) AS available_gb
FROM
    v$recovery_file_dest;
```
#### Cek InActive Session
```
SELECT S.SID,
       S.SERIAL#,
       S.USERNAME DATABASE_USER,
       TO_CHAR(LOGON_TIME, 'DD-MON-YYYY HH24:MI:SS' ) LOGON_TIME,
       S.STATUS,
       S.MACHINE,
       S.PORT,
       S.PROGRAM,
      CASE
        WHEN LAST_CALL_ET< 60 THEN LAST_CALL_ET || ' Seconds'
        WHEN LAST_CALL_ET< 3600 THEN ROUND(LAST_CALL_ET/60) || ' Minutes'
        WHEN LAST_CALL_ET< 86400 THEN ROUND(LAST_CALL_ET/60/60,1) || ' Hour(s)'
      ELSE
        ROUND(LAST_CALL_ET/60/60/24,1) || ' Day(s)'
     END INACTIVE_TIME
FROM
       V$SESSION S, V$PROCESS P
WHERE
      S.PADDR=P.ADDR AND
      S.STATUS = 'INACTIVE'
      -- To find out all session that are inactive more than 1 hour
      -- AND S.LAST_CALL_ET  >= 3600
ORDER BY LAST_CALL_ET DESC;
```
