# Oracle APEX Upgrade Guide

## Step 1: Backup Old APEX Folder
Move the old `apex` folder to `apex_bkp`.

## Step 2: Download New APEX
Download the latest APEX from [Oracle APEX Downloads](https://www.oracle.com/tools/downloads/apex-downloads/).

## Step 3: Unzip and Navigate
Unzip the new APEX package and go to the `apex` folder.

## Step 4: Login as SYSDBA
Login as `sys` with `sysdba` privileges from the new `apex` folder.

## Step 5: Run Upgrade SQL Scripts
Run the following SQL scripts:

```sql
@apexins1.sql apex apex temp /i/
@apexins2.sql apex apex temp /i/
-- Now stop ORDS or Tomcat
@apexins3.sql apex apex temp /i/
```

Then, execute the following PL/SQL block (change your APEX user name as needed):

```sql
BEGIN
  DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
    host => '*',
    ace => xs$ace_type(
      privilege_list => xs$name_list('connect'),
      principal_name => 'APEX_220200',
      principal_type => xs_acl.ptype_db
    )
  );
END;
/
```

## Step 6: Replace Images Folder
Find the images folder location (check Tomcat server config) and replace it with the new images folder.

## Step 7: Validate ORDS Connection
Go to the `ords` folder and validate the ORDS connection:

```sh
java -jar ords.war validate
```

## Step 8: Start Apache Tomcat
Start Apache Tomcat and check apex version.


