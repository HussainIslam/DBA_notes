## Session 10 - Creating and Altering Users

#### Relevant tables
* dba_users: to see users
* dba_ts_quotas: to show dba given tablespace quotas to users
* dba_profiles:	to see profiles

#### Query to create new user
```sql
create user hussain				//primary command and the name of the account
identified by password			//the password with which the user will be authenticated
profile default					//profile template
password expire					//password will expire when the user logs in first time
account unlock					//account will be at unlocked status
default tablespace MINE			//tablespace that will be default
temporary tablespace TEMP;		//tablespace that is be use temporarily
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
grant create table to foobar;		//granted permission to create table, not any space
```

#### Give user quota on tablespace to create data
```sql
alter user foobar				//primary command with user name
quota 5m						//how much space we are allocating
on mine; 						//which allocation will have the allocation
```

#### Give user unlimited quota
```sql
alter user foobar
quota unlimited
on mine;
```

#### Freeze user quota
```sql
alter user lisa					//this user will no longer show up in quotas view
quota 0							//also freeze current quota situation for the user
on joke;						//no new extents will be allocated
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
create table lisa.dept as 				//create a table for lisa copying the data from scott.dept
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
create profile "PROG" limit 							#creating a new profile called prog with following limits
	cpu_per_session					unlimited			#CPU time limit for a session, expressed in hundredth of seconds
	cpu_per_call					unlimited			#CPU time limit for a call (a parse, execute, or fetch)
	connect_time					unlimited			#total elapsed time limit for a session, in minutes.
	idle_time						60					#permitted periods of continuous inactive time during a session
	sessions_per_user				2					#number of concurrent sessions the user can have
	logical_reads_per_session		unlimited			#permitted number of data blocks read(memory and disk)
	logical_read_per_call			unlimited			
	private_sga						unlimited			#amount of private space a session can allocate
	composite_limit					unlimited			#total resource cost for a session, expressed in service units
	password_life_time				120					#number of days the same password can be used for authentication
	password_grace_time				7					#password expires if not changed with this time, conn refused afterwards
	password_reuse_max				unlimited			#number of times a password can be reused
	password_reuse_time				60					#number of days before which a password cannot be reused
	password_verify_function		null				#can pass PL/SQL script as argument
	failed_login_attempts			2					#will be locked after n+1 failed attempts
	password_lock_time				3/1440;				#number of days an account will be locked
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
