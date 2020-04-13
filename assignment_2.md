## Part 1

1. Create new user called DAVE and assign RED and TEMP tablespaces to him,
also DEFAULT profile. He should be allowed to use only 2M of RED tablespace, and also 1M of  INDX tablespace.
```sql
create user dave
identified by dave
default tablespace RED
temporary tablespace TEMP
profile default
quota 2m on RED
quota 1m on INDX;
```

2. Create another user called LARA and assign BLUE and TEMP tablespaces to her,
also DEFAULT profile, but no quota initially for her and her account should be locked. Both users should get role that will allow them to connect to SQL and individual privilege for creating tables.
```sql
create user lara
identified by lara
default tablespace BLUE
temporary tablespace TEMP
profile default
account lock;

grant connect, create table to dave;
grant connect, create table to lara;
```

3. Create new profile called ITPROF so that:
•   One session per user is allowed
•   Idle CPU time is no more than 1 hour
•   Three false logins are allowed and that account should remain locked for 2 minutes after the fourth false login
```sql
create profile ITPROF limit
idle_time 60
sessions_per_user 1
failed_login_attempts 3
password_lock_time 2/1440;

```

4. Assign this profile now to DAVE and enable all restrictions for profile ITPROF. Then connect to SQL*PLUS as DAVE and try to connect to SQL*PLUS as DAVE again (in a second session). What happened?
```sql
alter user dave profile ITPROF;

alter system set resource_limit=true;
```

5. As SYSTEM modify profile ITPROF so that only two false logins are allowed and that password life time is only 2 months with the grace period of 5 days. Then try to login as DAVE with the wrong password 3 times. Wait only 1 minute and provide now the right password. What happened and how can you rectify this problem -> show both methods?
```sql
alter profile ITPROF limit
failed_login_attempts 2
password_life_time 60
password_grace_time 5;

conn dave/d;    #intentional error
conn dave/d;    #intentional error
conn dave/d;    #intentional error

# Method 1: Wait for 2 mintues

# Method 2:
conn / as sysdba;
alter user dave account unlock;
conn dave/dave;
```

6. As SYSTEM try to create replica of SCOTT’s table EMP in both DAVE’s and LARA’s accounts. Was it successful in both cases? How can you fix this problem? After doing that, verify that LARA (after login) may access her table EMP.
```sql
create table dave.emp as
select * from scott.emp;

create table lara.emp as
select * from scott.emp;        # this will give error for not allocated space

alter user lara quota 2m on BLUE;
alter user lara account unlock;

create table lara.emp as 
select * from scott.emp;
select * from emp;
```

7. By joining 2 dictionary views display for users DAVE and LARA their account status, when the account will expire, what profile and tablespaces are assigned and what the  current and maximal byte situation is for these tablespaces?
```sql
select      u.username, u.account_status, u.expiry_date, u.profile, 
            u.default_tablespace, u.temporary_tablespace, 
            ts.bytes, ts.max_bytes
from        dba_users u, dba_ts_quotas ts
where       u.username=ts.username
and         u.username in ('DAVE','LARA');
```

8. Now give individual privilege to DAVE, so that he can browse and add data in a table in any account and also that he can continue to give this privilege. Then connect as DAVE and give the same privilege to LARA?
```sql
grant insert any table to dave with admin option;
grant select any table to dave with admin option;

conn dave/dave;

grant insert any table to lara;
grant select any table to lara;
```

9. As  SYSTEM, check the appropriate dictionary view and observe only system privileges for those two users (show only relevant columns from this view)
```sql
select grantee, privilege
from dba_sys_privs 
where grantee in ('DAVE','LARA');
```


10. Remove the privilege given to DAVE in h), then connect as LARA and try to ADD one row into SCOTT’s table EMP. Was it successful and why? Then repeat step i) and explain what is different now?
```sql
revoke insert any table from dave;
revoke select any table from dave;

conn lara/lara;
desc scott.emp;
insert into scott.emp(empno,ename,job) values(4444,'test','test');

-- The table creation was successful. Because Revoking system privilege from the user in the middle of the user chain will NOT cascade and break the chain. The user at the end of the chain will RETAIN the privilege.

select grantee, privilege
from dba_sys_privs
where grantee in ('DAVE','LARA');

--The thing that is different is Dave has lost Insert any table and select any table privileges.
```

11. In DB Express display situation for DAVE’s account firstly and then for LARA’s account as two different pages.



## Part 2

Firstly run the script cr_orders.sql as user DAVE (you will need to adjust this script regarding Tablespace name here).

1. As DAVE, create a PK constraint for table CUSTOMERS, so that it’s checking can be delayed (till saving) LATER in the future, but for now it will behave like the non-delayed one. The state should be set to check only incoming data. Also add PK constraint for table ORDERS, so that its checking can be delayed PROMPTLY after its creation, while its state should be set to check both existing and incoming data. Related indexes should go to Tablespace  INDX.
```sql
conn dave/dave;

alter table customers
add constraint cust_pk
primary key(cust_code)
deferrable initially immediate
using index tablespace indx
ENABLE  NOVALIDATE;

alter table orders
add constraint ord_pk
primary key (ord_id)
deferrable initially deferred
using index tablespace indx;
```

2. Also add an UK constraint on column NAME in table CUSTOMERS, so that will have default value for either its state or mode, and sub-default for the other one. Think hard here. You may want to check question g) below in order to make a right choice. The related index will also go to tablespace INDX. 
```sql
alter table customers
add constraint cust_uk
unique(name)
NOT deferrable disable;
```

3. Create a FK constraint for table ORDERS so that might be LATER manually delayed (at save time).  This constraint state follows the default value.
```sql
alter table orders
add constraint ord_cc_fk
foreign key(cust_code)
references customers(cust_code)
deferrable initially immediate;
```

4. Then declare a CK constraint with condition that date of delivery may not be later than two weeks from the order date and also not before it. This constraint's mode is the default one, while the state follows the sub-default value.
```sql
alter table orders
add constraint ord_dod_ck
check (date_of_dely>=ord_date
and date_of_dely<=ord_date+7)
enable novalidate;
```

5. As SYSTEM, join two most important constraint dictionary views to display the following: constraint name, type, status, validation, can be deferred or not, is it currently deferred, table name and what column(s) are involved for tables CUSTOMERS and ORDERS in DAVE’s account. 
```sql
select      c.constraint_name, c.constraint_type,c.status,c.validated,
            c.deferrable,c.deferred,c.table_name,cc.column_name
from        dba_constraints c, dba_cons_columns cc
where       c.constraint_name = cc.constraint_name
and         c.table_name in ('CUSTOMERS','ORDERS')
and         c.owner='DAVE'
and         cc.owner='DAVE';
```

6. As DAVE, try to make a short DML script (yours and original), that will show how it is possible to insert a child record before its parent record, if the child FK constraint checking is delayed till save time. Manually perform the change that will allow this check delay.
```sql
set constraint ord_cc_fk deferred;

# Sample script
INSERT INTO orders  VALUES(800,'01-JAN-98','J01',NULL);
INSERT INTO customers VALUES('J01','Sports Unlimited','West');
set constraint ord_cc_fk immediate;
commit;
```

7. Then make another short script that will show how it is possible to enter same names for different customers (three), if the appropriate constraint is turned off at the creation time.
```sql
insert into customers values('123','hussain','TOR');
insert into customers values('124','hussain','MIS');
insert into customers values('125','hussain','SCA');
```

8. Alter in DAVE’s table CUSTOMERS to the default state. What happened? 
```sql
alter table customers
enable constraint cust_uk;      #this will result in error
```

9. Then enable it so that it does NOT check the existing data. If not possible, fix this obstacle by creating a new object in DAVE’s schema.
```sql
alter table customers
enable novalidate constraint cust_uk;
```

10. Next, try to enter a row with an existing customer name. What happened? 
```sql
insert into customers values('126','hussain','OAK');    # will result in error
```

11. Perform the five step recipe for consolidating the situation with the Name column in DAVE’s CUSTOMERS table (get rid of all customers with duplicate names by modifying duplicates). You should download first the script utlexcpt.sql in order to create the 'container' table (if not done before).
```sql
-- Cleaning table with duplicate keys

-- Step 1: Create table to store Exceptions
@utlexcpt;
desc exceptions;

-- Step 2:  Try to validate constraint with exceptions table, which 
--          will collect any exceptions caused in the query run
alter table customers
enable constraint cust_uk
exceptions into exceptions;

select * from exceptions;

-- Step 3: Use NOVALIDATE to prevent incoming data from creating duplicate
alter table customers
enable novalidate constraint cust_uk;

-- Step 4: Check which rows are causing problem
desc customers;
select rowid, name, region
from customers
where rowid in (
                select row_id 
                from exceptions 
                where table_name='CUSTOMERS');

select * from customers;

-- Step 5: Correct the errors
update customers
set name='Hussain 1'
where rowid='AAAXo3AAKAAAAB4AAB';

update customers
set name='Hussain 2'
where rowid='AAAXo3AAKAAAAB4AAC';

update customers
set name='SHAPE UP 2'
where rowid='AAAXo3AAKAAAAB+AAE';

commit;

select * from customers;

-- Step 6: Enable the costraint and clean up exceptions
alter table customers
enable constraint cust_uk;
truncate table exceptions;
```

## Part 3

1. Configure following options in RMAN
•   Number of copies before becoming obsolete is 1
•   Turn on auto backup of Control File
•   Default folder to hold backup sets is /home/oracle/BACKUP and default name for these sets is hot_%u_%s_%p
•   Exclude EXAMPLE tablespace from every Backup
```sql
CONFIGURE RETENTION POLICY TO REDUNDANCY 1; 
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE CHANNEL DEVICE TYPE DISK FORMAT '/home/oracle/BACKUP/hot_%u_%s_%p';
CONFIGURE EXCLUDE FOR TABLESPACE EXAMPLE;
```

2. Perform WHOLE, FULL and COLD database backup. This one may be used later as a base for Incremental backups. Allow only 4 files per set. Now show all database backup sets as of now.
```sql
shutdown immediate;
startup mount;
backup incremental level 0 database filesperset=4;
list backup of database;
ls -l
```

3. Perform HOT and Incremental-Cumulative backup of tablespaces MINE and RED. Then show all backups of tablespace MINE.
```sql
backup incremental 1 cumulative tablespace mine, red;
show backup of tablespace mine;
```

4. Perform HOT and Incremental-Differential backup of both datafiles that belongs to tablespace RED.  Then show all backups of these datafiles. Perform three Log switches.
```sql
select file#, name from v$datafile;     #check datafile number
backup incremental level 1 datafile 10;
backup incremental level 1 datafile 11;
list backup of datafile 10;
list backup of datafile 11;
alter system switch logfile;
alter system switch logfile;
alter system switch logfile;
```

5. Display all files that are due for backup regarding retention policy established in a).
```sql
report need backup;
```

6. Perform  HOT and Incremental-Cumulative backup of database.
```sql
backup incremental 1 cumulative database;
```

7. Display all backup sets that are beyond retention policy established in a).  Then delete all these sets.
```sql
report obsolete;
delete obsolete;
```

8. Now show all backup sets remained. Also, show the content of /home/oracle/BACKUP folder
```sql
list backup;
ls -l
```

## Part 4

1. Create table  BALL as DAVE with one numeric column called Weight. Add 3 rows here and display the content of this table.
```sql
conn DAVE/dave;
create table ball (weight number);
insert into ball values(7);
insert into ball values(8);
insert into ball values(9);
commit;
select * from ball;
```

2. Then go to Linux and remove both Files that belongs to RED tablespace.
```bash
rm red*
```

3. Make two log switches and also perform manual checkpoint. Display info from V$LOG view.
```sql
alter system switch logfile;
alter system switch logfile;
alter system checkpoint;

select group#, sequence#, status from V$LOG;
```

4. Try to display data from table BALL. If you can still see it, then perform manual action that will give you the correct outcome of your search – Error.
```sql
select * from dave.ball;
alter system flush buffer_cache;
select * from dave.ball;
```

5. Display all backup sets of Tablespace RED and note their BS#.
```sql
list backup of tablesapce red;
-- BS number was 30
```

6. Then perform HOT TABLESPACE RECOVERY with a missing non-essential file(s) in 5 steps.
```sql
-- Step 1: Taking tablespace offline:
alter tablespace red offline immediate;
select * from V$RECOVERY_FILE;

-- Step 2: Run restore
restore tablespace red;

-- Step 3: Run recover
recover tablespace red;

-- Step 4: Take tablespace online
alter tablespace red online;

-- Step 5: Inspect no data loss
select * from V$RECOVERY_FILE;
select group#, sequence#, status from V$LOG;
```

7. How many Archived Logs were used for this Recovery and note the first and the last used. Which Backup sets were used here, note their BS#.
```sql
-- Archived log files used:
-- First used:
-- Last used:
-- Backup sets used:
-- BS number:
```

8. Try to display data from table BALL again. Was it success?
```sql
select * from dave.ball;
```