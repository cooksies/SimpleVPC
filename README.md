# SimpleVPC
<div align="center">
  <p>Demonstrating how a VPC is configured</p>
  <image src="https://github.com/cooksies/SimpleVPC/assets/75002188/46b4bc97-aa6d-43cb-b8e6-53dfddbf8d56">
</div>

# Setting up the environment
In the AWS Console, we create a Lab VPC with a CIDR of 10.0.0.0/16

This IP address will be our local IP address for the environment. This will be seen by default in a subnet's routes when it is created (associated) in the VPC.

| Element | Value |
|---|---|
| VPC | Lab VPC |
| CIDR block | 10.0.0.0/16 |
| DNS Resolution | Enabled |
| DNS Hostnames | Enabled |

# Creating Subnets

<strong><ins>Public Subnet</ins></strong>

When creating the subnets, the VPC ID should be Lab VPC.

Name the Subnet "Public Subnet" and choose the first Availability Zone. We are only utilizing one availability zone. 
For Example from the dropdown, you will only need "us-west-2a" for both the Public and Private subnets.

Give the IPv4 CIDR block as 10.0.0.0/24. Then Save the subnet.

Note: Even though the subnet has been named "Public Subnet", it is not yet public.
It still requires an internet gateway.

| Element | Value |
| --- | --- |
| VPC ID | Lab VPC |
| Subnet Name | Public Subnet |
| CIDR Block | 10.0.0.0/24 |

<strong><ins>Public Subnet</ins></strong>

The steps are similar to making a public subnet.

The differences are to name it "Private Subnet" and change the IPv4 CIDR block to "10.0.2.0/23"

Note: The CIDR block of 10.0.2.0/23 includes all IP addresses that start with 10.0.2.x and 10.0.3.x.\
This range is twice as large as the public subnet.

| Element | Value |
| --- |---|
| VPC ID | Lab VPC |
| Subnet Name | Private Subnet |
| CIDR Block | 10.0.2.0/23 |

# Creating an Internet Gateway

An Internet gateway is used to establish outside connectivity to EC2 instances in VPCs.

In the AWS console, when creating an internet gateway give the Name tag as "Lab IGW". Then confirm the settings.\
Ensure that it is attached to Lab VPC, by choosing the Actions dropdown choosing Attach to VPC and choosing "Lab VPC"

In doing this, the public subnet now has a connection to the internet, but the route table is not currently using the internet gateway,\
which means traffic from the subnet is not routed to the internet.

# Configure Route table

In the AWS console, the default route table during the creation of the VPC will be named "Private Route Table"
| Private route table | Details |
|---|---|
| Route | Default |
|Source|10.0.0.0/16 (VPC CIDR)|
|Destination|Local|

Now create a public route table to send traffic to the internet gateway. Ensure that it's VPC value is Lab VPC.\
After creating the route table, edit it route and add a route to direct internet-bound traffic to the internet gateway
| Public route table | Details |
|---|---|
|Name| Public Route Table|
|VPC| Lab VPC|
|Destination|0.0.0.0/0|
|Target|Lab IGW|

The final step is to associate the new public route table with the public subnet.\
Go to the route table's subnet association, edit the subnet association and select Public Subnet.

# Launching bastion server in the public subnet
In the EC2 management console, Launch an instance with the following options:
|Bastion server settings|Value|
|---|---|
| Name and tags| Bastion Server|
|Application and OS Images | Amazon Linux 2023 AMI |
|Key Pair | Proceed without a key pair|

|Network settings|Value|
|---|---|
|VPC| Lab VPC|
|Subnet| Public Subnet|
|Auto-assign public IP|Enable|

|Security GroupS|Value|
|---|---|
|Security Group name|Bastion security group|
|Description| Allow SSH|
|Type|SSH|
|Source Type| Anywhere|

After inputting the Bastion instance settings, launch the instance

# Creating a NAT gateway

A NAT gateway is launched in the public subnet.\
It is used to allow the instances in the private subnet to communicate to the internet while preventing traffic from entering the private subnet.

Create a NAT gateway and name it "Lab NAT Gateway" and choose its subnet to be Public Sebnet.\
Then click on Allocate Elastic IP. Then hit Create a NAT gateway

|NAT gateway| Detials|
|---|---|
|Name|Lab NAT gateway|
|Subnet| Public Subnet|
|Elastic IP| As allocated by the system|

Now that a NAT gateway is created, you can configure the private subnet to send internet-bound traffic to the NAT Gateway.\
This is configured under the route tables section and selecting "Private Route Table"

Under the Routes tab of Private Route Table, edit the routes and add destination of 0.0.0.0/0 with a target of NAT Gateway

|Private Route Table| Route details|
|---|---|
|Destination|0.0.0.0/0|
|Target|NAT gateway|
