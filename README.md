# Setup Oracle Database

1. Załóż i aktywuj konto na stronie: [container-registry.oracle](https://container-registry.oracle.com/ords/f?p=113:10::::::)
Zaloguj się na stronie i zaakceptuj licencję dla obrazu database/enterprise.


2. Następnie otwórz terminal i zaloguj się w Dockerze do rejestru Oracle:
    
    ```docker login container-registry.oracle.com```


3. Pobierz obraz bazy danych Oracle (Enterprise Edition):
    
    ```docker pull container-registry.oracle.com/database/enterprise:latest```


4. Sklonuj repozytorium i uruchom kontener za pomocą docker-compose:
    
    ```git clone git@github.com:trissve/nosql_lab.git```
    
    ```cd nosql_lab```
    
    ```docker-compose up -d```


5. Po uruchomieniu kontenera, wejdź do jego terminala i uruchom klienta sqlplus jako użytkownik systemowy (SYSDBA):

    ```docker exec -it oracle-db bash``` 
    
    ```sqlplus / as sysdba```


6. PDB to "pluggable database" – odseparowana baza danych w ramach jednej instancji Oracle. PDB działa wewnątrz CDB, czyli głównej kontenerowej bazy danych - to centralna instancja zarządzająca zasobami. Każda PDB może mieć własnych użytkowników, tabele, dane itd. Umożliwia to tworzenie środowisk testowych i izolację danych. 

    Stwórz nową PDB, razem z jej adminem oraz określ ścieżkę, gdzie będą zapisywane jej dane - zazwyczaj korzysta się już z "bazowej", służącej jako szablon wbudowanej PDB - pdbseed :

    ```
    CREATE PLUGGABLE DATABASE my_pdb
    ADMIN USER pdbadmin IDENTIFIED BY pdbadmin
    FILE_NAME_CONVERT = ('/pdbseed/', '/my_pdb/');
    ```

    Otwórz nowo utworzoną PDB:
    ```
    ALTER PLUGGABLE DATABASE my_pdb OPEN;
    ALTER PLUGGABLE DATABASE my_pdb SAVE STATE;
    ```

    Przełącz kontekst na PDB i nadaj adminowi wszystkie prawa do zarządzania nią:
    ```
    ALTER SESSION SET CONTAINER=my_pdb;
    GRANT DBA TO pdbadmin;
    ```

7. Stwórz dedykowaną przestrzeń na dane, uzytkownika i nadaj mu uprawnienia.

    ```
    CREATE TABLESPACE USERS
    DATAFILE '/opt/oracle/oradata/ORCLCDB/my_pdb/myspace01.dbf'
    SIZE 100M AUTOEXTEND ON NEXT 10M MAXSIZE UNLIMITED;

    CREATE USER username
    IDENTIFIED BY userpasswd
    DEFAULT TABLESPACE USERS
    TEMPORARY TABLESPACE TEMP
    QUOTA UNLIMITED ON USERS;

    GRANT CREATE SESSION, CREATE TABLE, CREATE SEQUENCE, CREATE VIEW, CREATE PROCEDURE TO username;
    GRANT DBA TO username;
    GRANT SELECT ON v_$session TO username;
    ```

8. Możesz teraz połączyć się PDB za pomocą dowolnego klienta Oracle (np. DataGrip). Dane połączenia:  
    Host: localhost  
    Port: 1521  
    Użytkownik: username  
    Hasło: userpasswd  
    Service name: my_pdb


