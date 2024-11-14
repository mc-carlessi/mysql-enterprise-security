# SECURITY - ENTERPRISE FIREWALL

## Introduction
MySQL Enterprise Firewall guards against cyber security threats by providing real-time protection against database specific attacks. Any application that has user-supplied input, such as login and personal information fields is at risk. Database attacks don't just come from applications. Data breaches can come from many sources including SQL virus attacks or from employee misuse. Successful attacks can quickly steal millions of customer records containing personal information, credit card, financial, healthcare or other valuable data.
Objective: Install and use data masking functionalities

Estimated Lab Time: -- 15 minutes

### Objectives

In this lab, you will:
* Install MySQL Enterprise Firewall
* Create a couple of MySQL Enterprise Firewall rules
* Test MySQL Enterprise Firewall

### Prerequisites

This lab assumes you have:
* An Oracle account
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
- The detailed documentation for MySQL Enterprise Firewall is located here:
- https://dev.mysql.com/doc/en/firewall.html

## Task 1: Install Firewall

1. Install MySQL Enterprise Firewall on mysql-advanced using CLI  
    ```
    <span style="color:green">shell-mysql></span><copy>mysqlsh admin@127.0.0.1 --mysql -D mysql < /usr/share/mysql-8.4/linux_install_firewall.sql</copy>
    ```

2. Connect to the instance with administrative account <span style="color:red">first SSH connection - administrative</span>  
    ```
    <span style="color:green">shell-mysql></span><copy>mysqlsh admin@127.0.0.1</copy>
    ```

3. <span style="color:red">Administrative account</span> be sure it has the proper permissions to manage Firewall properties  

    ```
    <span style="color:blue">mysql></span><copy>GRANT FIREWALL_ADMIN, FIREWALL_EXEMPT ON *.* TO 'admin'@'%';</copy>
    ```

    ```
    <span style="color:blue">mysql></span><copy>SHOW GLOBAL VARIABLES LIKE 'mysql_firewall_mode';</copy>
    ```

    ```
    <span style="color:blue">mysql></span><copy>SHOW GLOBAL STATUS LIKE "firewall%";</copy>
    ```

## Task 2: Setup Firewall User and Rules

1. Create user (appuser2) to run Firewall rules and inspect Information Schema tables: 

    ```
    <span style="color:blue">mysql></span><copy>CREATE USER 'appuser2'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

2. Grant proper permissions to sample database for appuser2
    ```
    <span style="color:blue">mysql></span><copy>GRANT ALL ON employees.* TO 'appuser2'@'localhost';</copy>
    ```

3. Create Group Profile name 'fwgrp' and turn on Recording of SQL commands
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_group_mode('fwgrp', 'RECORDING');</copy>
    ```

4. Add appuser2 to Firewall Group just created
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_firewall_group_enlist('fwgrp', 'appuser2@localhost');</copy>
    ```

5. Check environment

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'fwgrp';</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM performance_schema.firewall_membership WHERE GROUP_ID = 'fwgrp' ORDER BY MEMBER_ID;</copy>
    ```
    ```
    <span style="color:blue">mysql></span><copy>SELECT RULE FROM performance_schema.firewall_group_allowlist WHERE NAME = 'fwgrp';</copy>
    ```


## Task 3: Run queries to test Firewall characteristics.

1. Open a connection with <span style="color:red">appuser2</span> account in a separate terminal.

    **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <span style="color:green">shell-mysql></span><copy>mysqlsh appuser2@127.0.0.1</copy>
    ```

2. Run some sample queries that are acceptable

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>USE employees;SELECT emp_no, title, from_date, to_date FROM titles WHERE emp_no = 10001; </copy>
    ```

    ```
    <span style="color:blue">mysql></span><copy>UPDATE titles SET to_date = CURDATE() WHERE emp_no = 10001;</copy>
    ```

    ```
    <span style="color:blue">mysql></span><copy>SELECT emp_no, first_name, last_name, birth_date FROM employees ORDER BY birth_date LIMIT 10;</copy>
    ```

## Task 4: Inspect MySQL Firewall 

1. <span style="color:red">Administrative account</span> Return to admin connection and check firewall status

    a. Check firewall mode for the group 'fwgrp'

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>SELECT MODE FROM performance_schema.firewall_groups WHERE NAME = 'fwgrp';</copy>
    ```

    b. Check rules for 'fwgrp' group

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>SELECT RULE FROM performance_schema.firewall_group_allowlist WHERE NAME = 'fwgrp'\G</copy>
    ```

    c. Check rules counters

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>SHOW GLOBAL STATUS LIKE '%firewall%';</copy>
    ```

    d. Switch to protecting mode

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_group_mode('fwgrp', 'PROTECTING');</copy>
    ```

## Task 5: ReRun queries to test Firewall characteristics.

1. <span style="color:red">appuser2 Connection</span> Return to appuser2 conection

2. Run some sample queries to test firewall

    a. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <span style="color:blue">mysql></span><copy>USE employees;SELECT emp_no, title, from_date, to_date FROM titles WHERE emp_no = 10011; </copy>
    ```

    b. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>SELECT emp_no, title, from_date, to_date FROM titles WHERE emp_no = 10011 OR TRUE; </copy>
    ```

    c. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>SHOW TABLES LIKE '%salaries%';</copy>
    ```

## Task 6: test Firewall in detecting mode

1. <span style="color:red">Administrative Account</span> Return to admin connection

    a. Increase error log verbosity

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <span style="color:blue">mysql></span><copy>SET PERSIST log_error_verbosity=3;</copy>
    ```

    b. Set firewall in detecting mode

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <span style="color:blue">mysql></span><copy>CALL mysql.sp_set_firewall_group_mode('fwgrp', 'DETECTING');</copy>
    ```

    c. Check firewall status

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <span style="color:blue">mysql></span><copy>SHOW GLOBAL STATUS LIKE '%firewall%';</copy>
    ```

2. <span style="color:red">appuser2 Connection</span> Return to appuser2 connection and execute the last query that violates firewall rules

    b. **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <span style="color:blue">mysql></span><copy>USE employees; SELECT emp_no, title, from_date, to_date FROM titles WHERE emp_no = 10011 OR TRUE LIMIT 10; </copy>
    ```

3. <span style="color:red">Administration Connection</span> Return to admin connection and see how the status changed

    a. Check firewall status

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <span style="color:blue">mysql></span><copy>SHOW GLOBAL STATUS LIKE '%firewall%';</copy>
    ```

    b. Check error log content

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <span style="color:blue">mysql></span><copy>SELECT * FROM performance_schema.error_log WHERE ERROR_CODE='MY-011191'\G</copy>
    ```


This **end our workshop**


## Learn More

*(optional - include links to docs, white papers, blogs, etc)*

* [Firewall Docs](https://dev.mysql.com/doc/refman/8.4/en/firewall.html)
* [Enterprise Firewall with Drupal](https://dev.mysql.com/blog-archive/group-profiles-in-mysql-enterprise-firewall/)

## Acknowledgements

* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
