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
