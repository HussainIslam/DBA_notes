## Session 10 - Creating and Altering Users

#### Relevant tables
* dba_users: to see users
* dba_ts_quotas: to show dba given tablespace quotas to users
* dba_profiles: to see profiles

#### Query to create new user
```sql
create user hussain             //primary command and the name of the account
identified by password          //the password with which the user will be authenticated
profile default                 //profile template
password expire                 //password will expire when the user logs in first time
account unlock                  //account will be at unlocked status
default tablespace MINE         //tablespace that will be default
temporary tablespace TEMP;      //tablespace that is be use temporarily
```

#### Query all the 'open' users
```sql
select username, account_status, default_tablespace,
from dba_users
where account_status='OPEN';
```

#### Connect to SYSDBA
```sql
conn / as sysdba;
```

#### Granting Connect permission to a user
```sql
grant connect to foobar;
```

#### Granting CREATE TABLE to a user
```sql
grant create table to foobar;       //granted permission to create table, not any space
```

#### Give user quota on tablespace to create data
```sql
alter user foobar               //primary command with user name
quota 5m                        //how much space we are allocating
on mine;                        //which allocation will have the allocation
```

#### Give user unlimited quota
```sql
alter user foobar
quota unlimited
on mine;
```

#### Freeze user quota
```sql
alter user lisa                 //this user will no longer show up in quotas view
quota 0                         //also freeze current quota situation for the user
on joke;                        //no new extents will be allocated
                                //data can be inserted if current extension has space
```

#### Query given user quotas
```sql
select tablespace_name, username, bytes, max_bytes
from dba_ts_quotas
where username in ('TOM','FOOBAR');
```

#### Create user with quota
```sql
create user lisa
identified by lisa
default tablespace joke
temporary tablespace temp
quota 512k on joke;
```

#### DBA creating table for a user
```sql
create table lisa.dept as               //create a table for lisa copying the data from scott.dept
    select * from scott.dept;
```

#### DBA query information from another user's table
```sql
select * from lisa.dept;
```

#### Remove none empty user
```sql
drop user lisa cascade;
```

#### Create user profile
```sql
create profile "PROG" limit                             #creating a new profile called prog with following limits
    cpu_per_session                 unlimited           #CPU time limit for a session, expressed in hundredth of seconds
    cpu_per_call                    unlimited           #CPU time limit for a call (a parse, execute, or fetch)
    connect_time                    unlimited           #total elapsed time limit for a session, in minutes.
    idle_time                       60                  #permitted periods of continuous inactive time during a session
    sessions_per_user               2                   #number of concurrent sessions the user can have
    logical_reads_per_session       unlimited           #permitted number of data blocks read(memory and disk)
    logical_read_per_call           unlimited           
    private_sga                     unlimited           #amount of private space a session can allocate
    composite_limit                 unlimited           #total resource cost for a session, expressed in service units
    password_life_time              120                 #number of days the same password can be used for authentication
    password_grace_time             7                   #password expires if not changed with this time, conn refused afterwards
    password_reuse_max              unlimited           #number of times a password can be reused
    password_reuse_time             60                  #number of days before which a password cannot be reused
    password_verify_function        null                #can pass PL/SQL script as argument
    failed_login_attempts           2                   #will be locked after n+1 failed attempts
    password_lock_time              3/1440;             #number of days an account will be locked
```

#### Query to see the resources used by a profile
```sql
select profile, resource_name, limit, resource_type
from dba_profiles
where profile = 'PROG'
order by 1, 4, 2;
```

#### Unlock user from SYSDBA
```sql
alter user foobar account unlock;
```

#### Enforce resource limiting
```sql
alter system 
set resource_limit=true;
```


## Session 11 - Granting/Revoking System/Object Privilages

#### Important tables
* dba_sys_privs: all the system privilages
* dba_tab_privs: table privilages

#### Query number of system privilages
```sql
select count(*) 
from dba_sys_privilates;    
```

#### Query the privilages of users
```sql 
select *
from dba_sys_privs
where grantee in ('TOM','JANE','FOOBAR');
```

#### Give create tablespace and provide a role
```sql
grant create tablespace, select_catalog_role to tom;
grant create table, resource to jane;
```

#### Give a user privilage to create table in other user and make others dba
```sql
grant create any table to TOM           //tom can create a table in any user's account
with admin option;                      //tom can make any other user an admin
```

#### Take away a privilage
```sql
revoke create any table from tom;
```

#### Give a user some object privilage (can be done by DBA or owner of object)
```sql
grant insert, select                    //granting insert and select privilages
on scott.dept                           //target object will be scott.dept
to tom                                  //grantee is tom
with grant option;                      //tom can grant privilages to others
```

#### Query table privilages
```sql
select grantee, privilege, grantable, grantor
from dba_tab_privs
where owner='SCOTT' and table_name='DEPT';
```

#### Revoke object privilage from user
```sql
revoke select 
on scott.dept
from tom;
```

#### NOTE: 
1. Revoking system privilege from the user in the middle of the user chain will NOT cascade and break the chain. The user at the end of the chain will RETAIN the privilege.
2. Revoking the object privilege from the user in the middle of the user chain will cascade and break the chain. The user at the end of the chain will LOSE the privilege.


#### Session 12 - Creating/Altering Roles

#### Important Tables
* **dba_roles:** information on Roles
* **dba_role_privs:** role privileges
* **dba_tables:** all relational tables in database
* **role_sys_privs:** all system privs to roles
* **role_tab_privs:** all object privs to roles
* **role_role_privs:** all roles granted to other roles

#### Create a role
```sql
create role dev not identified;
```

#### Give roles to a role profile
```sql
grant connect, resource to dev;
```

#### Give privileges to a role profile
```sql
grant create any table to dev;
```

#### Give Object privilege to a Role
```sql
grant insert, select on scott.dept to dev;
```

#### Query all the possible roles
```sql
select roles from dba_roles;
```

#### Grant a Role to a User;
```sql
grant dev to jane;
```

#### Query users and their roles
```sql
select grantee, granted_role
from dba_role_privs
where grantee in ('SCOTT', 'TOM', 'JANE','DEV')
order by grantee;
```

#### Give user some role profile but restrict some
```sql
alter user tom default role all except select_catalog_role;
```
#### Query a users roles and default_role
```sql
select granted_role, default_role 
from dba_role_privs 
where grantee='TOM';
```

#### Query session privileges
```sql
select *
from session_privs;
```

#### Query session roles
```sql
select *
from session_roles;
```
#### Enable role for current session (disables all other roles)
```sql
set role select_catalog_role;
```

#### Enable all roles
```sql
set role all;
```

#### Query roles and system privileges
```sql
select * 
from role_sys_privs
where role='CONNECT';
```

#### Query object privileges granted to role
```sql
select owner, table_name, privilege
from role_tab_privs
where role='DEV';
```

#### Query granted roles to other roles
```sql
select granted_role 
from role_role_privs
where role='DEV';
```

#### Query number of granted roles to a role
```sql
select count(granted_role)
from role_role_privs
where role='DBA';
```

#### Notes
1. Since Oracle 12c, UNLIMITED TABLESPACE system privilege is not included in the RESOURCE role and can not be granted to a Role. It may be given to the User. In order to add some Object privilege to a created role, we need to edit it later.


## Session 13 - Adding/Viewing Table Constraints

#### Important data dictionaries
* **dba_constraints:** all constraint definitions on all tables in the database
* **dba_cons_columns:** all columns in the database that are specified in constraints
* **dba_indexes:** all indexes in the database

#### Give an user quota on a tablespace
```sql
alter user tom
quota 1m
on indx;
```

#### Give browse data dictionary role
```sql
grant select_catalog_role to tom;
```

#### Query constraint information
```sql
select      constraint_name, constraint_type, search_condition, 
            status, deferrable, deferred, validated, table_name
from        dba_constraints
where       table_name in ('CUSTOMERS','ORDERS','PRODUCTS')
and         owner='TOM'
order by    8, 2, 1;
```

#### Query columns names that have constraints
```sql
select constraint_name, column_name, position, table_name
from dba_const_column
where owner = 'TOM'
and table_name in ('CUSTOMERS','ORDERS','PRODUCTS')
order by 4,1;
```

#### Query what indexes are created by server
```sql
select index_name, index_type, uniqueness
from dba_indexes
where index_name in        (select constraint_name
                           from dba_constraints
                           where owner='TOM'
                           and table_name in ('CUSTOMERS','ORDER','PRODUCTS'));
```

#### Apply constraints and export exceptions
```sql
alter table products
enable constraint prod_uk
exceptions into exceptions;
```

#### Enable NOVALIDATE
```sql
alter table products
enable novalidate
constraint prod_uk;
```

#### Update a record
```sql
update products
set prod_code=2315
where rowid = 'AAAXmbAAIAAAAArAAB';
```

#### Truncate
```sql
truncate table exceptions;
```

#### Query constraints of a table of a users
```sql
select constraint_name, owner, constraint_type, status, deferrable, deferred
from dba_constraints
where table_name='ORDERS'
and owner='TOM';
```

#### Defer a constraint
```sql
set constraint ord_cc_fk deferred;          //deferred:     data will be validated when committing
set constraint ord_cc_fk immediate;         //immediate:    data will be validated when committing 
```

#### Notes
1. Index created by server are always created implicitly when developers specify either pk or UK constraints (but not if disabled) and they will be unique (unless created as DEFERRABLE, then they will be NONUNIQUE).
2. A NOVALIDATE constraint is basically a constraint which can be enabled but for which Oracle will not check the existing data to determine whether there might be data that currently violates the constraint.
3. FK constraint is created as DEFRRABLE and IMMEDIATE (sub-default mode) and it will perform line by line check (like NOT DEFERRABLE one )  it will ignore "bad" rows, but will process all other  rows that have no errors.  Later, you may switch this constraint to DEFERRABLE sub-mode and then it will perform only one check at Commit time.


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
show all;                                       // #default points to unchanged parameters
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
