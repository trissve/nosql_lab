# Setup Oracle Database

1. You need to have an activated account on container-registry.oracle. Next, you need to auth yourself and pull the db image:

```docker login container-registry.oracle.com```

```docker pull container-registry.oracle.com/database/enterprise:latest```


2. Cloning repository
```git clone git@github.com:trissve/nosql_lab.git```

```cd nosql_lab```

```docker-compose up -d```


3. Next, you need to enter the contener, login as SYSDBA in order to create new PDB and admin user.
```docker exec -it oracle-db bash``` 

```sqlplus / as sysdba```


4.  ```
    CREATE PLUGGABLE DATABASE my_pdb
    ADMIN USER pdbadmin IDENTIFIED BY pdbadmin
    FILE_NAME_CONVERT = ('/pdbseed/', '/my_pdb/');

    ALTER PLUGGABLE DATABASE my_pdb OPEN;
    ALTER PLUGGABLE DATABASE my_pdb SAVE STATE;

    ALTER SESSION SET CONTAINER=nosql_db;
    GRANT DBA TO admin;
    ```

5. Next, we need to create a dedicated tablespace, and user in new PDB.

    ```
    CREATE TABLESPACE USERS
    DATAFILE '/opt/oracle/oradata/ORCLCDB/my_pdb/myspace01.dbf'
    SIZE 100M AUTOEXTEND ON NEXT 10M MAXSIZE UNLIMITED;

    CREATE USER ewa
    IDENTIFIED BY ewa
    DEFAULT TABLESPACE USERS
    TEMPORARY TABLESPACE TEMP
    QUOTA UNLIMITED ON USERS;

    GRANT CREATE SESSION, CREATE TABLE, CREATE SEQUENCE, CREATE VIEW, CREATE PROCEDURE TO ewa;
    GRANT DBA TO ewa;
    GRANT SELECT ON v_$session TO ewa;
    ```

6. Now you can connect to newly created PDB, using PDB name as Service name, and users name and password.


