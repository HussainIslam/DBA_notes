# Session 13 - Adding/Viewing Table Constraints

## Important data dictionaries
dba_constraints				all constraint definitions on all tables in the database
dba_cons_columns			all columns in the database that are specified in constraints
dba_indexes					all indexes in the database

## Give an user quota on a tablespace
alter user tom
quota 1m
on indx;

## Give browse data dictionary role
grant select_catalog_role to tom;

## Query constraint information
select 		constraint_name, constraint_type, search_condition, 
			status, deferrable, deferred, validated, table_name
from 		dba_constraints
where 		table_name in ('CUSTOMERS','ORDERS','PRODUCTS')
and 		owner='TOM'
order by 	8, 2, 1;

## Query columns names that have constraints
select constraint_name, column_name, position, table_name
from dba_const_column
where owner = 'TOM'
and table_name in ('CUSTOMERS','ORDERS','PRODUCTS')
order by 4,1;

## Query what indexes are created by server
select index_name, index_type, uniqueness
from dba_indexes
where index_name in        (select constraint_name
                           from dba_constraints
                           where owner='TOM'
                           and table_name in ('CUSTOMERS','ORDER','PRODUCTS'));


## Apply constraints and export exceptions
alter table products
enable constraint prod_uk
exceptions into exceptions;

## Enable NOVALIDATE
alter table products
enable novalidate
constraint prod_uk;

## Update a record
update products
set prod_code=2315
where rowid = 'AAAXmbAAIAAAAArAAB';

## Truncate
truncate table exceptions;

## Query constraints of a table of a users
select constraint_name, owner, constraint_type, status, deferrable, deferred
from dba_constraints
where table_name='ORDERS'
and owner='TOM';

## Defer a constraint
set constraint ord_cc_fk deferred;			//deferred: 	data will be validated when committing
set constraint ord_cc_fk immediate;			//immediate:	data will be validated when committing 


## Notes
1. Index created by server are always created implicitly when developers specify either pk or UK constraints (but not if disabled) and they will be unique (unless created as DEFERRABLE, then they will be NONUNIQUE).
2. A NOVALIDATE constraint is basically a constraint which can be enabled but for which Oracle will not check the existing data to determine whether there might be data that currently violates the constraint.
3. FK constraint is created as DEFRRABLE and IMMEDIATE (sub-default mode) and it will perform line by line check (like NOT DEFERRABLE one )  it will ignore "bad" rows, but will process all other  rows that have no errors.  Later, you may switch this constraint to DEFERRABLE sub-mode and then it will perform only one check at Commit time.