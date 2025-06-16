# Session 2: Elastic Load Balancer (ELB) & Auto Scaling Groups (ASG)

**Demo Video:**

> **Watch Here:** 
[![Watch the demo video](https://img.youtube.com/vi/Gy0WUvrT4Kg/0.jpg)](https://www.youtube.com/watch?v=Gy0WUvrT4Kg)

https://youtu.be/Gy0WUvrT4Kg
## Topic & Key Concepts

### Elastic Load Balancer (ELB)
- **Topic:** Distributing incoming traffic across multiple EC2 instances
- **Key Concepts:**
  - **Application Load Balancer (ALB):** Layer 7 (HTTP/HTTPS) routing with advanced features
  - **Network Load Balancer (NLB):** Layer 4 (TCP/UDP) ultra-high performance
  - **Gateway Load Balancer (GWLB):** Layer 3 for third-party virtual appliances
  - **Target Groups:** Collections of targets (EC2 instances, IP addresses, Lambda functions)
  - **Health Checks:** Automatic monitoring of target health
  - **Sticky Sessions:** Route requests from same user to same instance
  - **Cross-Zone Load Balancing:** Distribute traffic evenly across AZs

### Auto Scaling Groups (ASG)
- **Topic:** Automatically scaling EC2 instances based on demand
- **Key Concepts:**
  - **Launch Templates/Configurations:** Blueprint for launching instances
  - **Scaling Policies:** Rules for when to scale in/out
  - **CloudWatch Metrics:** CPU, network, custom metrics for scaling triggers
  - **Desired/Min/Max Capacity:** Control instance count boundaries
  - **Cooldown Periods:** Prevent rapid scaling oscillations
  - **Instance Refresh:** Rolling updates with new launch templates

## Real-World Business Problem

**Scenario:** Our Local e-commerce company experiences unpredictable traffic patterns:
- **Peak Hours:** Black Friday sales cause 10x normal traffic
- **Off-Peak:** Late night/early morning minimal traffic
- **Geographic Distribution:** Users from multiple regions
- **High Availability Requirements:** 99.9% uptime SLA
- **Cost Constraints:** Budget concerns during low-traffic periods

**Current Issues:**
- Manual server management leads to downtime during traffic spikes
- Over-provisioning during low traffic wastes money
- Single server failure causes complete outage
- No way to handle regional traffic distribution efficiently

## Use Case & Solution Design

### Use Case
Deploy a highly available, auto-scaling web application that can:
1. **Handle Variable Load:** Automatically scale from 2 to 10 instances based on CPU usage
2. **Distribute Traffic:** Use ALB to route traffic across multiple AZs
3. **Ensure High Availability:** Maintain service even if entire AZ fails
4. **Optimize Costs:** Scale down during low traffic periods
5. **Health Monitoring:** Automatically replace unhealthy instances

### Solution Architecture
```
Internet Gateway
     ↓
Application Load Balancer (ALB)
     ↓
Target Groups (Multi-AZ)
     ↓
Auto Scaling Group
├── EC2 Instance (AZ-1a)
├── EC2 Instance (AZ-1b)  
└── EC2 Instance (AZ-1c)
     ↓
CloudWatch Metrics & Alarms
```

### Solution Components
- **ALB:** Routes HTTP traffic across healthy instances
- **ASG:** Maintains 2-10 instances across 3 AZs
- **Launch Template:** Standardized instance configuration
- **Target Group:** Health checks on port 80
- **CloudWatch:** CPU-based scaling policies

## Step-by-Step Implementation

### Phase 1: Create Launch Template
1. **Navigate to EC2 Console → Launch Templates**
2. **Create Launch Template:**
   - Name: `ecommerce-web-template`
   - AMI: Amazon Linux 2023
   - Instance Type: `t3.micro`
   - Key Pair: Select existing or create new
   - Security Groups: Create `web-server-sg`
     - Inbound: HTTP (80) from ALB Security Group
     - Inbound: SSH (22) from your IP
3. **Add User Data Script:**
   ```bash
   #!/bin/bash
   yum update -y
   yum install -y httpd stress
   systemctl start httpd
   systemctl enable httpd
   
   # Create a simple web page showing instance info
   cat > /var/www/html/index.html << 'EOF'
   <!DOCTYPE html>
   <html>
   <head>
       <title>E-Commerce Site</title>
       <style>
           body { font-family: Arial; text-align: center; padding: 50px; }
           .container { max-width: 600px; margin: 0 auto; }
           .instance-info { background: #f0f0f0; padding: 20px; border-radius: 5px; }
       </style>
   </head>
   <body>
       <div class="container">
           <h1>🛒 E-Commerce Platform</h1>
           <div class="instance-info">
               <h3>Instance Information</h3>
               <p><strong>Instance ID:</strong> $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</p>
               <p><strong>Availability Zone:</strong> $(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)</p>
               <p><strong>Private IP:</strong> $(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)</p>
               <p><strong>Load Time:</strong> <span id="loadTime"></span></p>
           </div>
           <h2>Featured Products</h2>
           <div style="display: flex; justify-content: space-around; margin-top: 30px;">
               <div>📱 Smartphones</div>
               <div>💻 Laptops</div>
               <div>🎧 Headphones</div>
           </div>
       </div>
       <script>
           document.getElementById('loadTime').textContent = new Date().toLocaleTimeString();
       </script>
   </body>
   </html>
   EOF
   
   # Replace placeholders with actual values
   sed -i "s/\$(curl -s http:\/\/169.254.169.254\/latest\/meta-data\/instance-id)/$(curl -s http://169.254.169.254/latest/meta-data/instance-id)/g" /var/www/html/index.html
   sed -i "s/\$(curl -s http:\/\/169.254.169.254\/latest\/meta-data\/placement\/availability-zone)/$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)/g" /var/www/html/index.html
   sed -i "s/\$(curl -s http:\/\/169.254.169.254\/latest\/meta-data\/local-ipv4)/$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)/g" /var/www/html/index.html
   ```

### Phase 2: Create Application Load Balancer
1. **Navigate to EC2 Console → Load Balancers**
2. **Create Application Load Balancer:**
   - Name: `ecommerce-alb`
   - Scheme: Internet-facing
   - IP address type: IPv4
   - VPC: Default VPC
   - Availability Zones: Select all available AZs
3. **Create ALB Security Group:**
   - Name: `ecommerce-alb-sg`
   - Inbound Rules:
     - HTTP (80) from 0.0.0.0/0
     - HTTPS (443) from 0.0.0.0/0
4. **Create Target Group:**
   - Name: `ecommerce-tg`
   - Protocol: HTTP
   - Port: 80
   - VPC: Default VPC
   - Health Check Path: `/`
   - Health Check Interval: 30 seconds
   - Healthy Threshold: 2
   - Unhealthy Threshold: 5

### Phase 3: Create Auto Scaling Group
1. **Navigate to EC2 Console → Auto Scaling Groups**
2. **Create Auto Scaling Group:**
   - Name: `ecommerce-asg`
   - Launch Template: `ecommerce-web-template`
   - VPC: Default VPC
   - Availability Zones: Select all available AZs
   - Load Balancer: Attach to `ecommerce-tg`
   - Health Check Type: ELB
   - Health Check Grace Period: 300 seconds
3. **Configure Group Size:**
   - Desired Capacity: 2
   - Minimum Capacity: 2
   - Maximum Capacity: 10
4. **Create Scaling Policies:**
   
   **Scale Out Policy:**
   - Policy Type: Target Tracking Scaling
   - Metric Type: Average CPU Utilization
   - Target Value: 70%
   - Instance Warm-up: 300 seconds
   
   **Scale In Policy:**
   - Automatic (handled by target tracking)
   - Scale-in Protection: Disabled

### Phase 4: Testing and Validation
1. **Wait for instances to launch and pass health checks**
2. **Access the application via ALB DNS name**
3. **Test High Availability:**
   - Terminate one instance manually
   - Verify ASG launches replacement
   - Confirm ALB removes unhealthy targets
4. **Test Auto Scaling:**
   - SSH into an instance: `ssh -i keypair.pem ec2-user@instance-ip`
   - Generate CPU load: `stress --cpu 2 --timeout 600`
   - Monitor CloudWatch metrics
   - Observe new instances launching when CPU > 70%
5. **Test Load Distribution:**
   - Refresh the web page multiple times
   - Notice different instance IDs and AZs serving requests

### Phase 5: Monitoring and Optimization
1. **Set up CloudWatch Alarms:**
   - High CPU utilization across ASG
   - ALB target response time
   - ALB HTTP 4xx/5xx errors


## Well-Architected Framework Review

### Operational Excellence
- ✅ **Infrastructure as Code:** Launch templates ensure consistent deployments
- ✅ **Automation:** Auto Scaling handles capacity management automatically
- ✅ **Monitoring:** CloudWatch provides visibility into system performance

### Security
- ✅ **Network Security:** Security groups restrict access appropriately
- ✅ **Least Privilege:** Web servers only accessible via ALB
- ✅ **Encryption:** Can enable HTTPS with SSL certificates
- ⚠️ **Improvement:** Consider using Systems Manager Session Manager instead of SSH

### Reliability
- ✅ **Multi-AZ Deployment:** Instances distributed across availability zones
- ✅ **Auto Recovery:** ASG automatically replaces failed instances

### Performance Efficiency
- ✅ **Auto Scaling:** Capacity matches demand automatically
- ✅ **Load Distribution:** ALB efficiently distributes traffic

### Cost Optimization
- ✅ **Dynamic Scaling:** Only pay for instances when needed
- ✅ **Free Tier:** Using t3.micro for cost-effective testing

### Sustainability
- ✅ **Resource Efficiency:** Auto Scaling minimizes idle resources
- ✅ **Right-sizing:** Avoiding over-provisioning
- 💡 **Future:** Consider graviton instances for better power efficiency

## Key Learnings & Best Practices

### Load Balancer Best Practices
1. **Use Target Groups:** Easier management and health checking
2. **Enable Cross-Zone Load Balancing:** Better traffic distribution
3. **Configure Proper Health Checks:** Appropriate thresholds and intervals
4. **Use Connection Draining:** Graceful instance removal during scaling
5. **Monitor Key Metrics:** Request count, latency, error rates

### Auto Scaling Best Practices
1. **Use Launch Templates:** More features than Launch Configurations
2. **Set Appropriate Scaling Policies:** Avoid thrashing with proper cooldowns
3. **Use Multiple Metrics:** Don't rely solely on CPU utilization
4. **Test Scaling Events:** Validate behavior under load
5. **Plan for Peak Capacity:** Ensure service limits can handle max scaling


