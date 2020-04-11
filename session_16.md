## Session 16 - More Backups with RMAN (ARCHIVELOG)

#### Including Archivelog in incremental level 0 backup
```sql
backup incremental level 0 database
plus Archivelog
fileperset=5;
```

#### Incremental Level 0 backup of specific tablespace
```sql
backup incremental level 0
tablespace users, mine;
```

#### Incremental Level 0 backup of specific datafile;
```sql
backup incremental Level 0
datafile 4;
```

#### Incremental Level 1 backup of specific tablespace
```sql
backup incremental Level 1
tablespace mine;
```

#### Incremental Level 1 Cumulative backup for specific datafile
```sql
Backup incremental level 1 Cumulative
datafile 4;
```

