# Tackle 3: Flask CRUD API with AWS EC2 + Amazon RDS (MySQL)

This is a simple Python Flask REST API that performs CRUD operations on a MySQL database hosted on Amazon RDS. The application is deployed on an Amazon EC2 instance running Amazon Linux.

## 🚀 Features
- ✅ Create a new customer
- 🔍 Read customer details
- ✏️ Update customer email
- ❌ Delete a customer

## 🛠 Stack
- **Python 3.8**
- **Flask**
- **PyMySQL**
- **Amazon EC2** (Amazon Linux AMI)
- **Amazon RDS** (MySQL)

## Topic Overview
**Key Concepts Learned (Pages 144-166):**
- AWS RDS (Relational Database Service)
- RDS MySQL with Multi-AZ deployment
- Database security and encryption
- EC2 instance configuration
- Flask REST API development
- Database connection management

## Real-World Business Problem

**Scenario:** A growing e-commerce startup "TechGadgets Pro" needs a simple customer management system. They have:

- **Current Pain Points:**
  - Need for basic customer data management
  - Simple CRUD operations for customer records
  - Cost-effective solution for small to medium traffic
  - Easy deployment and maintenance

- **Business Requirements:**
  - Simple customer registration and management
  - Reliable data storage with backup capabilities
  - Cost-effective hosting solution
  - Easy to understand and maintain codebase

## Solution Design

### Architecture Overview
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Internet      │────│    EC2 Instance  │────│   RDS MySQL     │
│   (Users)       │    │   (Flask App)    │    │   (Database)    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                                │
                       ┌────────▼────────┐
                       │   Security Groups│
                       │   & IAM Roles    │
                       └─────────────────┘
```

### Technology Stack
- **Database:** Amazon RDS MySQL (for reliable data storage)
- **Compute:** AWS EC2 (for hosting Flask application)
- **Framework:** Flask (for REST API development)
- **Driver:** PyMySQL (for database connectivity)
- **Security:** EC2 security groups and RDS security

## Implementation

### Step 1: Database Setup (RDS MySQL)

#### Database Schema
```sql
-- Create customers table
CREATE TABLE customers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO customers (name, email, phone) VALUES 
('John Doe', 'john.doe@example.com', '+1-555-0123'),
('Jane Smith', 'jane.smith@example.com', '+1-555-0456'),
('Bob Johnson', 'bob.johnson@example.com', '+1-555-0789');
```

### Step 2: Flask Application Implementation

#### Project Structure
```
flask-crud-app/
├── app.py                 # Main Flask application
└── requirements.txt      # Python dependencies
```

#### Flask Application Code

**requirements.txt**
```txt
Flask==2.3.3
PyMySQL==1.1.0
```

**.env (Environment Configuration) - Optional**
```env
# You can set these directly in the code or use environment variables
# DB_HOST=your-rds-endpoint.amazonaws.com
# DB_USER=admin
# DB_PASSWORD=your-secure-password
# DB_NAME=appdb
```

**app.py**
```python
from flask import Flask, request, jsonify
import pymysql

app = Flask(__name__)

# Replace with your real RDS credentials
db = pymysql.connect(
    host='your-rds-endpoint',
    user='admin',
    password='your-password',
    database='appdb',
    port=3306
)

@app.route('/create', methods=['POST'])
def create_customer():
    data = request.json
    cursor = db.cursor()
    cursor.execute("INSERT INTO customers (name, email) VALUES (%s, %s)", (data['name'], data['email']))
    db.commit()
    return jsonify({"message": "Customer created"}), 201

@app.route('/read/<int:cid>', methods=['GET'])
def get_customer(cid):
    cursor = db.cursor()
    cursor.execute("SELECT * FROM customers WHERE id=%s", (cid,))
    row = cursor.fetchone()
    return jsonify({"customer": row}) if row else ("Not found", 404)

@app.route('/update/<int:cid>', methods=['PUT'])
def update_customer(cid):
    data = request.json
    cursor = db.cursor()
    cursor.execute("UPDATE customers SET email=%s WHERE id=%s", (data['email'], cid))
    db.commit()
    return jsonify({"message": "Customer updated"})

@app.route('/delete/<int:cid>', methods=['DELETE'])
def delete_customer(cid):
    cursor = db.cursor()
    cursor.execute("DELETE FROM customers WHERE id=%s", (cid,))
    db.commit()
    return jsonify({"message": "Customer deleted"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

**.env (Environment Configuration)**
```env
# RDS MySQL Configuration (if you want to use environment variables)
DB_HOST=your-rds-endpoint.amazonaws.com
DB_PORT=3306
DB_NAME=appdb
DB_USER=admin
DB_PASSWORD=your-secure-password
```
```python
### Step 3: AWS Deployment

#### Setting up RDS MySQL
1. **Create RDS MySQL Instance:**
   ```bash
   aws rds create-db-instance \
     --db-instance-identifier techgadgets-mysql \
     --db-instance-class db.t3.micro \
     --engine mysql \
     --engine-version 8.0.35 \
     --master-username admin \
     --master-user-password YourSecurePassword123! \
     --allocated-storage 20 \
     --storage-type gp2 \
     --db-name techgadgets \
     --vpc-security-group-ids sg-xxxxxxxxx \
     --backup-retention-period 7 \
     --multi-az \
     --publicly-accessible false
   ```

2. **Configure Security Groups:**
   - Allow inbound connections on port 3306 from EC2 security group
   - Ensure EC2 can connect to RDS instance

#### Setting up EC2 Instance
1. **Launch EC2 Instance:**
   ```bash
   # Launch Amazon Linux 2 instance
   aws ec2 run-instances \
     --image-id ami-0abcdef1234567890 \
     --instance-type t3.small \
     --key-name your-key-pair \
     --security-group-ids sg-xxxxxxxxx \
     --subnet-id subnet-xxxxxxxxx
   ```

2. **Connect and Setup:**
   ```bash
   # Connect to EC2 instance
   ssh -i your-key.pem ec2-user@your-ec2-public-ip
   
   # Update system
   sudo yum update -y
   
   # Install Python 3.8 and pip
   sudo yum install python3 python3-pip git -y
   
   # Clone your application
   git clone https://github.com/your-repo/flask-crud-app.git
   cd flask-crud-app
   
   # Install dependencies
   pip3 install -r requirements.txt
     # Set environment variables (optional - you can edit directly in code)
   export DB_HOST=your-rds-endpoint.amazonaws.com
   export DB_NAME=appdb
   export DB_USER=admin
   export DB_PASSWORD=YourSecurePassword123!
   
   # Run the application
   python3 app.py
   ```

### Step 4: Testing the API

#### Using curl commands:

1. **Create a new customer:**
   ```bash
   curl -X POST http://your-ec2-ip/create \
     -H "Content-Type: application/json" \
     -d '{
       "name": "Alice Johnson",
       "email": "alice@example.com"
     }'
   ```

2. **Get a specific customer:**
   ```bash
   curl -X GET http://your-ec2-ip/read/1
   ```

3. **Update customer email:**
   ```bash
   curl -X PUT http://your-ec2-ip/update/1 \
     -H "Content-Type: application/json" \
     -d '{
       "email": "alice.johnson@newdomain.com"
     }'
   ```

4. **Delete a customer:**
   ```bash
   curl -X DELETE http://your-ec2-ip/delete/1
   ```

#### Expected API Responses:

**Successful Response:**
```json
{
  "message": "Customer created"
}
```

**Get Customer Response:**
```json
{
  "customer": [1, "Alice Johnson", "alice@example.com", null, "2024-01-15 10:30:00", "2024-01-15 10:30:00"]
}
```

**Error Response:**
```json
{
  "error": "Not found"
}
```
```

## Key Benefits of This Solution

### 1. **Simplicity**
- Easy to understand Flask application
- Straightforward deployment process
- Minimal configuration required

### 2. **Cost-Effective**
- Uses t3.micro instances (eligible for free tier)
- RDS MySQL with basic configuration
- No complex serverless architecture

### 3. **Reliability**
- Multi-AZ RDS deployment for high availability
- Automatic backups and point-in-time recovery
- Proven Flask framework for web applications

### 4. **Maintainability**
- Clean, readable Python code
- Standard Flask patterns
- Easy to debug and troubleshoot

## Security Best Practices

1. **Database Security:**
   - Use strong passwords
   - Enable encryption at rest
   - Restrict network access with security groups

2. **Application Security:**
   - Input validation and sanitization
   - Environment variables for sensitive data
   - Proper error handling without exposing internals

3. **Infrastructure Security:**
   - Regular security updates
   - VPC with private subnets for RDS
   - IAM roles with minimal permissions

## Monitoring and Logging

- **Application Logs:** Python logging to track operations
- **RDS Monitoring:** CloudWatch metrics for database performance
- **EC2 Monitoring:** System metrics and application health checks

## Next Steps

1. **Add Authentication:** Implement JWT or session-based auth
2. **Load Balancing:** Use Application Load Balancer for multiple instances
3. **SSL/TLS:** Configure HTTPS with Let's Encrypt or AWS Certificate Manager
4. **Automated Deployment:** Set up CI/CD pipeline with GitHub Actions---

**Completion Status:** ✅ **TACKLE 3 COMPLETE**

**Topics Mastered:**
- AWS RDS MySQL configuration and management
- EC2 instance setup and Flask application deployment
- Database connection management and security
- RESTful API development with Flask
- CRUD operations with proper error handling
- AWS security best practices for RDS and EC2

**Ready for Next Challenge:** 🚀 **TACKLE 4 - Advanced AWS Services & Serverless Architecture**