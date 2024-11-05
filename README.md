# snowflake_row_level_security
Use row access policies to limit the rows as user can see.

Run the commands below in a Snowflake worksheet to see a Row Access Policy in action.
This simple example limits the departments a user can see base on the Snowflake role they are using.


/*
ROW-LEVEL SECURITY - use row access policies to limit the rows as user can see.

Simple example of Row Level Access Policy.

Includes:
-Object Creation - DB, Schema, Table
-Sample data insert statements
-Role creation
-Assignment of permisions to roles
-Row Access Policy creation and addition
-Test statements to see polify in use
-Clean up/reset of environment

Learn more at - https://docs.snowflake.com/en/user-guide/security-row-intro

*/



---------------------------------
-- Set variables
---------------------------------
SET db_name = 'demo_rla';
SET schema_name = 'raw';
SET myuser_name = 'admin';


---------------------------------
-- Create Database and Schema
---------------------------------
use role sysadmin;
create or replace database IDENTIFIER($db_name);
create or replace schema IDENTIFIER($schema_name);


--------------------------------------
-- Specify Database and Schema to Use
--------------------------------------
use database IDENTIFIER($db_name);
use schema IDENTIFIER($schema_name);


---------------------------------
-- Create table
---------------------------------
create or replace TABLE STUDENTS_MAJOR (
	STUDENT_NAME VARCHAR,
    MAJOR varchar	
);


---------------------------------
-- Insert sample data
---------------------------------
INSERT INTO STUDENTS_MAJOR (STUDENT_NAME, MAJOR) VALUES
('Alice Johnson', 'Engineering'),
('Diana Prince', 'Engineering'),
('Bob Smith', 'Business'),
('Hannah Montana', 'Business'),
('Kevin Bacon', 'Business'),
('Charlie Brown', 'Chemistry'),
('Ian Fleming', 'Chemistry'),
('Anne Bird', 'Chemistry'),
('Mike Smithers', 'Chemistry');


--------------------------------------------------------------------
-- Grant usage on db to securityadmin so they can grant permissions
--------------------------------------------------------------------
grant usage on database IDENTIFIER($db_name) to role securityadmin;
grant usage on schema IDENTIFIER($schema_name) to role securityadmin;
grant select on all tables in schema IDENTIFIER($schema_name) to role securityadmin;


-----------------------------------
-- Create roles, one per department
-----------------------------------
use role useradmin;

create or replace role demo_rla_business_ro;

create or replace role demo_rla_engineering_ro;

create or replace role demo_rla_chemistry_ro;


------------------------------------------------------------------------------------
-- As securityadmin and grant permissions to role demo_rla_business_rw
-- Use the newly created role and observe they can view all 9 records in the tables
------------------------------------------------------------------------------------
SET role_name = 'demo_rla_business_ro';

use role securityadmin;
grant usage on database IDENTIFIER($db_name) to role IDENTIFIER($role_name);
GRANT SELECT ON  STUDENTS_MAJOR  to ROLE IDENTIFIER($role_name);
grant usage on schema IDENTIFIER($schema_name) to ROLE IDENTIFIER($role_name);
grant role IDENTIFIER($role_name) to user IDENTIFIER($myuser_name);

use role IDENTIFIER($role_name);
select * from STUDENTS_MAJOR order by MAJOR;


------------------------------------------------------------------------------------
-- As securityadmin and grant permissions to role demo_rla_engineering_ro
-- Use the newly created role and observe they can view all 9 records in the tables
------------------------------------------------------------------------------------
SET role_name = 'demo_rla_engineering_ro';

use role securityadmin;
grant usage on database IDENTIFIER($db_name) to role IDENTIFIER($role_name);
GRANT SELECT ON  STUDENTS_MAJOR  to ROLE IDENTIFIER($role_name);
grant usage on schema IDENTIFIER($schema_name) to ROLE IDENTIFIER($role_name);
grant role IDENTIFIER($role_name) to user IDENTIFIER($myuser_name);

use role IDENTIFIER($role_name);
select * from STUDENTS_MAJOR order by MAJOR;



------------------------------------------------------------------------------------
-- As securityadmin and grant permissions to role demo_rla_chemistry_ro
-- Use the newly created role and observe they can view all 9 records in the tables
------------------------------------------------------------------------------------
SET role_name = 'demo_rla_chemistry_ro';

use role securityadmin;
grant usage on database IDENTIFIER($db_name) to role IDENTIFIER($role_name);
GRANT SELECT ON  STUDENTS_MAJOR  to ROLE IDENTIFIER($role_name);
grant usage on schema IDENTIFIER($schema_name) to ROLE IDENTIFIER($role_name);
grant role IDENTIFIER($role_name) to user IDENTIFIER($myuser_name);

use role IDENTIFIER($role_name);
select * from STUDENTS_MAJOR order by MAJOR;
 
  
------------------------------------------------
-- Create then add row access policy
-- Note the roles are all specified in UPPER CASE
------------------------------------------------
use role sysadmin;
CREATE OR REPLACE ROW ACCESS POLICY STUDENTS_MAJOR_ROW_ACCESS_POLICY
AS (MAJOR STRING) RETURNS BOOLEAN ->
CASE
    WHEN CURRENT_ROLE() = 'DEMO_RLA_BUSINESS_RO' THEN major = 'Business'
    WHEN CURRENT_ROLE() = 'DEMO_RLA_ENGINEERING_RO' THEN major = 'Engineering'
    WHEN CURRENT_ROLE() = 'DEMO_RLA_CHEMISTRY_RO' THEN major = 'Chemistry'
    WHEN CURRENT_ROLE() = 'ACCOUNTADMIN' THEN TRUE
    ELSE FALSE
END;


ALTER TABLE STUDENTS_MAJOR
ADD ROW ACCESS POLICY STUDENTS_MAJOR_ROW_ACCESS_POLICY ON (MAJOR);

--ALTER TABLE STUDENTS_MAJOR
-- drop ROW ACCESS POLICY STUDENTS_MAJOR_ROW_ACCESS_POLICY;


------------------------------------------------
-- Test row access policy with various roles
-- Note the MAJOR returned and number of records
------------------------------------------------

--  See only Business records
------------------------------
use role demo_rla_business_ro;
select * from STUDENTS_MAJOR order by MAJOR;

--  Sees only Engineering records 
--------------------------------
use role demo_rla_engineering_ro;
select * from STUDENTS_MAJOR order by MAJOR;

--  Sees only Chemistry records
-------------------------------
use role demo_rla_chemistry_ro;
select * from STUDENTS_MAJOR order by MAJOR;

--  Sees all MAJORS and records
-------------------------------
use role accountadmin;
select * from STUDENTS_MAJOR order by MAJOR;


------------------------------------------------
-- Reset the environment
------------------------------------------------

use role useradmin;
drop role demo_rla_business_ro;
drop role demo_rla_engineering_ro;
drop role demo_rla_chemistry_ro;

use role sysadmin;
drop database IDENTIFIER($db_name);

