# Tackle 3: Flask CRUD API with AWS EC2 + Amazon RDS (MySQL)

This is a simple Python Flask REST API that performs CRUD operations on a MySQL database hosted on Amazon RDS. The application is deployed on an Amazon EC2 instance running Amazon Linux.

**Note:** While this hands-on implementation focuses on **Amazon RDS MySQL** for practical learning, this tackle also covers key concepts about **Amazon Aurora** (AWS's cloud-native database) and **caching strategies** with ElastiCache that are essential for the AWS Developer Associate exam.

## Features
- Create a new customer
- Read customer details
- Update customer email
- Delete a customer

## 🛠 Stack
- **Python 3.8**
- **Flask**
- **PyMySQL**
- **Amazon EC2** (Amazon Linux AMI)
- **Amazon RDS** (MySQL)

## Topic Overview
**Key Concepts Learned:**
- **AWS RDS** (Relational Database Service) - Multi-AZ, Read Replicas, Backup & Recovery
- **Amazon Aurora** - Cloud-native database, Aurora Serverless, Global Database
- **Database Performance** - Connection pooling, Query optimization, Indexing strategies
- **Caching Strategies** - ElastiCache (Redis/Memcached), Cache-aside patterns, TTL management
- **Database Security** - Encryption at rest/transit, VPC security, IAM database authentication
- **EC2 integration** - Security groups, Database connectivity, Application deployment

### **Deep Dive into Key Concepts:**

#### **1. AWS RDS (Relational Database Service)**
AWS RDS is a managed database service that handles the heavy lifting of database administration:
- **Multi-AZ Deployments:** Automatic failover to a standby instance in a different Availability Zone for high availability
- **Read Replicas:** Scale read operations by creating up to 15 read-only copies of your database
- **Automated Backups:** Point-in-time recovery with automated snapshots and transaction logs
- **Engine Support:** MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora

#### **2. Amazon Aurora**
Aurora is AWS's cloud-native relational database built for the cloud:
- **Performance:** Up to 5x faster than MySQL and 3x faster than PostgreSQL
- **Aurora Serverless:** Automatically scales compute capacity based on demand, perfect for intermittent workloads
- **Global Database:** Span multiple AWS regions with low-latency global reads and disaster recovery
- **Storage:** Automatically grows from 10GB to 128TB, with 6-way replication across 3 AZs

#### **3. Database Performance Optimization**
Critical techniques for maintaining high-performance databases:
- **Connection Pooling:** Reuse database connections to reduce overhead and improve response times
- **Query Optimization:** Use EXPLAIN plans, proper indexing, and efficient SQL queries
- **Indexing Strategies:** Create indexes on frequently queried columns, but balance with write performance
- **Monitoring:** Use CloudWatch metrics and Performance Insights to identify bottlenecks

#### **4. Caching Strategies with ElastiCache**
Implement caching to dramatically improve application performance:
- **Redis vs Memcached:** Redis for complex data structures and persistence, Memcached for simple key-value caching
- **Cache-Aside Pattern:** Application manages cache, checks cache first, then database if cache miss
- **Write-Through/Write-Behind:** Different patterns for updating cache when data changes
- **TTL Management:** Set appropriate Time-To-Live values to balance data freshness with performance

##### **Detailed Caching Patterns:**

**1. Cache-Aside (Lazy Loading) Pattern:**
```python
# Most common pattern - application manages cache
def get_customer(customer_id):
    # 1. Check cache first
    cache_key = f"customer:{customer_id}"
    cached_data = cache.get(cache_key)
    
    if cached_data:
        return cached_data  # Cache hit
    
    # 2. Cache miss - get from database
    customer = database.get_customer(customer_id)
    
    # 3. Store in cache for next time
    cache.set(cache_key, customer, ttl=3600)  # 1 hour TTL
    return customer
```
**Pros:** Only cache data that's actually requested, simple to implement
**Cons:** Cache miss penalty, potential stale data

**2. Write-Through Pattern:**
```python
# Write to cache and database simultaneously
def update_customer(customer_id, data):
    # 1. Update database first
    database.update_customer(customer_id, data)
    
    # 2. Update cache immediately
    cache_key = f"customer:{customer_id}"
    cache.set(cache_key, data, ttl=3600)
    
    return data
```
**Pros:** Cache is always consistent with database
**Cons:** Write latency, cache may contain unused data


##### **Redis vs Memcached Comparison:**

**Use Redis when you need:**
- Complex data structures (hashes, lists, sets, sorted sets)
- Persistence and data durability
- Pub/Sub messaging
- Lua scripting
- Atomic operations
- Master-slave replication

**Use Memcached when you need:**
- Simple key-value caching
- Multi-threaded performance
- Lower memory overhead
- Simple horizontal scaling


#### **5. Database Security Best Practices**
Comprehensive security measures for database protection:
- **Encryption at Rest:** Protect stored data using AWS KMS encryption
- **Encryption in Transit:** Secure data moving between application and database using SSL/TLS
- **VPC Security:** Isolate databases in private subnets with security groups controlling access
- **IAM Database Authentication:** Use AWS IAM instead of database passwords for enhanced security

#### **6. EC2 and Database Integration**
Best practices for connecting applications to databases:
- **Security Groups:** Configure inbound/outbound rules to allow only necessary database traffic
- **Network Architecture:** Place databases in private subnets, applications in public/private subnets
- **Connection Management:** Handle database connections efficiently to avoid connection pool exhaustion
- **Monitoring:** Track application and database metrics to ensure optimal performance

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
    host='techgadget.ccfig4ammrcv.us-east-1.rds.amazonaws.com',
    user='admin',
    password='farah1234',
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
   
   # Run this to install Mysql mariadb
   sudo yum install mariadb -y
   
   # Connect to your RDS MySQL instance
   mysql -h <your-rds-endpoint> -u admin -p


   # Create Directory and the editor
   mkdir flask-crud
   cd flask-crud
   nano app.py

   # Paste the code in the editor
   
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



