## Session 17 - Open (HOT) Complete Recovery (Non-essential file missing)

#### Alter checkpoint
```sql
alter system checkpoint;
```

#### Flush System buffer
```sql
alter system flush buffer_cache;
```

#### Restoring backup for a tablespace
1. Take tablespace offline and make sure the tablespace is offline
```sql
alter tablespace mine offline immediate;
select * from V$RECOVER_FILE;
```
2. Restore missing/corrupted tablespace in RMAN
```sql
restore tablespace mine;
```
3. Recover missing/corrupted tablespace in RMAN
```sql
recover tablespace mine;
```
4. Place tablespace online
```sql
alter tablespace mine online;
```
5. Inspect no data loss
```sql
select * from V$RECOVER_FILE;
select * from tom.play;         # as sysdba from SQL
select group#, sequence#, status from V$LOG;
```

