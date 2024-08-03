# SECURITY - MYSQL ENTERPRISE TRANSPARENT DATA ENCRYPTION

## Introduction
3c) MySQL Enterprise Transparent Data Encryption
Objective: Data Encryption in action…

This lab will walk you through encrypting InnoDB Tablespace files at rest

Estimated Lab Time: 20 minutes

### Objectives

In this lab, you will:
* Install and encrypt Data Files

### Prerequisites (Optional)

This lab assumes you have:

* All previous labs successfully completed

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell

**Notes:**
    - [InnoDB Data At Rest](https://dev.mysql.com/doc/en/innodb-data-encryption.html)

## Task 1: Install and setup TDE  

1. Install MySQL Enterprise Transparent Data Encrytption on mysql-enterprise using <span style="color:red">Administrative Account</span> MySQL client connections 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -u root -pWelcome1! -P3306 -h127.0.0.1 </copy>
    ```
2. Check to see if any keyring plugin is installed and load if not:

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME LIKE 'keyring%'; </copy>
    ```

    b. Edit the my.cnf setting in /etc/my.cnf

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo nano /etc/my.cnf</copy>
    ```

    b. Add the following lines to load the plugin and set the encrypted key file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>early-plugin-load=keyring_encrypted_file.so    
    keyring_encrypted_file_data=/var/lib/mysql-keyring/keyring-encrypted    
    keyring_encrypted_file_password=V&rySec4eT</copy>    
    ```

    c. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```

3. "Spy" on employees.employees table

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```


4. Now with <span style="color:red">Administrative Account</span> we enable Encryption on the employees.employees table:

    a.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -u root -pWelcome1! -P3306 -h127.0.0.1 </copy>
    ```

    b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
    ```
    <copy>USE employees;</copy>
    ```

    c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>ALTER TABLE employees ENCRYPTION = 'Y';</copy>
    ```


5. "Spy" on employees.employees table again:

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```


6. Administrative commands

    a. Get details on encrypted key file:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SHOW VARIABLES LIKE 'keyring_encrypted_file_data'\G</copy>
    ```

    b. Set default for all tables to be encrypted when creating them:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SET GLOBAL default_table_encryption=ON;</copy>
    ```

    c. Peek on the mysql System Tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

    d. Encrypt the mysql System Tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>ALTER TABLESPACE mysql ENCRYPTION = 'Y';</copy>
    ```

    e. Validate encryption of the mysql System Tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

    f. Show all the encrypted tables:
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT SPACE, NAME, SPACE_TYPE, ENCRYPTION FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE ENCRYPTION='Y'\G</copy>
    ```

## Learn More

* [Keyring Plugins](https://dev.mysql.com/doc/en/keyring.html)
* [InnoDB Data At Rest](https://dev.mysql.com/doc/en/innodb-data-encryption.html)

## Acknowledgements

* **Author** - Dale Dasker, MySQL Solution Engineering

* **Last Updated By/Date** - Dale Dasker, January 2023
