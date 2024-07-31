# SECURITY - MYSQL USERS

## Introduction

Users management
Objective: explore user creation and privileges on a Server

*This lab walks you through creating some users which will be used to Audit.

Estimated Time: 10 minutes

### Objectives

In this lab, you will  do the followings:

- Connect to mysql-enterprise
- Create appuser

### Prerequisites

This lab assumes you have:

- All previous labs successfully completed

- Lab standard

  - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
  - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
  - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell

**Notes:**

- Open a notepad file and  your linux Private IP on student###-serverA

- serverA  PRIVATE ip: (client_ip)

## Task 1: Connect to mysql-enterprise on Server

1. Connect to your mysql-enterprise with administrative user

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>mysql -uroot -pWelcome1! -h 127.0.0.1 -P 3306</copy>
    ```

2. Create a new user and restrict the user to your “Server” IP

 a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>CREATE USER 'appuser1'@'127.0.0.1' IDENTIFIED BY 'Welcome1!';</copy>
    ```

 b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>GRANT ALL PRIVILEGES ON employees.* TO 'appuser1'@'127.0.0.1';</copy>
    ```

 c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>SHOW GRANTS FOR 'appuser1'@'127.0.0.1';</copy>
    ```

## Task 2: Add additional users

1. Using the Administrative Connection, create a new user and restrict the user to your “Server” IP

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>CREATE USER 'appuser2'@'127.0.0.1' IDENTIFIED BY 'Welcome1!';</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>GRANT ALL PRIVILEGES ON employees.* TO 'appuser2'@'127.0.0.1';</copy>
    ```

## Task 3: Connect to the Employee App using  `appuser1`@`127.0.0.1`

1. Go to the development folder

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```bash
    <copy>cd /var/www/html</copy>
    ```

2. Replace the username and server  in config.php file with **appuser1** and **127.0.0.1** then save the file.

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```bash
    <copy>sudo nano config.php</copy>
    ```

3. Run the application as follows:

    http://computeIP/emp_apps/list_employees.php

## Task 4: Use appuser1 connection - **** OPTIONAL ****

1. Close and reopen the appuser1 connection for the user, then repeat above commands. There is a difference?

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>exit</copy>
    ```

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>mysql -uroot -pWelcome1! -h 127.0.0.1 -P 3306</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>USE employees;</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>SELECT * FROM employees;</copy>
    ```

2. Switch to the administrative connection revoke ‘USAGE’ privilege using and administrative connection and verify (tip: this privilege can’t be revoked…)

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>REVOKE USAGE ON *.* FROM 'appuser1'@'127.0.0.1';</copy>
    ```
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>SHOW GRANTS FOR 'appuser1'@'127.0.0.1';</copy
    ```

3. Run the application as follows (The application should run):

    http://computeIP/emp_apps/list_employees.php

4. Using the administrative connection revoke all privileges using and administrative connection and verify

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>REVOKE ALL PRIVILEGES ON *.* FROM 'appuser1'@'127.0.0.1';</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>SHOW GRANTS FOR 'appuser1'@'127.0.0.1';</copy>
    ```

5. Close and reopen appuser session, do you see schemas?

6. Run the application as follows (The application will not run):

    http://computeIP/emp_apps/list_employees.php

## Task 5: Restore user privileges  **** OPTIONAL ****

1. Using the administrative connection restore user privileges to reuse it in next 

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

    ```
    <copy>GRANT ALL PRIVILEGES ON employees.* TO 'appuser1'@'127.0.0.1';</copy>
    ```

6. Run the application as follows (The application will run):

    http://computeIP/emp_apps/list_employees.php


You may now **proceed to the next lab**

## Learn More

- [CREATE USER](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)
- [MySQL Access Control Lists](https://dev.mysql.com/doc/refman/8.0/en/access-control.html)

## Acknowledgements

- **Author** - Dale Dasker, MySQL Solution Engineering
- **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
