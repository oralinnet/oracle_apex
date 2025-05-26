
## Prerequisites

Before you begin, ensure you have the following:

- **Oracle Database** (already installed and running)
- **Oracle SYSDBA access** (for running installation scripts)
- **APEX 22.2 ZIP file** (downloaded from Oracle)
- **ORDS 22.x ZIP file** (downloaded from Oracle)
- **Apache Tomcat 9.x** (downloaded from [Tomcat website](https://tomcat.apache.org/))
- **JDK 20** (downloaded from [Oracle website](https://www.oracle.com/java/technologies/javase/jdk20-archive-downloads.html))
- **Linux server** (commands assume a Linux environment; adjust paths for your OS)
- **Sufficient disk space** for tablespaces and software
- **Firewall access** to open required ports (e.g., 8080)
- **Basic command-line skills** (for running shell and SQL commands)
- **(Optional) Nginx** if you plan to use a reverse proxy

## Step 1: Create APEX Tablespace

```sh
sqlplus / as sysdba
create tablespace apex datafile '/u01/app/oracle/oradata/CLOUDDB/oradev/apex01.dbf' size 3G autoextend on next 100M;
```

---

## Step 2: Download and Install APEX 22.2

1. **Unzip APEX:**
    ```sh
    unzip apex_22.2.zip
    cd apex
    ```

2. **Login as SYSDBA:**
    ```sh
    sqlplus 'sys/xxxxxxx@oradev' as sysdba
    ```

3. **Run Installation Scripts:**
    ```sql
    @apexins apex apex temp /i/
    @apxchpwd.sql
    ALTER USER ANONYMOUS ACCOUNT UNLOCK;
    ALTER USER XDB ACCOUNT UNLOCK;
    ALTER USER APEX_220200 ACCOUNT UNLOCK; 
    ALTER USER FLOWS_FILES ACCOUNT UNLOCK;
    ALTER USER APEX_PUBLIC_USER ACCOUNT UNLOCK;
    @apex_rest_config.sql
    ```

4. **Set Passwords:**
    ```sql
    ALTER USER APEX_LISTENER  ACCOUNT UNLOCK identified by passwod;
    ALTER USER APEX_PUBLIC_USER ACCOUNT UNLOCK identified by passwod;
    ALTER USER APEX_REST_PUBLIC_USER ACCOUNT UNLOCK identified by passwod;
    ALTER USER APEX_INSTANCE_ADMIN_USER ACCOUNT UNLOCK identified by passwod;
    ALTER USER APEX_220200 ACCOUNT UNLOCK identified by passwod;
    ```

5. **Configure Network ACLs:**
    ```sql
    BEGIN
      DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host => '*',
        ace => xs$ace_type(privilege_list => xs$name_list('connect'),
        principal_name => 'APEX_220200',
        principal_type => xs_acl.ptype_db)
      );
    END;
    /

    BEGIN
      DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host => 'localhost',
        ace => xs$ace_type(privilege_list => xs$name_list('connect'),
        principal_name => 'APEX_220200',
        principal_type => xs_acl.ptype_db)
      );
    END;
    /

    EXEC DBMS_XDB.sethttpport(0);
    ```

---

## Step 3: Install Tomcat

1. **Download Tomcat and JDK**

2. **Install JDK:**
    ```sh
    rpm -ivh jdk-20_linux-x64_bin.rpm
    ```

3. **Set JAVA_HOME:**
    ```sh
    vim /root/.bash_profile
    # Add:
    JAVA_HOME=/usr/java/jdk-20;export JAVA_HOME
    PATH=$JAVA_HOME/bin:$PATH;
    # Save and exit (:x)
    ```

4. **Extract Tomcat:**
    ```sh
    tar -zxvf apache-tomcat-9.0.26.tar.gz
    ```

5. **Start/Stop Tomcat:**
    ```sh
    /u01/oradev/tomcat/apache-tomcat-9.0.26/bin/startup.sh
    /u01/oradev/tomcat/apache-tomcat-9.0.26/bin/shutdown.sh
    ```

6. **Configure Tomcat:**
    ```sh
    vim /u01/oradev/tomcat/apache-tomcat-9.0.26/conf/server.xml
    ```

7. **Open Tomcat Port in Firewall:**
    ```sh
    firewall-cmd --permanent --add-port=8080/tcp
    firewall-cmd --reload
    ```

---

## Step 4: Install ORDS

1. **Unzip ORDS:**
    ```sh
    mkdir /u01/oradev/ords
    unzip ords-22.4.4.041.1526.zip
    ```

2. **Create Directories:**
    ```sh
    mkdir -p /u01/oradev/ords/config
    mkdir -p /u01/oradev/ords/config/logs
    ```

3. **Set Environment Variables:**
    ```sh
    export ORDS_HOME=/u01/oradev/ords
    export ORDS_CONFIG=/u01/oradev/ords/config
    export ORDS_LOGS=${ORDS_CONFIG}/logs
    ```

4. **Install ORDS:**
    ```sh
    ${ORDS_HOME}/bin/ords --config ${ORDS_CONFIG} install
    ```

    - **Follow the prompts:**
        - Choose installation type: `2`
        - Database connection type: `1`
        - Host: `localhost`
        - Port: `8804`
        - Service name: `oradev`
        - Admin username: `sys as sysdba`
        - Default tablespace: `apex`
        - Temporary tablespace: `TEMP`
        - Features: `1`
        - Standalone mode: `2`

---

## Step 5: Deploy ORDS to Tomcat

1. **Copy `ords.war` to Tomcat webapps:**
    ```sh
    cp ords.war ../tomcat/apache-tomcat-9.0.26/webapps/
    ```

2. **Configure Tomcat to use ORDS config:**
    - Edit `catalina.sh` in Tomcat's `bin` directory:
        ```sh
        # Add this line:
        JAVA_OPTS="-Dconfig.url=/u01/oradev/ords/config"
        ```

3. **Configure APEX Images:**
    - Edit `server.xml` in Tomcat's `conf` directory.
    - In the `<Host ...>` section, add:
        ```xml
        <Context docBase="/u01/oradev/apex/images" path="/i" />
        ```

4. **Restart Tomcat.**

---

## Step 6: Troubleshooting

- **Error:** `ORA-01035: ORACLE only available to users with RESTRICTED SESSION privilege`
    ```sql
    grant RESTRICTED SESSION to APEX_PUBLIC_USER;
    grant RESTRICTED SESSION to ORDS_PUBLIC_USER;
    ```

- **Proxy Error (if using nginx):**
    ```sql
    alter user BEH grant connect through APEX_REST_PUBLIC_USER;
    ```

- **Restart Tomcat server after making changes.**

---

## Notes

- Replace passwords and sensitive information as needed.
- Adjust paths for your environment.