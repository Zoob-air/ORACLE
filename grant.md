### **Grant select untuk schema SIMBG_READ ke semua table SUB_SIMBG** <br>
- Jalankan dengan SYS <br>

Table
```
BEGIN
   FOR t IN (SELECT table_name FROM dba_tables WHERE owner = 'SUB_SIMBG') LOOP
      EXECUTE IMMEDIATE 'GRANT SELECT ON SUB_SIMBG.' || t.table_name || ' TO SIMBG_READ';
   END LOOP;
END;
/
```

View 
```
BEGIN
   FOR v IN (SELECT view_name FROM dba_views WHERE owner = 'SUB_SIMBG') LOOP
      EXECUTE IMMEDIATE 'GRANT SELECT ON SUB_SIMBG.' || v.view_name || ' TO SIMBG_READ';
   END LOOP;
END;
/
```
