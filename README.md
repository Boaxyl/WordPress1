# WordPress Deployment on AWS – Project Documentation


# WordPress Deployment on AWS – Project Documentation

## Project Overview
In this project, I deployed a **highly available WordPress site** on AWS using the following services:

- **Amazon VPC** with public and private subnets across 2 Availability Zones  
- **Internet Gateway** and **NAT Gateways** for internet connectivity  
- **Amazon EC2** instances in an **Auto Scaling Group** for web servers  
- **Amazon EFS** for shared WordPress wp-content storage  
- **Amazon RDS (MySQL)** for the WordPress database  
- **Application Load Balancer (ALB)** to distribute traffic  
- **Security Groups** for controlled access  
- **Amazon Route 53** for domain name resolution  

---

## Step 1: Create VPC and Subnets
1. I created a VPC with CIDR block `10.0.0.0/16`.  
2. I created **6 subnets**:
   - Public Subnet A: `10.0.0.0/24`
   - Public Subnet B: `10.0.1.0/24`
   - Private App Subnet A: `10.0.10.0/24`
   - Private App Subnet B: `10.0.11.0/24`
   - Private DB Subnet A: `10.0.20.0/24`
   - Private DB Subnet B: `10.0.21.0/24`
3. I attached an **Internet Gateway** to the VPC.  
4. I created a **Public Route Table** and added a default route (`0.0.0.0/0`) to the Internet Gateway.  
5. I created **Private Route Tables** for app and DB subnets.  
6. I allocated 2 Elastic IPs and created **2 NAT Gateways** (one in each public subnet).  
7. I updated private route tables to route internet traffic through the corresponding NAT Gateway.  

---

## Step 2: Configure Security Groups
I created the following security groups:

- **ALB-SG**  
  - Inbound: HTTP (80) from anywhere  
  - Outbound: All traffic  

- **Web-SG**  
  - Inbound: HTTP (80) from ALB-SG  
  - Outbound: MySQL (3306) to RDS-SG, NFS (2049) to EFS-SG  

- **RDS-SG**  
  - Inbound: MySQL (3306) from Web-SG  

- **EFS-SG**  
  - Inbound: NFS (2049) from Web-SG  

---

## Step 3: Create Amazon EFS
1. I created an EFS file system inside the VPC.  
2. I enabled mount targets in the private app subnets.  
3. I attached the **EFS-SG** to the mount targets.  
4. I noted the EFS DNS name for later mounting.  

---

## Step 4: Create Amazon RDS (MySQL)
1. I created a **MySQL RDS instance** in Multi-AZ mode.  
2. I placed it inside the private DB subnets.  
3. I attached the **RDS-SG** to the database.  
4. I set a DB name, username, and password to connect from WordPress.  

---

## Step 5: Create Application Load Balancer
1. I created an **Application Load Balancer** in the two public subnets.  
2. I attached the **ALB-SG**.  
3. I created a **target group** (HTTP, port 80) for the web servers.  
4. I added a listener on port 80 forwarding traffic to the target group.  

---

## Step 6: Create IAM Role for EC2
1. I created an IAM role `EC2-WP-Role`.  
2. I attached the following policies:
   - **AmazonSSMManagedInstanceCore**  
   - **AmazonElasticFileSystemClientFullAccess**  
   - **CloudWatchAgentServerPolicy** (optional)  
3. I attached this IAM role to my EC2 instances via the launch template.  

---

## Step 7: Configure Launch Template with User Data
I created a **Launch Template** for my EC2 web servers with the following settings:
- AMI: Amazon Linux 2  
- Instance type: t3.medium  
- Security Group: Web-SG  
- IAM Role: EC2-WP-Role  
- User Data script to:
  - Install Apache, PHP, and WordPress  
  - Mount **EFS** to `/var/www/html/wp-content`  
  - Configure WordPress to use **RDS**  
  - Start the web server  

---

## Step 8: Configure Auto Scaling Group
1. I created an **Auto Scaling Group (ASG)** with:
   - Launch Template from Step 7  
   - Private app subnets (A and B)  
   - Attached to the ALB target group  
2. I set:
   - Desired capacity: 2  
   - Min: 2  
   - Max: 4  
3. I enabled **ELB health checks** so unhealthy instances are replaced automatically.  

---

## Step 9: Configure Route 53
1. I created an **A Record (Alias)** in Route 53 pointing to the ALB DNS name.  
2. Now my domain resolves to the WordPress site running behind the ALB.  

---

## Step 10: Test the Setup
- I accessed the ALB DNS name and confirmed WordPress installation screen loaded.  
- I uploaded media and confirmed it persisted across multiple EC2 instances (thanks to EFS).  
- I checked database connection with RDS – it worked fine.  
- I scaled the Auto Scaling Group up and down to confirm high availability.  
- I secured the site by adding an **SSL certificate** via ACM and updating the ALB listener to HTTPS.  

---

## Final Notes
By completing this project:
- I learned how to design and deploy a highly available WordPress architecture on AWS.  
- I implemented **VPC, subnets, NAT Gateway, ALB, Auto Scaling, RDS, and EFS**.  
- I tested resilience, database connectivity, and shared storage persistence.  

This architecture is production-ready and can scale to handle high traffic.  
```

---



