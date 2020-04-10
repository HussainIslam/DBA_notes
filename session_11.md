# Session 11 - Granting/Revoking System/Object Privilages

## Important tables
dba_sys_privs 			all the system privilages
dba_tab_privs 			table privilages

## Query number of system privilages
select count(\*) from dba_sys_privilates;  	//the * is escaped to italizing in .md file

## Query the privilages of users
select *
from dba_sys_privs
where grantee in ('TOM','JANE','FOOBAR');

## Give create tablespace and provide a role
grant create tablespace, select_catalog_role to tom;
grant create table, resource to jane;

## Give a user privilage to create table in other user and make others dba
grant create any table to TOM 			//tom can create a table in any user's account
with admin option;						//tom can make any other user an admin

## Take away a privilage
revoke create any table from tom;

## Give a user some object privilage (can be done by DBA or owner of object)
grant insert, select 					//granting insert and select privilages
on scott.dept 							//target object will be scott.dept
to tom 									//grantee is tom
with grant option; 						//tom can grant privilages to others

## Query table privilages
select grantee, privilege, grantable, grantor
from dba_tab_privs
where owner='SCOTT' and table_name='DEPT';

## Revoke object privilage from user
revoke select 
on scott.dept
from tom;

## NOTE: 
1. Revoking system privilege from the user in the middle of the user chain will NOT cascade and break the chain. The user at the end of the chain will RETAIN the privilege.
2. Revoking the object privilege from the user in the middle of the user chain will cascade and break the chain. The user at the end of the chain will LOSE the privilege.