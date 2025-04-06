# AWS EC2 Auto Scaling Setup with Load Balancer and Apache Web Server

This guide documents the step-by-step process I followed to launch an EC2 instance using the AWS Management Console, configure a custom VPC, install Apache, and set up an Auto Scaling Group with a Load Balancer. Everything was done manually using the AWS Console.

---

## Table of Contents

- [AWS EC2 Auto Scaling Setup with Load Balancer and Apache Web Server](#aws-ec2-auto-scaling-setup-with-load-balancer-and-apache-web-server)
	- [Table of Contents](#table-of-contents)
	- [Overview](#overview)
	- [VPC and Networking Configuration](#vpc-and-networking-configuration)
	- [Security Group Configuration](#security-group-configuration)
	- [EC2 Launch Template](#ec2-launch-template)
		- [Target Group and Load Balancer](#target-group-and-load-balancer)
		- [Auto Scaling Group Setup](#auto-scaling-group-setup)
		- [Installing Apache Web Server](#installing-apache-web-server)
		- [Challenges Faced](#challenges-faced)
		- [Conclusion](#conclusion)
		- [SCREENSHOTS](#screenshots)

---

## Overview

The goal of this project was to:

- Create a fully functional web server accessible over the internet.
- Use Apache as the web server installed on EC2.
- Build a scalable and highly available setup using an Auto Scaling Group.
- Distribute traffic with a Load Balancer.
- Manually configure all infrastructure using the AWS Console.

---

## VPC and Networking Configuration

1. **Create a Custom VPC**
   - CIDR block: `10.0.0.0/16`

2. **Create Subnets**
   - Created 2 public subnets in different availability zones:
     - Subnet A (e.g., `10.0.1.0/24`)
     - Subnet B (e.g., `10.0.2.0/24`)

3. **Create and Attach an Internet Gateway**
   - Created an IGW and attached it to the custom VPC.

4. **Create a Route Table**
   - Added a route for `0.0.0.0/0` via the Internet Gateway.
   - Associated the route table with both subnets to make them public.

---

## Security Group Configuration

1. **Custom Security Group**
   - Inbound rules:
     - HTTP (80) — from `0.0.0.0/0`
     - SSH (22) — from my IP only (for security)
   - Outbound rules:
     - All traffic allowed (default)

---

## EC2 Launch Template

1. **Create a Launch Template**
   - Used Ubuntu AMI.
   - Configured with the custom security group.
   - Added a user data script to install Apache and serve a simple HTML page.

```bash
#!/bin/bash
yes | sudo apt update 
yes | sudo apt install apache2
echo “<h1>Server Details</h1><p><strong>Hostname:</strong> $(hostname)
</p><p><strong>IP Address:</strong> $(hostname -I | cut-d” “ -f1)</р>” >
/var/www/html/index.html
sudo systemctl restart apache2
```


---

### Target Group and Load Balancer
	1.	Create a Target Group
	•	Type: Instance
	•	Protocol: HTTP
	•	Port: 80
	•	Health check path: /
	2.	Create an Application Load Balancer (ALB)
	•	Internet-facing.
	•	Attached to the 2 public subnets.
	•	Security group allowed inbound traffic on port 80.
	•	Listener: Forward traffic to the target group.

---

### Auto Scaling Group Setup
	1.	Create an Auto Scaling Group
	•	Based on the launch template.
	•	Attached to the created load balancer.
	•	Configured across the 2 public subnets.
	•	Set desired capacity: 2
	•	Min capacity: 1
	•	Max capacity: 3
	2.	Scaling Policy
	•	Default manual policy used initially (no dynamic scaling setup).

---

### Installing Apache Web Server

Apache was installed as part of the User Data script in the Launch Template, ensuring that every new EC2 instance created by the Auto Scaling Group is automatically configured as a web server.

---

### Testing the Setup
	•	Visited the Load Balancer DNS name in the browser.
	•	Verified that the HTML page was served successfully.
	•	Terminated one EC2 instance and verified that Auto Scaling replaced it.
	•	Apache remained running and serving traffic after replacement.

---

  ### Challenges Faced
	1.	VPC Misconfiguration
	•	Initially forgot to associate subnets with the route table — no internet access.
	2.	Security Group Issues
	•	Missed allowing HTTP port — couldn’t access the web page.
	3.	Load Balancer Health Check Failures
	•	Health checks failed because Apache wasn’t running immediately on instance boot — solved with user data.
	4.	Auto Scaling Group Not Launching Instances
	•	Incorrect subnet selection (only one AZ) — resolved by spreading across two AZs.
	5.	Apache Not Running
	•	User data script had syntax errors — used #!/bin/bash and tested commands manually before finalizing script.

---

  ### Conclusion

This hands-on project helped me understand the foundational services of AWS including VPC, EC2, Auto Scaling, Load Balancing, and security configurations. I was able to create a scalable, highly available web application setup entirely from the AWS Console.

---
### SCREENSHOTS
![APACHE](https://github.com/Nifesimi-p/aws-compute-services/blob/main/APACHE.png)
![INSTANCES](https://github.com/Nifesimi-p/aws-compute-services/blob/main/INSTANCES.png)
![LOADBALANCER](https://github.com/Nifesimi-p/aws-compute-services/blob/main/LOADBALANCER.png)





