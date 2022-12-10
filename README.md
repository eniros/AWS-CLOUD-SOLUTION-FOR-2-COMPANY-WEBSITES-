# AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-
AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![sitearc](https://user-images.githubusercontent.com/61475969/206863730-b1bd4c3e-8e56-43ed-a736-c8ac48699520.png)

STEP 1: SET UP ORGANISATION UNIT (OU) and a SUB-ACCOUNT

Properly configure your AWS account and Organization Unit

Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)

Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)

Move the DevOps account into the Dev OU.

Login to the newly created AWS account using the new email address.

Created a free domain name for company at Freenom domain registrar here

Created a hosted zone in AWS, and map it to your domain

<img width="1031" alt="Screenshot 2022-12-10 at 16 14 03" src="https://user-images.githubusercontent.com/61475969/206865382-25f24aee-f99d-4aa7-9ae1-8f32531812b9.png">

<img width="948" alt="Screenshot 2022-12-10 at 16 40 28" src="https://user-images.githubusercontent.com/61475969/206865519-a7155fe2-1d92-4541-9813-53c567710cd3.png">

Setting Up Infrastucture

1.Created a VPC
2.Created subnets as shown in the architecture 

<img width="775" alt="Screenshot 2022-12-10 at 17 14 34" src="https://user-images.githubusercontent.com/61475969/206867059-533b761d-8252-4c68-b2d3-9e395ce314f1.png">

<img width="775" alt="Screenshot 2022-12-10 at 17 15 22" src="https://user-images.githubusercontent.com/61475969/206867082-c78e16e0-a9ed-4dd2-975a-b198ad0c1120.png">

3. Created a route table and associate it with public subnets

<img width="963" alt="Screenshot 2022-12-10 at 17 16 32" src="https://user-images.githubusercontent.com/61475969/206867133-b6e0a4af-ad07-4854-886c-d45a7980597d.png">

4. Created a route table and associate it with private subnets 
<img width="915" alt="Screenshot 2022-12-10 at 17 17 36" src="https://user-images.githubusercontent.com/61475969/206867173-2ddcee6b-21fb-4a45-99d6-353610f2e353.png">

5. Created an Internet Gateway

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
<img width="803" alt="Screenshot 2022-12-10 at 17 19 19" src="https://user-images.githubusercontent.com/61475969/206867251-df5594df-4e61-49f5-9fdc-d7a8cdf61c67.png">
 
7. Created 3 Elastic IPs 
<img width="971" alt="Screenshot 2022-12-10 at 17 20 19" src="https://user-images.githubusercontent.com/61475969/206867324-41fc9557-2731-4bce-8b36-d4ba6c1ed0bd.png">

8.Created a Nat Gateway and assigned one of the Elastic IPs (*The other 2 was used by Bastion hosts) 
<img width="965" alt="Screenshot 2022-12-10 at 17 22 06" src="https://user-images.githubusercontent.com/61475969/206867357-48c0e98e-90b8-49a2-bc6f-932065f0d39c.png">

9. Created security groups for different services in the architecture

Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.

Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. (This is the IP address that you will use to SSH into the bastion servers). **On inbound rules, pick TCP (22) and choose my IP (just to prevent an attempt from a hacker to SSH into the bastion servers). **

Application Load Balancer: ALB will be available from the Internet

Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.

Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

<img width="997" alt="Screenshot 2022-12-10 at 17 23 04" src="https://user-images.githubusercontent.com/61475969/206867394-b97af95d-3dce-4f61-a2f5-518503fbf672.png">

Set Up Compute Resources for Nginx

Provision EC2 Instances for Nginx

Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region close to the target users. Use EC2 instance of T2 family (e.g. t2.micro or similar)

Ensure that it has the following software installed: python, ntp,net-tools,vim,wget,telnet,epel-release,htop.

Create an AMI out of this instance

Prepare Launch Template For Nginx (One Per Subnet)

1. Make use of the AMI to set up a launch template 

2. Ensure the Instances are launched into a public subnet (public subnet 1 or 2) 
<img width="961" alt="Screenshot 2022-12-10 at 17 24 09" src="https://user-images.githubusercontent.com/61475969/206867456-68e25741-0be2-46b2-b1b0-f68bf432bc40.png">

3. Assign appropriate security group (NGINX SG)

4. Configure Userdata to update yum package repository and install nginx

choose the instance name and stop it
click on actions >> instance settings >> edit userdata to create a userdata script
click on the userdata script to edit it

```
Content-Type: multipart/mixed; boundary="//"
 MIME-Version: 1.0

 --//
 Content-Type: text/cloud-config; charset="us-ascii"
 MIME-Version: 1.0
 Content-Transfer-Encoding: 7bit
 Content-Disposition: attachment; filename="cloud-config.txt"

 #cloud-config
 cloud_final_modules:
 - [scripts-user, always]

 --//
 Content-Type: text/x-shellscript; charset="us-ascii"
 MIME-Version: 1.0
 Content-Transfer-Encoding: 7bit
 Content-Disposition: attachment; filename="userdata.txt"

 #!/bin/bash
 yum update -y
 yum install nginx -y
 --//--
 ```
- click on the save button
- start the instance


Configure Target Groups

Go to Target Groups section and create a new target group.

Select Instances as the target type
Ensure the protocol HTTPS on secure TLS port 443
Ensure that the health check path is /healthstatus
Register Nginx Instances as targets
Ensure that health check passes for the target group

<img width="808" alt="Screenshot 2022-12-10 at 17 26 55" src="https://user-images.githubusercontent.com/61475969/206867562-bd10c06e-5778-4c2e-ac46-2cd9d0394548.png">

Configure Autoscaling For Nginx

1. Select the right launch template 

<img width="808" alt="Screenshot 2022-12-10 at 18 01 36" src="https://user-images.githubusercontent.com/61475969/206868977-3a6f6fbb-e76e-495f-b0f4-47600ce75cf6.png">

2. Select the VPC

<img width="797" alt="Screenshot 2022-12-10 at 18 03 21" src="https://user-images.githubusercontent.com/61475969/206869056-f23c77bf-80c7-44e1-8273-49459fcf92f8.png">

3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
<img width="934" alt="Screenshot 2022-12-10 at 18 07 40" src="https://user-images.githubusercontent.com/61475969/206869247-6d553c2e-7222-4d85-a2ab-a5a3c76a0ba7.png">
5. Select the target group you created before
6. Ensure that you have health checks for both EC2 and ALB 
<img width="952" alt="Screenshot 2022-12-10 at 18 08 32" src="https://user-images.githubusercontent.com/61475969/206869287-0f258df1-380c-4631-b8c1-935cdf408150.png">

7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90% 

<img width="852" alt="Screenshot 2022-12-10 at 18 09 38" src="https://user-images.githubusercontent.com/61475969/206869327-a10ac9b8-b323-4893-8890-3e281f2a5c8b.png">

11. Ensure there is an SNS topic to send scaling notifications 
<img width="900" alt="Screenshot 2022-12-10 at 18 10 24" src="https://user-images.githubusercontent.com/61475969/206869351-2d4113e5-ca55-45e3-99f9-6453ac4d5f8c.png">

Set Up Compute Resources for Bastion

Provision the EC2 Instances for Bastion

Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
Ensure that it has the following software installed: python, ntp,net-tools,vim,wget,telnet,epel-release,htop.
Associate an Elastic IP with each of the Bastion EC2 Instances
Create an AMI out of the EC2 instance
Prepare Launch Template for Bastion (One Per Subnet)

Make use of the AMI to set up a launch template

Ensure the Instances are launched into a private subnet (private subnet 1 or 2)

Assign appropriate security group (Bastion SG)

Configure Userdata to update yum package repository and install ansible and git

choose the instance name and stop it
click on actions >> instance settings >> edit userdata to create a userdata script
click on the userdata script to edit it

```
Content-Type: multipart/mixed; boundary="//"
 MIME-Version: 1.0

 --//
 Content-Type: text/cloud-config; charset="us-ascii"
 MIME-Version: 1.0
 Content-Transfer-Encoding: 7bit
 Content-Disposition: attachment; filename="cloud-config.txt"

 #cloud-config
 cloud_final_modules:
 - [scripts-user, always]

 --//
 Content-Type: text/x-shellscript; charset="us-ascii"
 MIME-Version: 1.0
 Content-Transfer-Encoding: 7bit
 Content-Disposition: attachment; filename="userdata.txt"

 #!/bin/bash
 yum update -y
 yum install ansible git  -y
 --//--
```

- click on the save button
- start the instance.

Configure Target Groups

Go to Target Groups section and create a new target group.

Select Instances as the target type
Ensure the protocol HTTPS on secure TLS port 443
Ensure that the health check path is /healthstatus
Register Bastion Instances as targets
Ensure that health check passes for the target group
Configure Autoscaling For Bastion

Select the right launch template
Select the VPC
Select both public subnets
Enable Application Load Balancer for the AutoScalingGroup (ASG)
Select the target group you created before
Ensure that you have health checks for both EC2 and ALB
The desired capacity is 2
Minimum capacity is 2
Maximum capacity is 4
Set scale out if CPU utilization reaches 90%
Ensure there is an SNS topic to send scaling notifications
Set Up Compute Resources for Webservers

Provision the EC2 Instances for Webservers

create 2 separate launch templates for both the WordPress and Tooling websites

Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).
N.B: I used userdata to install the apps in step 2 below:

At the **Configure Instance stage ** in creating an instance, scroll down to userdata session and insert the following:

```
Content-Type: multipart/mixed; boundary="//"
MIME-Version: 1.0

--//
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config.txt"

#cloud-config
cloud_final_modules:
- [scripts-user, always]

--//
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userdata.txt"

#!/bin/bash
yum update -y
yum install ntp-y
yum install net-tools -y
yum install vim -y
yum install wget -y
yum install telnet -y
yum install epel-release -y
yum install htop -y
yum install php -y
--//--
```

You wii tweak the script above to add ```yum install wordpress -y``` to the userdata script to install WordPress (for the wordpress instance ). 2. Ensure that it has the following software installed: python, ntp,net-tools,vim,wget,telnet,epel-release,htop,php.

Create an AMI out of the EC2 instance
Prepare Launch Template For Webservers (One per subnet)

Make use of the AMI to set up a launch template
Ensure the Instances are launched into a private subnet (private subnet 1 or 2)
Assign appropriate security group (Webservers SG)

STEP 3: TLS Certificates From Amazon Certificate Manager (ACM)

You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

Navigate to AWS ACM
Request a public wildcard certificate for the domain name you registered in Freenom
Use DNS to validate the domain name.
Tag the resource
Read the documentation here to learn how to validate DNS with Route 53

<img width="936" alt="Screenshot 2022-12-10 at 18 14 44" src="https://user-images.githubusercontent.com/61475969/206869508-55f6ee18-8fef-4781-abfb-2d3f93f67b23.png">

STEP 4: CONFIGURE APPLICATION LOAD BALANCER (ALB)

Application Load Balancer To Route Traffic To NGINX

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. This will allow us to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

1. Create an Internet facing ALB

2. Ensure that it listens on HTTPS protocol (TCP port 443)

3. Ensure the ALB is created within the appropriate VPC | AZ | Subnets 

<img width="704" alt="Screenshot 2022-12-10 at 18 15 47" src="https://user-images.githubusercontent.com/61475969/206869544-45d629f7-61fc-4fd9-b905-c65d477ccfe2.png">

4. Choose the Certificate from ACM 
5. Select Security Group 
<img width="962" alt="Screenshot 2022-12-10 at 18 17 42" src="https://user-images.githubusercontent.com/61475969/206869606-774483b4-dfa5-412a-bce4-e9370f7270cd.png">

6.Select Nginx Instances as the target group 

<img width="912" alt="Screenshot 2022-12-10 at 18 18 25" src="https://user-images.githubusercontent.com/61475969/206869631-ae2089a9-ba97-4446-a59d-79d38d30ada4.png">


Application Load Balancer To Route Traffic To Web Servers (this would be repeated for Wordpress webserver and tooling webservers)

Because of autoscaling, Nginx will not know about the new IP addresses, or the ones that get removed when there is a scale-out of the instances. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer since the webservers are within a private subnet, and we do not want direct access to them.

1.Create an Internal ALB
2.Ensure that it listens on HTTPS protocol (TCP port 443)
3.Ensure the ALB is created within the appropriate VPC | AZ | Subnets
4.Choose the Certificate from ACM
5.Select Security Group
6.Select webserver Instances as the target group
7.Ensure that health check passes for the target group

STEP 5:Setup EFS

1. Create an EFS filesystem
2. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer 
3. Associate the Security groups created earlier for data layer.
4. Create an EFS access point. (Give it a name and leave all other settings as default)
<img width="912" alt="Screenshot 2022-12-10 at 18 21 39" src="https://user-images.githubusercontent.com/61475969/206869741-c24e4dd6-700b-4a05-a0b5-5cc57ab69f8a.png">

STEP 6: Setup RDS

We need to create a KMS to encrypt our database as a security measure.

To ensure that our databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance.

1. Create a subnet group and add 2 private subnets (data Layer) 

<img width="912" alt="Screenshot 2022-12-10 at 18 23 09" src="https://user-images.githubusercontent.com/61475969/206869772-82996c7b-2d4c-4f5e-87be-79f96009b4d7.png">

2. Create an RDS Instance for mysql 8

<img width="606" alt="Screenshot 2022-12-10 at 18 23 55" src="https://user-images.githubusercontent.com/61475969/206869796-ead0b906-9f2a-42fa-8482-26778d429c65.png">

3. To satisfy our architectural diagram, select either Dev/Test or Production Sample Template. But to minimize AWS cost, selected the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment) 
<img width="606" alt="Screenshot 2022-12-10 at 18 24 37" src="https://user-images.githubusercontent.com/61475969/206869834-ea9607e4-2810-4fd8-9841-975c9c9fa784.png">

4. Configure other settings accordingly (For test purposes, most of the default settings are good to go).
5. Configure VPC and security (ensure the database is not available from the Internet)
6. Configure backups and retention
7. Encrypt the database using the KMS key created earlier
8. Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit). 

<img width="606" alt="Screenshot 2022-12-10 at 18 25 33" src="https://user-images.githubusercontent.com/61475969/206869886-caed57bd-fcb7-441f-bd72-7a011933aa5f.png">

STEP 6: Configuring DNS with Route53

You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name.

1. Create an alias record for the root domain and direct its traffic to the ALB DNS name. 
<img width="875" alt="Screenshot 2022-12-10 at 18 27 42" src="https://user-images.githubusercontent.com/61475969/206869963-55672cd5-7e58-4728-86d9-b376b21b4cae.png">

Create an alias record for tooling.com and direct its traffic to the ALB DNS name. 

<img width="1075" alt="Screenshot 2022-12-10 at 18 31 30" src="https://user-images.githubusercontent.com/61475969/206870121-7401d748-e203-47ad-b96a-d9a4f14cbcaf.png">



