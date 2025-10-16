Create a New VPC with Public & Private Subnets¶
We'll design a custom VPC called ditiss-lab.

Step 1: Create VPC
```
Go to VPC in console → Create VPC.
Resources to create: VPC Only
Name: ditiss-lab.
IPv4 CIDR block: IPv4 CIDR manual input
IPv4: 172.20.0.0/16.
Leave all other settings as defaul
Click on Create VPC button.
```
Step 2: Create Subnets
```
On the VPC Dashboard page, on the right side under Virtual Private Cloud, click on Subnets
Click on Create Subnet
VPC ID: Select the subnet which has ditiss-lab as suffix
Public Subnet:
Subnet Name: public-subnet.
Availability Zone: No Preference
IPv4 VPC CIDR block: Selected by default 172.20.0.0/16
IPv4 subnet CIDR block: 172.20.5.0/24.
Private Subnet:
Click on Add new subnet button
Subnet Name: private-subnet.
IPv4 VPC CIDR block: Selected by default 172.20.0.0/16
IPv4 subnet CIDR block: 172.20.10.0/24.
Click on Create Subnet button
```
Step 3: Internet Gateway (IGW)
```
On the VPC Dashboard page, on the right side under Virtual Private Cloud, click on Internet Gateways
Click on Creat Internet gateway
Name Tag: igw-dittis-lab
Click on Create Internet Gateway
One the homepage of Internet gateways, select your newly created igw-dittis-lab
Click on Action button on the top right side
Select Attach to VPC
Select ditiss-lab
Click on Attach internet gateway
```

Step 4: Create Route Tables
```
On the VPC Dashboard page, on the right side under Virtual Private Cloud, click on Route Tables
Click on Create Route Tables
Name: rtb-dittis-lab
VPC: ditiss-lab
Click on Create route table
On the Route tables Dashboard, select the newly created route table rtb-dittis-lab and click on Action button on top right side.
Click on Edit Routes
Click on Add Route
Destination: 0.0.0.0/0
Target: Internet Gateway
Select igw-dittis-lab
Click on Save Changes
```
Step 5: Change settings of public-subnet
```
On the VPC Dashboard page, on the right side under Virtual Private Cloud, click on Subnets
Select public-subnet
Click on Actions button on top right side
Click on Edit subnet settings
Select/Check the Enable auto-assign public IPv4 address in Auto-assign IP settings window
Click on Save
Again, on Subnets dashboard, select public-subnet and click on Actions button on top right side
The select Edit route table association
Route Table ID: Select rtb-ditiss-lab
Click on Save
```
Step 6: Change settings of private-subnet

Alert
This setting is only for demo purposes, in real world, a private subnet should never have a public IP assigned. Instead we use a Jump/Bastion Host which acts as a jump betwen internet and private subnets.
On the VPC Dashboard page, on the right side under Virtual Private Cloud, click on Subnets
```
Select private-subnet
Click on Actions button on top right side
Click on Edit subnet settings
Select/Check the Enable auto-assign public IPv4 address in Auto-assign IP settings window
Click on Save
Again, on Subnets dashboard, select public-subnet and click on Actions button on top right side
The select Edit route table association
Route Table ID: Select rtb-ditiss-lab
Click on Save
```
Step 7: Create Security Groups
Info
Security Groups are integral part of any VPC. This security groups are the network firewall rules which controls what traffic to flow into the hosts/machines inside those security groups. Essentailly we tell the virtual machines which networks and ports you can access or cannot access.
```
On the VPC Dashboard page, on the right side under Security, click on Security Groups
Click on Create security group button on top right side
Enter following details
Security group name: sg-public
Description: public ssh & http access
VPC: Select ditiss-lab
Inside Inbound rules tab, click on Add rule
Type: SSH
Source: Custom
IP Range: 0.0.0.0/0
Again click on Add rule
Type: HTTP
Source: Custom
IP Range: 0.0.0.0/0
Keep other settings as default
Click on Create security group

Repeat the same steps again for creating one more security group for private subnet

On the VPC Dashboard page, on the right side under Security, click on Security Groups
Click on Create security group button on top right side
Enter following details

Security group name: sg-private
Description: private ssh & http access
VPC: Select ditiss-lab
Inside Inbound rules tab, click on Add rule

Type: SSH
Source: Custom
IP Range: 0.0.0.0/0
Again click on Add rule
Type: HTTP
Source: Custom
IP Range: 172.20.0.0/0
Keep other settings as default
Click on Create security group
```
Step 8: Create Instances
```
In search bar, search for EC2
Click on Launch Instances

Public Instance:
Name: public-instance.
Choose Amazon Linux 2 AMI (Free Tier eligible).
Select t3.micro (Free Tier).
Use the same key-pair used for creating inital virtual machine at the beginning or create new one as per your prefrence.
In the Network Settings:

Click on Edit
VPC: Select the ditiss-lab
Subnet: public-subnet
Firewall (security groups): Select existing security groups
Select sg-public from the dropdown.
Keep other settings as default
Click on Launch Instance

Private Instance:
Name: private-instance.
Choose Amazon Linux 2 AMI (Free Tier eligible).
Select t3.micro (Free Tier).
Use the same key-pair used for creating inital virtual machine at the beginning or create new one as per your prefrence.
In the Network Settings:
Click on Edit
VPC: Select the ditiss-lab
Subnet: private-subnet
Firewall (security groups): Select existing security groups
Select sg-private from the dropdown.
Keep other settings as default
Click on Launch Instance
```
