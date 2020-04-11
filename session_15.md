## Session 15 - Performing Backups with RMAN (ARCHIVELOG)

#### Check status of ARCHIVELOG mode (only SYSDBA)
```sql
archive log list;
```

#### Steps of Switching ARCHIVELOG mode:
1. Modify 2 parameters in SPFILE; one to set folder location, the other one to set log name convention:
```sql
alter system set 
log_archive_dest_1='location=/home/oracle/ARCHIVE/' 
scope=SPFILE;

alter system set
log_archive_format = 'arch_%r_%t_%s.log'
scope=spfile;
```
2. Close the database:
```sql
shutdown immediate;
```
3. Mount the database:
```sql
startup mount;
```
4. Turn on archiving:
```sql
alter database archivelog;
```
5. Open the database:
```sql
alter database open;
```
6. Switching log files:
```sql
alter system switch logfile;
alter system switch logfile;
```
7. Check status of archivelog mode:
```sql
archive log list;
```
8. Check status of redo log files:
```sql
select group#, sequence#, archived, status
from V$LOG;
```
9. Check the physical archive files:
```bash
host;
ls -al;
```

#### List all backup of tablespace
```sql
list backup of tablespace mine;
```

#### List all backup of a datafile
```sql
list backup of datafile 4;
```

#### List all copy of a datafile
```sql
list copy of datafile 8;
```

#### Check all files that need backup
```sql
report need backup;
```

#### Backup spcific tablespaces
```sql
backup tablespace mine, users filesperset=2;
```

#### Backup specific datafile;
```sql
backup datafile 4;
```

#### Backup as ordinary copy
```sql
backup as copy datafile 8;
```

#### Delete extra/obsolete backups
```sql
delete obsolete;
```

#### Delete specific backupset
```sql
delete backupset 1;
```

#### Delete multiple without prompting
```sql
delete noprompt backupset 5, 6, 7;
```

#### Delete datafilecopy
```sql
delete datafilecopy 1;
```


#### Types of backup we can perform in ARCHIVELOG mode:
1. Whole Database/Tablespace/Datafile/Archived Log
2. Full/Incremental Level (0 or 1)
3. Off-line (Cold) or On-line (Hot)


