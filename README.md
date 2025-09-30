# 🚀 Highly Available & Scalable Web Deployment on AWS with CloudFormation
### This project shows how to deploy a highly available and scalable web infrastructure on AWS using CloudFormation.

The stack creates:

- An Application Load Balancer (ALB) to distribute traffic
- An Auto Scaling Group (ASG) with EC2 instances
- Security Groups for controlled access
- SNS Notifications for scaling activities
- A simple Apache web server running on each EC2 instance



### 📌 Features

- Infrastructure as Code (IaC) – reproducible CloudFormation template
- Scalable & Highly Available – Auto Scaling with multiple subnets (AZs)
- Secure Access – ALB only allows HTTP, EC2 allows SSH from your IP
- Notifications – Auto Scaling events sent to email (SNS)
- Web Ready – Apache installed automatically with a sample HTML page

### ⚙️ Prerequisites

Before starting, make sure you have:
1. AWS Account with permissions to create VPC, EC2, ALB, ASG, and SNS resources.
2. An EC2 Key Pair in your AWS region (to SSH into instances if needed).
- Create one:
  - Go to EC2 Console → Key Pairs → Create key pair
  - Download the .pem file and keep it safe.
3. A VPC with at least 2 public subnets in different Availability Zones.
- You can use your default VPC (check VPC Console → Subnets).

---

### 🍞 Baking a Custom AMI (Optional – for your own website)

By default, the template installs Apache and serves a simple **Hello World** page.  
If you want to deploy your own website instead, you can bake a **custom AMI** with your web content:

1. Launch a temporary EC2 instance (Amazon Linux 2).
2. Connect via SSH:
   ```bash
   ssh -i your-key.pem ec2-user@<EC2-Public-IP>
   ```
3. Install Apache and copy your website files:
  ```bash
  sudo yum update -y
  sudo yum install -y httpd
  sudo systemctl enable httpd
  sudo systemctl start httpd
  sudo rm -rf /var/www/html/*
  sudo cp -r /home/ec2-user/your-website/* /var/www/html/
```
4. Test in browser → http://<EC2-Public-IP> (make sure security group allows port 80).
5. Create an AMI from this instance:
  - Go to EC2 Console → Instances
  - Right-click your instance → Image and templates → Create Image
  - Give it a name like my-custom-web-ami
  - Click Create image
6. Copy the new AMI ID (e.g., ami-0abcd1234efgh5678).
7. Use this AMI ID as the LatestAmiId parameter when creating your CloudFormation stack.

✅ Result: Every instance in your Auto Scaling Group will boot with your website already baked in.

---

### 🚀 Step-by-Step Deployment (AWS Console)

**1. Log in to AWS Console**
- Open: https://console.aws.amazon.com/
- Select the region where you want to deploy (top right).

**2. Go to CloudFormation**
- Navigate to Services → CloudFormation.
- Click Create stack → With new resources (standard).

**3. Upload the Template**
- In Specify template:
    - Choose Upload a template file.
    - Click Choose file and select template.yaml from this repo.
    -  Click Next.

 **4. Configure Stack Details**

Fill in the parameters:
- Stack name: my-web-app (or any name)
- Parameters:
  - InstanceType → choose t2.micro (default, free-tier)
  - KeyName → your existing EC2 key pair
  - LatestAmiId → leave default (Amazon Linux 2)
  - OperatorEmail → your email (for scaling notifications)
  - SSHLocation → your IP (recommended) or 0.0.0.0/0 (open to all)
  - Subnets → select at least two public subnets (hold CTRL/CMD to pick multiple)
  - VPC → choose your VPC ID
    
Click Next.

**5. Configure Stack Options**

- You can leave everything as default.
- (Optional) Add tags for cost tracking, e.g.:
    - Key: Project, Value: WebApp
- Click Next.

**6. Review & Create**

- Review all settings.
- Scroll to the bottom → check the box:
   - "I acknowledge that AWS CloudFormation might create IAM resources."
- Click Create stack.

**7. Wait for Deployment**

- The stack will take a few minutes to create resources.
- Watch progress in the Events tab.
- Once complete, status will show: CREATE_COMPLETE.

<img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Scalable-Web-Deployment-on-AWS-with-CloudFormation/raw/main/images/cf01.png" width="600" alt="SNS Config">

**8. Get the Load Balancer DNS**
- Open the stack → Outputs tab.
- Copy the value of ALBDNSName.
  
  Example:
``my-web-app-alb-1234567890.us-east-1.elb.amazonaws.com``

**9. Test Your Website**

- Open the DNS in a browser: ``http://my-web-app-alb-1234567890.us-east-1.elb.amazonaws.com``
 
✅ You should now see your own customized website.

<img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Scalable-Web-Deployment-on-AWS-with-CloudFormation/raw/main/images/cf5.png" width="600" alt="SNS Config">

(This stack uses a custom AMI baked with the web application, and that AMI ID is referenced in the Launch Template. Instead of the default Apache “Hello World” page, your pre-baked site is served immediately after deployment.)

**🔔 Confirm Email Subscription (SNS)**

- Check your inbox for an email from AWS Notifications.
- Click Confirm subscription.
- After that, you’ll get emails whenever Auto Scaling adds/removes instances.

---

**📖 What This Template Builds (Expanded)**

 **🔒 Security Groups**
  - ELBSecurityGroup → Allows inbound HTTP (port 80) traffic from anywhere (0.0.0.0/0). This makes your Application Load Balancer publicly accessible.
  - EC2SecurityGroup →
    - Allows inbound HTTP traffic only from the ALB Security Group (not directly from the internet, so instances aren’t exposed).
    - Allows inbound SSH (port 22) traffic only from your IP (or a CIDR you specify), keeping remote access restricted and secure.

 **🌍 Application Load Balancer (ALB)**
  - Internet-facing load balancer that distributes web traffic across EC2 instances in multiple Availability Zones.
  - Improves high availability (traffic keeps flowing even if one AZ or instance fails).

**🎯 Target Group**
  - Registered with the ALB, this is the "pool" of EC2 instances that receive traffic.
  - Performs health checks (HTTP 200 on /) to ensure only healthy instances serve traffic.

**📦 Launch Template**
- Defines how each EC2 instance is configured:
  - Uses Amazon Linux 2 AMI.
  - Installs and starts Apache HTTP Server.
  - Serves a sample Hello World page (or your app).
- Provides a consistent baseline for all Auto Scaling instances.

<img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Scalable-Web-Deployment-on-AWS-with-CloudFormation/raw/main/images/cf4.png" width="600" alt="SNS Config">

**⚖️ Auto Scaling Group (ASG)**
  - Ensures that a minimum of 1 instance is always running and can scale up to 3 instances if demand increases.
  - Automatically replaces unhealthy instances.
  - Spreads instances across multiple subnets / Availability Zones → fault tolerance.

<img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Scalable-Web-Deployment-on-AWS-with-CloudFormation/raw/main/images/cf3.png" width="600" alt="SNS Config">

**📢 SNS Topic**
  - Sends notifications via email whenever scaling events occur (instance launched, terminated, failed health check, etc.).
  - Keeps operators aware of infrastructure changes.

<img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Scalable-Web-Deployment-on-AWS-with-CloudFormation/raw/main/images/cf6.png" width="600" alt="SNS Config">

**📤 Outputs**
- Provides easy-to-copy values after stack creation:
  - ALB DNS Name → the URL of your deployed app.
  - ASG Name → useful if you want to manage scaling later.
  - SNS ARN → reference for the notification topic.

 ---

 **🛠️ Updating the Stack**

If you edit template.yaml (e.g., change instance type, user data, etc.):
1. Go to CloudFormation Console.
2. Select your stack.
3. Click Update → Replace current template → Upload file.
4. Upload the new template and click Next until finished.

**🗑️ Cleanup**

To avoid costs, delete the stack when done:
  1. Go to CloudFormation Console.
  2. Select your stack.
  3. Click Delete → Delete stack.
     
This removes all resources created by the template (EC2, ALB, Security Groups, SNS, etc.).

**📎 References**

- AWS CloudFormation User Guide
- Auto Scaling Documentation
- Application Load Balancer Guide
