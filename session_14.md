## Session 14 - Performing Backups with RMAN

#### Go to RMAN
```bash
rman target /
```

#### List all datafile and tempfiles in RMAN
```sql
report schema;
```

#### List all RMAN parameters
```sql
show all;			                            // #default points to unchanged parameters
```

#### Change redundancy policy of how many backups to keep
```sql
configure retention policy to redundancy 2;     #3rd or more backups will be obsolete
```

#### Make backup followed by backup of Controlfile and SPFILE
```sql
configure controlfile autobackup on;            #constrol files wills will also be backed up with usual backup
```

#### Setting location and name of our backup sets
```sql
configure channel device type disk format '/home/oracle/BACKUP/full_%u_%s_%p';
```

#### Steps to Backup in Whole, Full, Off-line mode (NOARCHIVELOG mode)
1. ```shutdown immediate;```
2. ```startup mount;```
3. ```backup database filesperset=4;```

#### List all backups
```sql
list backup of database;
```

#### Exclude some tablespace from backup
```sql
configure exclude for tablespace example;
```

#### Check obsolede backups
```sql
report obsolete;
```

#### Notes:
1. We can perform only Whole Database, Full, and Off-line(Cold) Backup; that is the ONLY option in NOARCHIVELOG mode.


#### Questions:
1. what is `filesperset`?
