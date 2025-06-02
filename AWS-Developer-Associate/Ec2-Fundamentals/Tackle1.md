# Session 1: EC2 Instance Deployment & Security Groups

[![Watch the demo video](https://img.youtube.com/vi/EBY5z_ji9GE/0.jpg)](https://www.youtube.com/watch?v=EBY5z_ji9GE)

## Topic & Key Concepts
- **Topic:** Deploying an EC2 Instance & Security Group Concepts
- **Key Concepts:**
  - EC2 instance types and launch process
  - Security groups (virtual firewalls for instances)
  - Inbound/outbound rules
  - SSH key pairs for access

## Real-World Business Problem
- A startup needs to quickly deploy a web server for their MVP, ensuring only authorized users can access it securely over the internet.

## Use Case & Solution Design
- **Use Case:** Deploy a Linux EC2 instance, configure a security group to allow SSH (port 22) and HTTP (port 80) access, and connect to the instance from a developer’s laptop.
- **Solution Overview:**
  - Launch an EC2 instance using the AWS Console
  - Create and assign a security group with specific inbound rules
  - Generate/download an SSH key pair
  - Connect to the instance using SSH

## Step-by-Step Implementation
1. Open AWS Console and navigate to EC2 dashboard.
2. Click "Launch Instance" and select an Amazon Linux AMI.
3. Choose an instance type (e.g., t2.micro for free tier).
4. Configure instance details as needed.
5. Add storage (default is fine for demo).
6. Create a new security group:
   - Allow inbound SSH (port 22) from your IP
   - Allow inbound HTTP (port 80) from anywhere (0.0.0.0/0)
7. Generate/download a new SSH key pair and save it securely.
8. In the "Advanced details" section, add the following User Data script to automate web server setup:

   ```bash
   #!/bin/bash
   yum update -y
   yum install -y httpd
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Welcome to Our Startup from $(hostname -f)</h1>" > /var/www/html/index.html
   ```

9. Launch the instance.
10. After the instance is running, copy its public IP address.
11. Open a terminal and connect using: `ssh -i <keyfile.pem> ec2-user@<public-ip>`
12. Visit `http://<public-ip>` in your browser to see the welcome page.

## Well-Architected Framework Review
- **Operational Excellence:**
  - Used AWS Console for guided deployment; steps are repeatable and well-documented.
- **Security:**
  - Restricted SSH access to a specific IP; used key pair authentication.
  - Security group acts as a firewall, minimizing attack surface.
- **Reliability:**
  - Default settings provide basic reliability; can add monitoring/alerts for production.
- **Performance Efficiency:**
  - Selected appropriate instance type for workload.
- **Cost Optimization:**
  - Used free tier instance; only running necessary resources.

## Lessons Learned
- Security groups are stateful and easy to configure for least-privilege access.
- Always restrict SSH to known IPs and never expose private keys.