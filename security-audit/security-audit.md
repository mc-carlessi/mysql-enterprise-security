# SECURITY - MYSQL ENTERPRISE AUDIT

## Introduction

MySQL Enterprise Audit
Objective: Auditing in action…

Estimated Lab Time: 20 minutes

### Objectives

In this lab, you will:

* Setup Audit Log
* Use Audit

### Prerequisites

- All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![green-dot](./images/green-square.jpg) shell>  
  The command must be executed in the Operating System shell
* ![blue-dot](./images/blue-square.jpg) mysql>  
  The command must be executed in a client like MySQL, MySQL Shell or similar tool
* ![yellow-dot](./images/yellow-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell

**Notes:**

* Audit can be activated and configured without stopping the instance. In the lab we edit my.cnf because we set the few non dynamic variables  
* This lab it's easier using 2 shell sessions, one for administrative commands, one for user activities

## Task 1: Setup Audit Log

1. If already connected to MySQL Shell, exit to work with the Operating System shell

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
        <copy>\exit</copy>
    ```

2. Enable Audit Log (this is a MySQL Enterprise only plugin). please note that the script need to be executed with standard protocol, that's the reason for the option '--mc' in MySQL Shell connection

    a. Load Audit functions.  If running in a replicated environment, load the plugin no each of the Replicas first and then modify the SQL script to only load the functions.

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>mysqlsh admin@127.0.0.1 --mysql -D mysql < /usr/share/mysql-8.4/audit_log_filter_linux_install.sql</copy>
    ```
    
    b. Edit the my.cnf setting in /mysql/etc/my.cnf

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>sudo nano /etc/my.cnf</copy>
    ```

    c. Add the following lines to the bottom of the file.  These lines will make sure that the audit plugin can't be unloaded and that the file is automatically rotated at 20 MB and format of data is JSON.

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>
    audit_log=FORCE_PLUS_PERMANENT
    audit_log_rotate_on_size=20971520
    audit_log_format=JSON</copy>
    ```

    d. Because we changed teh my.cnf, we need now to restart MySQL (majority of the audit variables can be changed dynamically, but not 'audit\_log' and 'audit\_log\_format')

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>sudo service mysqld restart</copy>
    ```

3. Connect to your mysql-enterprise with administrative user

    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

4. Verify that the sudit plugin is now loaded

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SHOW PLUGINS;</copy>
    ```

## Task 2: Use Audit - log all activity and all users

1. Using the <span style="color:red">Administrative Account</span> , create a Audit Filter for all activity and all users. Privileges required are AUDIT_ADMIN and SUPER

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');</copy>
    ```

2. Assign the new filter to all users

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT audit_log_filter_set_user('%', 'log_all');</copy>
    ```

3. Return to shell prompt

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <copy>\exit</copy>
    ```

4. Monitor the output of the audit.log file:

    **![green-dot](./images/green-square.jpg) shell>**
    ```
    <copy>sudo tail -f /var/lib/mysql/audit.log</copy>
    ```

5. Run the application as follows (The application should run):

    <http://computeIP/emp_apps/list_employees.php>

6. Go to the Monitor terminal to view the output of the audit.log file
    ![MDS](./images/audit-log.png "audit-log")

7. Return to second SSH connection, or reopen it if you already closed it

    ```
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_compute_instance_ip></copy>
    ```

8. Login to mysql-enterprise with the user <span style="color:red">appuser1 Connection</span>, then submit some commands

    a. **![green-dot](./images/green-square.jpg) shell>**  

    ```
    <copy>mysqlsh appuser1@127.0.0.1</copy>
    ```

    b. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>USE employees;</copy>
    ```

    c. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT * FROM employees limit 10;</copy>
    ```

    d. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT emp_no,salary FROM employees.salaries WHERE salary > 90000 LIMIT 10;</copy>
    ```

9. Go to the Monitor terminal to view the output of the audit.log file

## Task 3: Use Audit - only log connections

1. Let's setup Audit to only log connections. Using the <span style="color:red">Administrative Account</span>, create a Audit Filter for all connections

    a. Connect as administrative user  
    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

    b. Remove  **log_all** filter:  
    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT audit_log_filter_remove_filter('log_all ');</copy>
    ```

    c. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SET @f = '{ "filter": { "class": { "name": "connection" } } }';</copy>
    ```

    d. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT audit_log_filter_set_filter('log_conn_events', @f);</copy>
    ```

    e. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SELECT audit_log_filter_set_user('%', 'log_conn_events');</copy>
    ```
    ```
    <copy>\exit</copy>
    ```

2. Monitor the output of the audit.log file:
    **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>sudo tail -f /var/lib/mysql/audit.log</copy>
    ```


3. Using the second shell, login to mysql-enterprise with the user <span style="color:red">appuser1 Connection</span>, then submit some commands

    a. **![green-dot](./images/green-square.jpg) shell>**

    ```
    <copy>mysqlsh appuser1@127.0.0.1</copy>
    ```

    b. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>USE employees;</copy>
    ```

    c. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT * FROM employees limit 25;</copy>
    ```

    d. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>SELECT emp_no,salary FROM employees.salaries WHERE salary > 90000  limit 10;</copy>
    ```

    e. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**

    ```
    <copy>\exit</copy>
    ```

4. Return to the session with audit log and **check the audit log**: it has now only login and logout messages


## Task 4: Administrative commands for working Audit

1. Some Administrative commands for checking Audit filters and users.  Log in using the <span style="color:red">Administrative Account</span>

   **![green-dot](./images/green-square.jpg) shell>**
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

   a. Check existing filters:

   **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SELECT * FROM mysql.audit_log_filter\G</copy>
    ```

   b. Check Users being Audited:

   **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SELECT * FROM mysql.audit_log_user\G</copy>
    ```

   c. Show filter in human readable format

   **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SELECT JSON_PRETTY(FILTER) as filter FROM mysql.audit_log_filter\G</copy>
    ```

   d. Global Audit log disable

   **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SET GLOBAL audit_log_disable = true;</copy>
    ```

   e. Check that the Audit plugin loaded

   **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME LIKE 'audit%';</copy>
    ```

   e. Close MySQL Shell connection to be ready for next lab
   
   **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>\exit</copy>
    ```

2. You can check the documentation about other Log filters & policies

You may now **proceed to the next lab**
## Learn More

* [Writing Audit Filters](https://dev.mysql.com/doc/en/audit-log-filtering.html)
* [Audit Filter Definitions](https://dev.mysql.com/doc/en/audit-log-filter-definitions.html)

## Acknowledgements

- **Author** - Dale Dasker, MySQL Solution Engineering
- **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
