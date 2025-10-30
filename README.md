🏗️ Auto Scaling and Load Balancing for a Web Application
🎯 Objective

To understand scalability and fault tolerance in cloud computing by setting up a Load Balancer and Auto Scaling Group (ASG) on AWS that automatically increases or decreases the number of EC2 instances based on CPU utilization.

⚙️ Architecture Overview

The setup includes:

VPC with public subnets

Launch Template with a user-data script to deploy a web page automatically

Auto Scaling Group (ASG) with scaling policies

Application Load Balancer (ALB) to distribute traffic

Target Tracking Policy to scale based on average CPU utilization

🧩 Steps Performed
1️⃣ Create Launch Template

Go to EC2 → Launch Templates → Create launch template

Enter details:

Name: WebServerTemplate

AMI: Amazon Linux 2

Instance type: t3.micro

Key pair: select existing

Security group: allow ports 22 (SSH) and 80 (HTTP)

In Advanced Details → User data, add the script below:

#!/bin/bash#!/bin/bash
# Update and install Apache
yum update -y
yum install -y httpd

# Enable and start Apache
systemctl enable httpd
systemctl start httpd

# Create custom HTML page
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
  <head>
    <title>Welcome from ASG Instance</title>
  </head>
  <body style="text-align:center; background-color:#e6f7ff;">
    <h1>It Works! 🎉</h1>
    <p>This page is automatically deployed using User Data from Launch Template.</p>
    <p>Instance ID: $(curl http://169.254.169.254/latest/meta-data/instance-id)</p>
  </body>
</html>
EOF

# Restart Apache to apply everything
systemctl restart httpd

✅ This ensures every new instance automatically installs Apache, starts the web server, and serves a webpage showing its hostname.

2️⃣ Create Target Group

Navigate to EC2 → Target Groups → Create target group

Choose:

Target type: Instances

Protocol: HTTP

Port: 80

VPC: select your VPC

Register no instances now (ASG will attach them later).

3️⃣ Create Application Load Balancer (ALB)

Go to Load Balancers → Create Load Balancer → Application Load Balancer

Configure:

Name: WebApp-LB

Scheme: Internet-facing

IP type: IPv4

Add at least 2 public subnets in different AZs

Under Listeners → Default Action, choose the target group created earlier.

Create the ALB.

✅ Note down the DNS name of the ALB — used for web access.

4️⃣ Create Auto Scaling Group (ASG)

Go to EC2 → Auto Scaling Groups → Create Auto Scaling Group

Steps:

Select Launch Template created earlier

Choose VPC and public subnets

Attach Application Load Balancer and Target Group

Configure Group Size:

Minimum capacity: 2

Desired capacity: 2

Maximum capacity: 4

Enable health checks: EC2 + ELB

Click Next → Create ASG.

5️⃣ Configure Scaling Policy

In your ASG → Automatic Scaling → Add policy

Choose Target Tracking Scaling Policy

Configure:

Metric type: Average CPU utilization

Target value: 45

Instance warm-up time: 60 seconds

Scale in: Enabled

Save policy.

✅ ASG will now automatically:

Scale out (add instances) when CPU > 45%

Scale in (terminate instances) when CPU < 45%

6️⃣ Generate CPU Load (To Test Scaling Out)

SSH into one instance and run:

sudo stress-ng --cpu 4 --timeout 300

or

dd if=/dev/zero of=/dev/null &

Monitor CPU metrics in:

CloudWatch → Metrics → AutoScaling → Group Metrics → CPUUtilization

When average CPU > 45%, ASG launches new instances (up to 4).

7️⃣ Verify Load Balancer

Open Load Balancer DNS in browser:

http://<your-load-balancer-DNS-name>

Refresh multiple times — each refresh should show a different instance hostname, proving that the Load Balancer distributes traffic.

8️⃣ Test Scale-In

Stop the CPU load:

sudo pkill -f stress-ng or kill 

Wait 5–10 minutes.

ASG will automatically terminate extra instances, scaling back to 2.

Check in:

EC2 → Auto Scaling Groups → Activity tab

You’ll see logs such as:

Terminating EC2 instance: Changing desired capacity from 4 to 2

📊 Expected Behavior
Condition	Action
CPU > 45%	ASG scales out (adds instance)
CPU < 45%	ASG scales in (removes instance)
Min Instances	2
Max Instances	4
Load Balancer	Routes traffic to healthy instances

✅ Conclusion

Successfully configured:

-Elastic Load Balancing (ELB) to distribute web traffic
-Auto Scaling Group (ASG) to automatically add/remove instances based on demand
-User Data automation to deploy a web server instantly on each instance
-This demonstrates high availability, scalability, and fault tolerance in a cloud environment.🚀

Author
Apurva Kadam | Cloud Enthusiast
