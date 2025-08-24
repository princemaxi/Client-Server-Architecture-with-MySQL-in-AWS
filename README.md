# 🚀 Deploying a Client–Server Architecture with MySQL

# 📑 Table of Contents  

- [🚀 Deploying a Client–Server Architecture with MySQL](#-deploying-a-clientserver-architecture-with-mysql)
- [📑 Table of Contents](#-table-of-contents)
  - [Introduction](#introduction)
  - [Components](#components)
    - [1. Server Side](#1-server-side)
    - [2. Client Side](#2-client-side)
  - [How Client–Server Works in MySQL](#how-clientserver-works-in-mysql)
  - [Architecture Diagram](#architecture-diagram)
  - [🛠 Prerequisites](#-prerequisites)
  - [🚀 Deployment Guide](#-deployment-guide)
    - [Set Up the MySQL Server](#set-up-the-mysql-server)
    - [Configure Firewall / Security Groups](#configure-firewall--security-groups)
    - [Setup and Connect From the Client](#setup-and-connect-from-the-client)
  - [Business and Technical Use Cases](#business-and-technical-use-cases)
    - [Business Use Cases](#business-use-cases)
    - [Technical Use Cases](#technical-use-cases)
  - [Benefits of Client–Server Architecture with MySQL](#benefits-of-clientserver-architecture-with-mysql)

## Introduction

A client–server architecture is a distributed application framework that divides tasks between providers of a resource or service, called servers, and service requesters, called clients.

In the context of MySQL, the MySQL server provides database services (storage, query processing, transaction handling), while clients (applications, users, or other services) connect to it to request data or perform operations.

## Components
### 1. Server Side

- **MySQL Server (mysqld):** The main database server process that manages databases, handles SQL queries, enforces security, and ensures concurrency.

- **Database (Schema):** Logical collections of tables, views, indexes, and stored procedures.

### 2. Client Side

- **Applications:** Web apps, desktop apps, scripts, or command-line clients that query the database.

- **MySQL Client Utilities:** Such as mysql CLI, GUI tools (MySQL Workbench, phpMyAdmin), or custom applications via libraries (JDBC, ODBC, Python connectors).

## How Client–Server Works in MySQL

1. **Client Connection:** A client (user or application) sends a connection request to the MySQL server using IP address/hostname, port 3306 (default), username, and password.
  
    Example: `mysql -h 192.168.1.10 -u dbuser -p`

2. **Authentication:** The MySQL server validates credentials against its user privilege system.

3. **Request Processing:** 

    - The client sends SQL queries (e.g., SELECT * FROM users;).

    - The server parses, optimizes, and executes the query.

4. **Response:** The server returns the result set or error messages to the client.

5. **Disconnection:** The session ends when the client disconnects, releasing server resources.

## Architecture Diagram

```pgsql
+-----------------+        Query / Request        +-----------------+
|   MySQL Client  | ----------------------------> |   MySQL Server  |
| (App/User/CLI)  | <---------------------------- |    (mysqld)     |
+-----------------+        Response / Data        +-----------------+
          |
          |
  Libraries / Drivers
   (JDBC, ODBC, Python,
    PHP MySQLi, etc.)
```

## 🛠 Prerequisites

- In AWS Management Console create and configure 2 Linux based Ec2 instances
    ```
    Ec2 1: Mysql Server
    Ec2 2: Mysql Client
    ```

    ![Ec2 Image](/images/ec2a.png)

- SSH into the servers

    ![SSH Image](/images/ssh1.png)
    ![SSH Image](/images/ssh2.png)

## 🚀 Deployment Guide

### Set Up the MySQL Server

- **In Mysql server session, run the cmd below and Configure root password and basic security.:**
    ```bash
    # Update system
    sudo apt update && sudo apt upgrade -y

    # Install MySQL server
    sudo apt install mysql-server -y

    # Secure the installation
    sudo mysql_secure_installation
    ```

    ![Server Image](/images/mysqlserve1.png)

- **Configure MySQL for Remote Access (Server-Side):** By default, MySQL only listens on localhost (127.0.0.1).

    1. Edit the config file:
        ```bash
        sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
        ```
        ![Server Image](/images/mysqlserve2.png)
    2. Look for:
        ```ini
        bind-address = 127.0.0.1
        ```
        ![Server Image](/images/mysqlserve3a.png)
    3. 👉 Change it to allow external clients:
        ```ini
        bind-address = 0.0.0.0
        ```
        ![Server Image](/images/mysqlserve3b.png)
    4. Save and close 

- **Restart MySQL:**
    ```bash
    sudo systemctl restart mysql
    ```
    ![Server Image](/images/mysqlserve4.png)

- **Create a Database & User**

    1. **Login as root**
        ```bash
        sudo mysql -u root -p
        ```
        ![Server Image](/images/mysqlserve5.png)
    
    2. **Create database & a user with remote access, Grant permissions and Apply changes.**

        ```sql
        CREATE DATABASE clientserver_db;
        CREATE USER 'client_user'@'%' IDENTIFIED BY 'Strong.Password123';
        GRANT ALL PRIVILEGES ON clientserver_db.* TO 'client_user'@'%';
        FLUSH PRIVILEGES;
        exit;
        ```
        ![Server Image](/images/mysqlserversetup.png)
    3. Verify Databases
        ```sql
        SHOW databases;
        ```

- **Create a Table and insert sample data in the Database created `clientserver_db`**

    - ✅ Step 1: Switch to your database
        ```sql
        USE clientserver_db;
        ``` 
    - ✅ Step 2: Create a table, Here’s an employees table:
        ```sql
        CREATE TABLE employees (
            id INT PRIMARY KEY AUTO_INCREMENT,
            first_name VARCHAR(50) NOT NULL,
            last_name VARCHAR(50) NOT NULL,
            email VARCHAR(100) UNIQUE NOT NULL,
            department VARCHAR(50),
            salary DECIMAL(10,2)
        );
        ```
    - ✅ Step 3: Insert 10 sample rows:
        ```sql
        INSERT INTO employees (first_name, last_name, email, department, salary) VALUES
        ('John', 'Doe', 'john.doe@example.com', 'IT', 60000.00),
        ('Jane', 'Smith', 'jane.smith@example.com', 'HR', 50000.00),
        ('Michael', 'Brown', 'michael.brown@example.com', 'Finance', 70000.00),
        ('Emily', 'Johnson', 'emily.johnson@example.com', 'Marketing', 55000.00),
        ('Daniel', 'Williams', 'daniel.williams@example.com', 'Operations', 65000.00),
        ('Sarah', 'Davis', 'sarah.davis@example.com', 'IT', 62000.00),
        ('David', 'Miller', 'david.miller@example.com', 'Finance', 72000.00),
        ('Sophia', 'Wilson', 'sophia.wilson@example.com', 'HR', 48000.00),
        ('James', 'Taylor', 'james.taylor@example.com', 'IT', 64000.00),
        ('Olivia', 'Anderson', 'olivia.anderson@example.com', 'Marketing', 56000.00);
        ```
        ![Server Image](/images/table.png)
    - ✅ Step 4: Verify data
        ```sql
        SELECT * FROM employees;
        ```
        ![Server Image](/images/table0.png)

---

### Configure Firewall / Security Groups
In AWS Management Console, select Mysql Server EC2:

- Go to security group / firewall settings.

- Allow port 3306 inbound from your client’s IP (for security, don’t open to 0.0.0.0/0 unless testing).

![Server Image](/images/inbound.png)

---

### Setup and Connect From the Client

- **From the client machine `Mysql Client`, install MySQL client:**

    ```bash
    sudo apt install mysql-client -y
    ```
    ![Server Image](/images/mysqlclient1.png)

- **Connect to the Mysql Server:**
    ```bash
    mysql -h <SERVER_IP> -u client_user -p
    ```
    ![Server Image](/images/connected.png)    
You should now reach the MySQL server remotely.

- **Test for Connectivity using:**
    - **Ping:**
    ```bash
    ping <mysql server private IP>  
    ```
    ![Server Image](/images/network1.png) 
    - **Telnet:**
    ```bash
    telnet <mysql server private IP> 3306  
    ```
    ![Server Image](/images/network2.png)  

- **Querry for the Database we created earlier on Mysql Server from this Client side:**

    ```sql
    USE clientserver_db;
    SELECT * FROM employees;
    ```
    ![Server Image](/images/table2.png)

_We have successfuly completed our deployment as our client can make requests from our server_

## Business and Technical Use Cases

### Business Use Cases

1. **E-commerce Platforms**

    - Scenario: Online retailers (e.g., Amazon, Jumia) use MySQL to store customer profiles, product catalogs, order histories, and payment records.

    - Why MySQL? Reliable transaction support (ACID compliance), scalability, and high availability for millions of concurrent users.

2. **Banking and Financial Services**

    - Scenario: Banks use MySQL for managing customer accounts, transactions, loan applications, and reporting dashboards.

    - Why MySQL? Ensures data consistency, supports complex queries, and provides security for sensitive financial data.

3. **Healthcare Management Systems**

    - Scenario: Hospitals use MySQL to store patient records, prescriptions, appointment scheduling, and billing information.

    - Why MySQL? Enables fast access to structured medical data while ensuring compliance with data regulations.

4. **Education Systems (Learning Management Systems)**

    - Scenario: Universities and e-learning platforms use MySQL to manage student records, grades, and course content.

    - Why MySQL? Lightweight, scalable, and cost-effective for both small and large institutions.

5. **Logistics and Transportation**

    - Scenario: Fleet management systems use MySQL to track shipments, vehicles, fuel consumption, and route optimization.

    - Why MySQL? Efficient query handling and integration with real-time tracking applications.

### Technical Use Cases

1. **Web Application Backends**

    - MySQL acts as the primary data store for dynamic websites (e.g., blogs, forums, SaaS apps).

    - Integrated with application servers like Node.js, Python (Django/Flask), or PHP (Laravel).

2. **Data Warehousing and Reporting**

    - MySQL supports ETL pipelines where transactional data is pulled from operational databases into analytics systems.

    - Used with BI tools (e.g., Tableau, Power BI) for real-time dashboards.

3. **Microservices Architecture**

    - Each microservice can have its own dedicated MySQL database, ensuring data isolation and better scalability.

    - Example: An e-commerce app might have separate databases for payments, orders, and inventory.

4. **Replication and High Availability Systems**

    - Master-Slave Replication in MySQL allows read scalability by directing queries to replicas.

    - Failover and Backup solutions ensure uptime and disaster recovery.

5. **IoT Data Storage**

    - MySQL can be used as a backend database for IoT platforms to store telemetry data from sensors and devices.

    - Example: Smart home applications storing temperature, motion, or energy usage data.

6. **API Backends**

    - REST or GraphQL APIs rely on MySQL databases for structured storage of user and application data.

    - Ensures consistent data retrieval across mobile, web, and desktop clients.

## Benefits of Client–Server Architecture with MySQL

- **Scalability:** Multiple clients can connect to the same server concurrently.

- **Centralized Management:** Data stored and managed in one place.

- **Security Control:** Centralized authentication and authorization.

- **Flexibility:** Clients can be written in multiple languages using MySQL connectors.