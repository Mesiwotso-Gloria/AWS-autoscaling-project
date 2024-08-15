# Create a load-balanced web server with auto scaling
## Overview

This tutorial walks you through the process of creating a web server which is managed by an auto scaling group. The auto scaling group uses a launch template to create the servers. The auto scaling group launches the servers into a target group. An internet-facing load balancer directs traffic to the target group.

Errors or corrections? Contact [mesiwotsodakpah@gmail.com](mailto:mesiwotsodakpah@gmail.com).

---

<!--Final rev. for launch Oct 2020-->

## Architecture Diagram
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/autoscale-diagram.png?raw=true)

## Create a VPC
A VPC is an isolated, private network you can create to run your workloads. You have complete control over your VPC when you create one.
1. Click on [Create VPC](https://eu-west-1.console.aws.amazon.com/vpc/home?region=eu-west-1#CreateVpc:createMode=vpcWithResources) from the VPC Dashboard to create a new VPC
2. Select **VPC and more** 
3. Enter a name of your choice under **Auto-generate**
4. Choose a **10.0.0.0/16** IPV4 Cidr block
5. Number of Availability Zones (AZs) = **"2"**
6. Number of public subnets = **"2"**
7. NAT gateways ($) = **"In 1 AZ"**
8. *Optional: VPC endpoints* = "S3 Gateway"
9. Leave all other settings as default
10. Click **Create VPC**

Your configuration should match what is shown below:
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20163913.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20082848.png?raw=true)

## Create a Launch Template 

The launch template will serve as the blueprint for creating the exact type of server we need to meet our web server demands. A launch template can be modified to create new versions when you need to change a config.

1. Click on [Create launch template](https://eu-west-1.console.aws.amazon.com/ec2/home?region=eu-west-1#CreateTemplate:) from the EC2 console to create a new launch template
2. Launch template name - required = **"autoscale-webserver"**
3. Check the **Provide guidance to help me set up a template that I can use with EC2 Auto Scaling** box
4. Under **Application and OS Images (Amazon Machine Image) - required**, choose **Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type**
5. Instance type = **"t2.micro"**
6. Key pair - Create a new one or use existing key pair
7. Subnet - **Don't include in launch template**
8. Create security group = **"autoscale-webserver-sg"**
9. Allow SSH and HTTP traffic from 0.0.0.0/0 (Ignore the warning about security group. We will edit it later)
10. VPC - Select the VPC you created
11. Under **Advanced network configuration**, choose **"Enable"** under **Auto-assign public IP**
12. Under **Storage**, leave all other configuration as default and choose **"gp3"** for **Volume type**
13. Resource tags: **Key: Name**, **Value: autoscale-webserver**
14. Under **Advanced details**, scroll down to the **User data** section and enter the following lines of code exactly as shown

```
#!/bin/bash -ex
sudo su
yum -y update
yum install httpd -y
systemctl start httpd
systemctl enable httpd
systemctl status httpd
echo "<html>Hello World, welcome to my server</html>" > /var/www/html/index.html
systemctl restart httpd
amazon-linux-extras install epel -y
yum install stress -y
```
Your configuration should look like this:
![](
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20090330.png?raw=true)
