# 3-tier-applications

# Part 0: Set up
we will be downloading the code from GitHub and uploading it to S3 so our instances can access it. We will also create an AWS Identity and Access Management EC2 role so we can use AWS Systems Manager Session Manager to connect to our instances securely and without needing to create SSH key pairs.

**Learning Objectives:**
- S3 Bucket Creation
- IAM EC2 Instance Role Creation
- Download Code from GitHub Repository

**1. Download the Code from GitHub**

Download the code from this repository into your local environment by running the command below. If you don't have git installed, you can just download the zip. Save it somewhere you can easily access.

`git clone https://github.com/aws-samples/aws-three-tier-web-architecture-workshop.git`

**2. S3 Bucket Creation**

1. Navigate to the S3 service in the AWS console and create a new S3 bucket.
2. Give it a unique name, and then leave all the defaults as in. Make sure to select the region that you intend to run this whole lab in. This bucket is where we will upload our code later.

**3. IAM EC2 Instance Role Creation**

1. Navigate to the IAM dashboard in the AWS console and create an EC2 role.
2. Select EC2 as the trusted entity.
3. When adding permissions, include the following AWS-managed policies. You can search for them and select them. These policies will allow our instances to download our code from S3 and use Systems Manager Session Manager to securely connect to our instances without SSH keys through the AWS console.

**AmazonSSMManagedInstanceCore**

**AmazonS3ReadOnlyAccess**

4. Give your role a name, and then click Create Role.

# Part 1: Networking and Security

We will build out the VPC networking components and security groups that will add a layer of protection around our EC2 instances, Aurora databases, and Elastic Load Balancers.

**Learning Objectives:**

Create an isolated network with the following components:
- VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT gateway
- Security Groups

**VPC and Subnets.**

**1. VPC Creation.**

- Navigate to the VPC dashboard in the AWS console and navigate to Your VPCs on the left-hand side.
- Make sure VPC only is selected, and fill out the VPC Settings with a Name tag and a CIDR range of your choice.

  *NOTE: Make sure you pay attention to the region you’re deploying all your resources in. You’ll want to stay consistent for this 
   workshop.*

   *NOTE: Choose a CIDR range that will allow you to create at least 6 subnets.(10.0.0.0/16)*
  
**2. Subnet Creation.**

- Next, create your subnets by navigating to Subnets on the left side of the dashboard and clicking Create Subnet.
- We will need six subnets across two availability zones. That means that three subnets will be in one availability zone, and three subnets will be in another zone. Each subnet in one availability zone will correspond to one layer of our three-tier architecture. Create each of the 6 subnets by specifying the VPC we created in part 1 and then choose a name, availability zone, and appropriate CIDR range for each of the subnets.

NOTE: It may be helpful to have a naming convention that will help you remember what each subnet is for. For example, in one AZ you might have the following: Public-Web-Subnet-AZ-1, Private-App-Subnet-AZ-1, Private-DB-Subnet-AZ-1.

NOTE: Remember, your CIDR range for the subnets will be subsets of your VPC CIDR range.

**Internet Connectivity**

**1. Internet Gateway**

- In order to give the public subnets in our VPC internet access we will have to create and attach an Internet Gateway. On the left-hand side of the VPC dashboard, select Internet Gateway.








