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
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20102430.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20090330.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20102624.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20102715.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20102904.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20%20123230.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20102934.png?raw=true)

## Create Target Group
A target group will route requests to the web servers we create. Our load balancer will need this target group to know what set of servers to distribute traffic to. Our auto scaling group will also be associated with this target group so it launches our servers into the target group.

1. Click on [Create target group](https://eu-west-1.console.aws.amazon.com/ec2/home?region=eu-west-1#CreateTargetGroup:) from the EC2 console to create a target group
2. Choose a target type: **"Instances"**
3. Target group name: **"autoscale-webserver"**
4. Protocol: **"HTTP"**
5. Port: **"80"**
6. VPC: Select the VPC you created
7. Leave every other value on this page as default. **Next**
8. Register Targets: Leave as is. 
9. Click **Create target group**

![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20112959.png?raw=true)

## Create Load Balancer
An application load balancer acts as the entry point for traffic to our webservers. Instead of allowing users to access our application directly, we will use the load balancer to distribute traffic equally among our autoscaling group of web servers. This is better for load management, security and reliability of our application.

1. Click on [Create load balancer](https://eu-west-1.console.aws.amazon.com/ec2/home?region=eu-west-1#SelectCreateELBWizard:) from the EC2 console to create a load balancer
2. Type: **"Application Load Balancer"**
3. Scheme: **"Internet-facing"**
4. IP address type: **"IPV4"**
5. VPC: Select the VPC you created
6. Mappings: Check the box beside the two AZs listed
7. Subnet: For each AZ selected, choose the public subnet in the dropdown menu
8. At this point, go to the Security groups console and create a new security group for the load balancer. The inbound rule should allow HTTP traffic from anywhere. 
9. Select this security group as the load balancer security group
10. Listeners and routing: Leave protocol and port as **HTTP:80**. Select the target group you created as target group
11. Leave every other config as default and click **Create load balancer**

![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20115625.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20115706.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20115013.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20%20115747.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20120350.png?raw=true)

## Create Auto Scaling Group
The auto scaling group configures and controls how your application scales automatically in response to varying traffic situations.

1. Click on [Create Auto Scaling group](https://eu-west-1.console.aws.amazon.com/ec2/home?region=eu-west-1#CreateAutoScalingGroup:) from the EC2 console to create an auto scaling group
2. Enter a name
3. Choose the launch template you created. Click **Next**
4. Select your webserver VPC created from the VPC step
5. Under **Availability Zones and subnets**, select the two public subnets in your VPC, in different AZs. Click **Next** 

***NB:** Note that you can use the auto scaling group to override your instance type config from the launch template*

6. Under **Load balancing**, choose the option **"Attach to an existing load balancer"**
7. Select **Choose from your load balancer target groups**
8. Select the target group you created
9. Select VPC Lattice service to attach: **"No VPC Lattice service"**
10. Additional health check types - optional: **"Turn on Elastic Load Balancing health checks"**
11. Leave every other config as default. **Next**
12. Group size: Desired: **"2"**, Minimum: **"1"**, Maximum: **"4"**
13. Scaling policies: **"Target Tracking Policy"**
14. Metric type: **"Average CPU Utilization"**
15. Target Value: **"50%"**
16. Add notifications - optional (Skipped)
17. Add tags - optional (Skipped)
18. **Create Auto Scaling Group**

Check your configuration below:
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20120800.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20120836.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20120944.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20121043.png?raw=true)
![](https://github.com/Mesiwotso-Gloria/AWS-autoscaling-project/blob/main/images/Screenshot%20%20122038.png?raw=true)

Immediately you create your autoscaling group, you should see two new instances getting created in the EC2 console. This is because we specified a desired count of 2. Also note that they are automatically placed one in each AZ to support high availability.
