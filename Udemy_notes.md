## Backup and Recovery (User Managed)

### Types of Failure:
1. **User Process Failure:** User system shutdown, program got error. Rollback uncommitted data. Checked by PMON. Usually DBA intervention not needed.
2. **Network Failure:** Listener failed, NIC failure, network connection failure. Configure multiple listener, multiple NIC, multiple network connecitons. No DBA neeeded.
3. **User Error:** Drop a table, delete datafile. DBA has to recover. DBA give access only required privilages. 
4. **Instance Failure:** Power outage, hardware failure, Critical process failure. Oracle Restart will attempt to recover, if turned on. If not possible, DBA interviens. 
5. **Media Failure:** Disk drive fail, disk controller fail, datafiles corrupted. Restore the files from backup. 

### NOARCHIVELOG mode and Redo log:
Redo logs are circular type. This means that the LGWR (Log Writer) will write to a log file until it is full. When the log file gets full, it will move to the next log file. LGWR will keep doing this until it has written to all the log files. After that LGWR will move back to first redo log file and literally will overwrite everything it has written before. 

**NOARCHIEVELOG mode ON:** In such scenario, keeping redo logs doesn't seem to be very helpful because we are overwriting all that redo's. We can only recover only upto the point where we backed-up our files.

**NOARCHIVELOG mode OFF:** In such scenario, when the LGWR moves writing to next redo file (after one file is full), another process ARCH (Archiver) will copy the full redo log file to a archieve location. That means we are backing up all the redo log file. 

### Backup Types
There are mainly two types of back-ups, which are further divided into sub categories:

1. **Physical Backup**
    1. **No Archive Log Mode**
        * *Cold Backup (Off-line):* Shutdown the database before backing up. May not be feasible for my business. This is also called _consistent backup_ because System Change Number (SCN) in data blocks and SCN in Control Files will be same.
    2. **Archive Log Mode**
        * *Cold Backup (Off-line):*
        * *Hot Backup (Online):* Taking the backup when the back is running and users are using the database. This is right backup for many business. This is also called _inconsistent back_.
2. **Logical Backup**
    * *Data Pump/Export:* Backup of a specific table, for example.

Other types of backup include:

1. **Whole Database Backup:** This includes all datafiles and at least 1 control file.
2. **Partial Database backup:** This includes only required tablespaces, datafiles, and control files.
3. **Full Backup:** all datafile blocks that contain data are backed up. This applies to one or more data files and up to all of them.
4. **Incremental Backup:** This is opposite to Full Backup. It backsup only the data blocks that are new since the last backup. This is used to reduce backup time and space.

### Cold Backup (NOARCHIVELOG)
1. Before shutting down, determine the location of datafiles, control files, and other files.
2. Shutdown the database in immediate/transactional/normal.
3. Perform backup of datafiles, control files and other files.
4. Startup database.

### Cold Backup (ARCHIVELOG)
This is similar to NOARCHIVELOG mode except in the 3rd step, we also backup the Archive Log files.

### Hot Backup (ARCHIVELOG)
1. Make sure the database is in achiving mode
2. Determine the locations of the datafiles, control files, and other files
3. Check the current maximum sequence number from online redo log files (eg 41)
4. Put the database in backup mode
5. Backup data files using the OS utility cp
6. Take the database out of backup mode
7. Trigger a checkpoint (archive the current logfile)
8. Backup the control file
9. Find the current maximum sequence number from online redo log files (eg 46)
10. Take the backup of archived logs generated during backup (41 - 45)

### Incremental Backup
Each datablock has a SCN (System Change Number). During the incremental backup, it will check the current SCN with the checkpoint SCN of the parent incremental backup. If datablock SCN is greather than checkpoint SCN, the datablock is copied. Incremental Backup has 2 types of backups:

1. **Level 0:** Like full backup, backs up entire data, will be used as parent for Level 1 backup. 
2. **Level 1:** This has further 2 types:
    1. **Differential:** Take backup only the data blocks that were added since the last Level 0 or Level 1 backup.

    ![Differential Back](https://docs.oracle.com/cd/B19306_01/backup.102/b14192/img/brbsc009.gif)
    
    2. **Cumulative:** Take backup of the data blocks that were added since the last Level 0 backup.

    ![Cumulative Backup](https://docs.oracle.com/cd/B19306_01/backup.102/b14192/img/brbsc010.gif)

#### Cumulative vs. Incremental Backup:
1. **Recovery Speed:** Cumulative backups are faster because fewer backups need to be applied.
2. **Backup Speed:** Differential backups are faster because it doesn't duplicate previously backed up data.
3. **Disk space usage:** Differential backups take less space.

### What is RMAN?
**R**ecovery **MAN**ager (RMAN) is a free utility by Oracle. It has powerful job control and scripting language. This also provides API to other backup softwares. It is used perform backup and recovery on databases, manually and automatically. It can do full backup or incremental backups. This also allows compress the backup set. Other benefits of RMAN are:

* Parallel backups
* Generates log of backups
* Allows hot and cold backups
* Allows backup on tape (with Media Management Software)
* Knows what backup is redundant/obsolete.
* Oracle strong recommends to run in ARCHIVELOG mode.

### RMAN Terminologies
* **Target Database:** This is database we are backing up or restoring.
* **Media Manager:** Interface for sequential media devices (like tape). Media manager controls these devices duing backup or recover, manages loading, labeling and unloading. This is optional to use.
* **Recovery Catalog:** This is seperate database that tracks all the logs related to backup. This needs to be registered with the target database. This is optional to use.

* **Retention Policy:** This means how long do we keep the backup. Once the back gets older than time specified, it is marked as obsolete.

* **Parallelism:** Backups can be performed in parallel, if we have multiple number of disks.

* **Image Copies:** Creates copy of original file, like _copy_ command. Includes free and corrupted blocks for copying. Only files needed can be retrieved while restoring files from backup.

![Image Copies](https://i.ibb.co/2kpB5Wm/image-copies.jpg)

* **Backup Sets:** Collection of one or more _binary files_ that contain data, files, control files, parameter files or archived log files. This doesn't backup empty data. They can be compressed. Entire backup needs to be restored before we extract specific files.

![Backup Sets](https://i.ibb.co/zVw9cPz/backup-sets.jpg)

### RMAN Commands:
1. `SHOW ALL;`: Show all the configuration parameters by RMAN.
2. `CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;`: All the backups will be kept for 7 days after which it marks them obsolete.
3. `CONFIGURE CONTROLFILE AUTOBACKUP ON;`: Automatically backup control files while backing up the database.
4. `BACKUP DATABASE;`: Backsup the database.
5. `BACKUP AS COPY DATABASE;`: This is used to copy only the datafiles.
6. `BACKUP CURRENT CONTROLFILE;`: Used to backup the current control file.
7. `BACKUP AS BACKUPSET DATAFILE 'ORACLE_home/.../users01.dbf', 'ORACLE_home/.../tbs01.dbf';`: Backup 2 files in format of BACKUPSET.
8. `BACKUP ARCHIVELOG COMPLETION TIME BETWEEN 'SYSDATE-30' AND 'SYSDATE';`: Backup all the archivelog that are within last 30 days.
9. `BACKUP TABLESPACE system, users, tbsl;`: Backup these 3 tablespaces.
10. `BACKUP SPFILE;`: Backup the spfile.
11. `LIST BACKUP OF DATABASE;`: List all the backups that we have performed.
12. `BACKUP INCREMENTAL LEVEL 0 DATABASE;`: Perform incremental Level 0 backup.
13. `BACKUP INCREMENTAL LEVEL 1 DATABASE;`: Perform incremental Level 1 backup. The default is differential backup.
14. `BACKUP INCREMENTAL LEVEL 1 CUMULATIVE DATABASE;`: Perform incremental Level 1 cumulative backup.
15. `BACKUP INCREMENTAL LEVEL 1 TABLESPACE SYSTEM DATAFILE 'ORA_HOME/.../tbs01.dbf';`: Backup incrementally a specific datafile.
16. `BACKUP INCREMENTAL LEVEL = 1 CUMULATIVE TABLESPACE users;`: Cumulative backup on a tablespace.