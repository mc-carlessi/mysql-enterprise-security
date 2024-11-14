# SECURITY - MYSQL ENTERPRISE TRANSPARENT DATA ENCRYPTION

## Introduction

MySQL Enterprise Transparent Data Encryption
Objective: Data Encryption in action…

This lab will walk you through encrypting InnoDB Tablespace files at rest

Estimated Lab Time: 20 minutes

### Objectives

In this lab, you will:

* Setup TDE using file encryption component
* Practice using TDE

### Prerequisites (Optional)

This lab assumes you have:

* All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![green-dot](./images/green-square.jpg) shell>  
  The command must be executed in the Operating System shell
* ![blue-dot](./images/blue-square.jpg) mysql>  
  The command must be executed in a client like MySQL, MySQL Shell or similar tool
* ![yellow-dot](./images/yellow-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell

**Notes:**
    - [InnoDB Data At Rest](https://dev.mysql.com/doc/en/innodb-data-encryption.html)

## Task 1: Setup required files for TDE

1. If nopt already connected, SSH into server instance

2. Retrieve mysqld installation directory, where we later create the global manifest file

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>whereis mysqld</copy>
    ```

3. Retrieve the position of components, where we later create the configuration file

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>mysqlsh admin@127.0.0.1 --table -e 'SELECT @@plugin_dir'</copy>
    ```

4. Now we create the global manifest file

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>cd /usr/sbin </copy>
    ```

    ```
    <copy>sudo nano mysqld.my</copy>
    ```

5. copy the following  content to mysqld.my save and exit

    ```
    <copy>
    {
        "components": "file://component_keyring_encrypted_file"
    }
    </copy>
    ```

6. Create the global configuration file

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>cd /usr/lib64/mysql/plugin/</copy>
    ```

    ```
    <copy>sudo nano component_keyring_encrypted_file.cnf</copy>
    ```

7. copy the following  content to component\_keyring\_encrypted\_file.cnf save and exit

    ```  
    <copy> 
    {
        "path": "/var/lib/mysql-keyring/keyring-encrypted",
        "password": "Welcome1!",
        "read_only": false
    } </copy>
    ```

8. Restart MySQL

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>sudo service mysqld restart</copy>
    ```

## Task 2: Use TDE

1. "Spy" on employees.employees table

    a. **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```

2. Open another terminal window.

    Now with <span style="color:red">Administrative Account</span> we enable Encryption on the employees.employees table:

    a.  **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

    b. Verify the component is loaded and active: 
    
    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT * FROM performance_schema.keyring_component_status;</copy>
    ```

    c. Let's now encrypt the employees table

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>USE employees;</copy>
    ```

    ```
    <copy>ALTER TABLE employees ENCRYPTION = 'Y';</copy>
    ```

3. From a second session, "spy" on employees.employees table again:

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```

4. Administrative commands

    a. Set default for all tables to be encrypted when creating them:
    
    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SET GLOBAL default_table_encryption=ON;</copy>
    ```

    b. Peek on the mysql System Tables:

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

    c. Encrypt the mysql System Tables:

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>ALTER TABLESPACE mysql ENCRYPTION = 'Y';</copy>
    ```

    d. Show all the encrypted tables:

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SELECT SPACE, NAME, SPACE_TYPE, ENCRYPTION FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE ENCRYPTION='Y'\G</copy>
    ```

5. Exit from MySQL Shell

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```

5. Validate encryption of the mysql System Tables:

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

6. Run the application to see if it was affected by TDE:

    <http://computeIP/emp_apps/list_employees.php>

You may now **proceed to the next lab**

## Learn More

* [Keyring Plugins](https://dev.mysql.com/doc/en/keyring.html)
* [InnoDB Data At Rest](https://dev.mysql.com/doc/en/innodb-data-encryption.html)

## Acknowledgements

* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
