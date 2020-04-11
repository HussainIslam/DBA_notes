## Session 18 - Closed (Cold) Complete Recovery (Essential File missing)

#### Mimicking Essential File loss
```sql
shutdown abort;
rm undotbs01.dbf;
```

#### Cold Recovery steps:
1. Mount database and inspect for recovery
```sql
startup;        # it will stop at mount. alternative: startup mount;
SELECT FILE#, ONLINE_STATUS, ERROR, CHANGE# FROM V$RECOVER_FILE;
```
2. Restore missing datafile (or tablespace)
```sql
restore datafile 4;
```
3. Recover missing datafile
```sql
recover datafile 4;
```
4. Open database
```sql
alter database open;
```
5. Verify recovery file and no data loss
```sql
SELECT FILE#, ONLINE_STATUS, ERROR, CHANGE# FROM V$RECOVER_FILE;
select * from tom.play;
SELECT  GROUP#, SEQUENCE#, STATUS  FROM V$LOG;
```

#### Notes
1. Datafile 4 from the UNDOTBS1 tablespace becomes unusable. This is one of TWO ESSENTIAL Datafiles (the other one is from SYSTEM Tablespace) and they can NOT be taken offline. In that case Database will most likely shut down itself and you will need to perform CLOSED COMPLETE RECOVERY. This type Recovery happens while in MOUNT state and NO data loss will occur, because ALL Archive Log files since tha LAST FULL Backup of Datafile 4 are available.
2. In the case that just one of these Archive Log files (needed for recovery) is lost or corrupted, you can perform only INCOMPLETE RECOVERY till Point in Time in the Past, when the Redo Log file (of that lost Archive Log file) became current OR till SCN that was saved with the previous Redo Log file (the LAST_CHANGE#) 