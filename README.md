# Three-Tier Project Deployment on AWS.

## Introduction
 This project presents a **Java-based Student Registration Web Application** deployed using a **Three-Tier Architecture on Amazon Web Services (AWS)**. The infrastructure features an **NGINX Proxy Server** within the public subnet, an **Apache Tomcat Application Server** in a private subnet, and a **MariaDB Database** hosted on **Amazon RDS**. The **Proxy Server** not only manages incoming HTTP requests but also functions as a **Bastion Host (Jump Server)** to enable secure SSH access to the private-tier instances.
The entire environment is deployed within a **custom AWS VPC, utilizing subnets, route tables, a NAT gateway, and security groups** to ensure controlled and secure communication between layers. This setup effectively demonstrates how user requests travel through the proxy to the application and database layers, modeling a real-world, scalable, and secure cloud architecture.

## Architecture Overview

The architecture consists of:

- **VPC (10.0.0.0/16)** – Custom Virtual Private Cloud hosting all resources.
- **Public Subnet (10.0.0.0/20)** – Hosts the Proxy (Web) Server.
- **Private Subnet 1 (10.0.16.0/20)** – Hosts the Application Server.
- **Private Subnet 2 (10.0.32.0/20)** – Hosts the Database (Amazon RDS).
- **NAT Gateway**: In Public Subnet with Elastic IP
- **Security Group Ports**:
  - 22 (SSH)
  - 80 (HTTP)
  - 8080 (Java Application)
  - 3306 (MySQL)

![Three Tier Architecture](picture/Three-Tier-Project.drawio.png)

## Tech Stack

| Tier | Component | Technology Used |
|------|------------|-----------------|
| Presentation | Proxy / Web Server | Nginx / Apache / React / HTML |
| Application | App Server | Node.js / Flask / Spring Boot |
| Database | Database Server | Amazon RDS (MySQL / PostgreSQL) |
| Cloud Infrastructure | AWS Services | VPC, EC2, RDS, NAT Gateway, Route Tables, IGW, Subnets |

## Deployment Steps
## PART 1: Create Networking Resources
### Step 1: Create VPC
- Name tag: three-tier-vpc
- IPv4 CIDR block: 10.0.0.0/16
- Tenancy: Default

![Three Tier Architecture](picture/1.png)
### Step 2: Create Subnet
- Public Subnet: 10.0.0.0/20 (for Proxy Server)
- Private Subnet 1: 10.0.16.0/20 (for App Server)
- Private Subnet 2: 10.0.32.0/20 (for DB Server)

![Three Tier Architecture](picture/2.png)

![Three Tier Architecture](picture/3.png)

![Three Tier Architecture](picture/4.png)

![Three Tier Architecture](picture/5.png)

### Step 3: Create Internet Gateway
- In the VPC Dashboard, select Internet Gateways and click “Create Internet Gateway.”
  - Name tag: three-tier-igw
- After creation, choose the new Internet Gateway and click “Attach to VPC.”
  - Select your previously created VPC (three-tier-vpc).

![Three Tier Architecture](picture/7.png)

![Three Tier Architecture](picture/8.png)

### Step 4: Configure Public Route Table
Update Public Route Table to add an IGW route.
![Three Tier Architecture](picture/9.png)

### Step 5: Create NAT Gateway         
Configure the NAT Gateway as follows:
  - Name tag: three-tier-nat-gateway
  - Subnet: Select the Public Subnet (where the Proxy Server resides).
  - Elastic IP allocation: Choose “Allocate Elastic IP automatically.”

![Three Tier Architecture](picture/11.png)

### Step 6: Create Private Route table inside your VPC and add route of NAT gateway
Edit Subnet Association
![Three Tier Architecture](picture/10.png)

Add Route of NAT Gateway
![Three Tier Architecture](picture/12.png)

### Step 7: Create a Security Group
Create a Security Group with inbound rules:
- 22 (SSH)
- 80 (HTTP)
- 8080 (Tomcat)
- 3306 (MySQL/RDS)

![Three Tier Architecture](picture/13.png)

![Three Tier Architecture](picture/14.png)

## PART 2: Launch EC2 Instances and Create RDS
| **Tier**             | **Subnet**       | **Description**                                                          | **Security Group** |
| -------------------- | ---------------- | ------------------------------------------------------------------------ | ------------------ |
| **Proxy Tier**       | Public Subnet    | NGINX Reverse Proxy Server (also acts as Bastion Host for SSH access)    | `three-tier-sg`    |
| **Application Tier** | Private Subnet 1 | Apache Tomcat Server hosting the Java-based Student Registration Web App | `three-tier-sg`    |
| **Database Tier**    | Private Subnet 2 | Amazon RDS instance running MariaDB for secure and reliable data storage | `three-tier-sg`    |

### Step 1: Create Proxy-server
![Three Tier Architecture](picture/15.png)

![Three Tier Architecture](picture/16.png)

### Step 2: Create App-server
![Three Tier Architecture](picture/17.png)

![Three Tier Architecture](picture/18.png)

### Step 3: Create DB-server
![Three Tier Architecture](picture/19.png)

![Three Tier Architecture](picture/20.png)

### Step 4: Create RDS instance (MariaDB) in the same VPC with the same security group.
![Three Tier Architecture](picture/42.png)
#### Copy the endpoint and password
![Three Tier Architecture](picture/33.png)

## PART 3: Proxy Server Setup (NGINX)
### Step 1: Connect via SSH to the Proxy instance.
![Three Tier Architecture](picture/21.png)
### Step 2: Install and start NGINX
```bash
sudo yum update -y
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```
![Three Tier Architecture](picture/22.png)
### Step 3: Edit NGINX configuration
#### Go to /etc/nginx/nginx.conf
![Three Tier Architecture](picture/23.png)
#### Inside the server block, add: location / { proxy_pass http://:8080/student/; }
![Three Tier Architecture](picture/24.png)
### Step 4: Restart NGINX

```bash
sudo systemctl restart nginx
```
NGINX will now forward external traffic to your Tomcat server.

## PART 4: Application Server Setup (Tomcat)
### Step 1: From jump server(Proxy) Connect to your App instance.
![Three Tier Architecture](picture/25.png)
### Step 2: Install Java and Tomcat
- update system
- install java
- install tomcat
```bash
sudo yum update -y
sudo yum install java -y
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.98/bin/apache-tomcat-
9.0.98.tar.gz
sudo tar -xvzf apache-tomcat-9.0.98.tar.gz -C /opt
```
![Three Tier Architecture](picture/27.png)

![Three Tier Architecture](picture/28.png)
### Step 3: Check that tomcat is installed correctly
![Three Tier Architecture](picture/29.1.png)
### Step 4: Deploy your application WAR file
#### Deploy your web application inside webapps
```bash
cd /opt/apache-tomcat/webapps
curl -O <S3-Bucket-URL-to-App-Code>
```
![Three Tier Architecture](picture/29.png)
### Step 5: Restart Tomcat
``` bash
cd /opt/apache-tomcat/bin
./catalina.sh stop
./catalina.sh start
```
### Step 6: Check Java
- http://Proxy-Public-IP

![Three Tier Architecture](picture/30.png)
## PART 5: Database Setup (MariaDB / RDS)
### Step 1: SSH into DB instance
![Three Tier Architecture](picture/31.png)
### Step 2: Take access of RDS
- Install mariadb
![Three Tier Architecture](picture/32.png)
- Take access of RDS
![Three Tier Architecture](picture/34.png)
### Step 3:Create database and table
``` bash
CREATE DATABASE studentapp;

USE studentapp;

CREATE TABLE students (
student_id INT NOT NULL AUTO_INCREMENT,
student_name VARCHAR(100) NOT NULL,
student_addr VARCHAR(100) NOT NULL,
student_age VARCHAR(3) NOT NULL,
student_qual VARCHAR(20) NOT NULL,
student_percent VARCHAR(10) NOT NULL,
student_year_passed VARCHAR(10) NOT NULL,
PRIMARY KEY (student_id)
);
```
![Three Tier Architecture](picture/35.png)

## PART 6: Connect App Server to RDS
### Step 1: Install JDBC connector in App server
![Three Tier Architecture](picture/36.png)
### Step 2: Edit the context file
``` bash
cd /opt/apache-tomcat/conf
vim context.xml
```
![Three Tier Architecture](picture/38.png)
### Step 3: Add this configuration inside context block
``` bash
<Resource name="jdbc/TestDB" auth="Container"
type="javax.sql.DataSource"
maxTotal="500" maxIdle="30" maxWaitMillis="1000"
username="admin" password="redhat123!"
driverClassName="com.mysql.jdbc.Driver"
url="jdbc:mysql://<RDS-ENDPOINT>:3306/studentapp?
useUnicode=yes&characterEncoding=utf8"/>
```
![Three Tier Architecture](picture/37.png)
### Step 4: Restart Tomcat
``` bash
cd /opt/apache-tomcat/bin
./catalina.sh stop
./catalina.sh start
```
## PART 7: Access the Application and add entries
#### visit: http://Proxy-Public-IP
![Three Tier Architecture](picture/39.png)
![Three Tier Architecture](picture/40.png)
## PART 8: Verify entries in the RDS
![Three Tier Architecture](picture/41.png)
## Final Result
### Successfully deployed a **3-Tier Web Application on AWS** featuring:

- Isolated **VPC architecture** for secure networking  
- **Public and private subnets** for controlled access  
- **NAT Gateway** enabling private subnet internet access  
- **NGINX Reverse Proxy** for traffic routing and load balancing  
- **RDS (MariaDB)** integration for database storage  
- Fully functional **Student Registration System** hosted on AWS 



