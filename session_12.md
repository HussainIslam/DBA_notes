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