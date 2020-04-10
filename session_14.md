# Session 14 - Performing Backups with RMAN

## Go to RMAN
rman target /

## List all datafile and tempfiles in RMAN
report schema;

## List all RMAN parameters
show all;			// #default points to unchanged parameters

## Change redundancy policy
configure retention policy to redundancy 2;

## Make backup followed by backup of Controlfile and SPFILE
configure controlfile autobackup on;

## Setting location and name of our backup sets
configure channel device type disk format '/home/oracle/BACKUP/full_%u_%s_%p';

## Steps to Backup in Whole, Full, Off-line mode (NOARCHIVELOG mode)
1. shutdown immediate;
2. startup mount;
3. backup database filesperset=4;

## List all backups
list backup of database;

## Notes:
1. We can perform only Whole Database, Full, and Off-line(Cold) Backup; that is the ONLY option in NOARCHIVELOG mode.



## Terminologies:
Whole Database Backup:
Full Backup:
Off-Line/Cold Backup:
NOARCHIVELOG mode: