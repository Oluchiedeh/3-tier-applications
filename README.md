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

NOTE: It may be helpful to have a naming convention that will help you remember what each subnet is for. For example, in one AZ you might have the following: Public-Web-mantlesubnet-AZ-1, Private-App-mantlesubnet-AZ-1, Private-DB-mantlesubnet-AZ-1.

NOTE: Remember, your CIDR range for the subnets will be subsets of your VPC CIDR range (10.0.0.0/24).

**Internet Connectivity**

**1. Internet Gateway**

- In order to give the public subnets in our VPC internet access we will have to create and attach an Internet Gateway. On the left-hand side of the VPC dashboard, select Internet Gateway.
- Create your internet gateway by simply giving it a name and clicking Create internet gateway.
- After creating the internet gateway, attach it to the VPC that you create in the VPC and Subnet Creation step of the workshop. You have a couple of options on how to do this, either with the creation success message or the Actions drop-down.
- Then, select the correct VPC and click Attach Internet gateway.

**2. NAT Gateway**

- In order for our instances in the app layer private subnet to be able to access the internet they will need to go through a NAT Gateway. For high availability, you’ll deploy one NAT gateway in each of your public subnets. Navigate to NAT Gateways on the left side of the current dashboard and click Create NAT Gateway.
- Fill in the Name, choose one of the public subnets you created in part 2, and then allocate an Elastic IP. Click Create NAT gateway.
- Repeat steps 1 and 2 for the other subnet.

**Routing Configuration.**

- Navigate to Route Tables on the left side of the VPC dashboard and click Create route table First, let’s create one route table for the web layer public subnets and name it accordingly.
- Edit the Explicit Subnet Associations of the route table by navigating to the route table details again. Select Subnet Associations and click Edit subnet associations.
- Select the two web layer public subnets you created earlier and click Save associations.
- Now create 2 more route tables, one for each app layer private subnet in each availability zone. These route tables will route app layer traffic destined for outside the VPC to the NAT gateway in the respective availability zone, so add the appropriate routes for that.
- Once the route tables are created and routes added, add the appropriate subnet associations for each of the app layer private subnets.

**Security Groups.**

- Security groups will tighten the rules around which traffic will be allowed to our Elastic Load Balancers and EC2 instances. Navigate to Security Groups on the left side of the VPC dashboard, under Security.
- The first security group you’ll create is for the public, internet-facing load balancer. After typing a name and description, add an inbound rule to allow HTTP-type traffic for Anywhere-IPv4.
- The second security group you’ll create is for the public instances in the web tier. After typing a name and description, add an inbound rule that allows HTTP-type traffic from your internet-facing load balancer security group you created in the previous step. This will allow traffic from your public-facing load balancer to hit your instances. Then, add an additional rule that will allow HTTP-type traffic for your IP. This will allow you to access your instance when we test.
- The third security group will be for our internal load balancer. Create this new security group and add an inbound rule that allows HTTP-type traffic from your public instance security group. This will allow traffic from your web tier instances to hit your internal load balancer.
- The fourth security group we’ll configure is for our private instances. After typing a name and description, add an inbound rule that will allow TCP-type traffic on port 4000 from the internal load balancer security group you created in the previous step. This is the port our app tier application is running on and allows our internal load balancer to forward traffic on this port to our private instances. You should also add another route for port 4000 that allows your IP for testing.
- The fifth security group we’ll configure protects our private database instances. For this security group, add an inbound rule that will allow traffic from the private instance security group to the MYSQL/Aurora port (3306).

# Part 2: Database Deployment

This section of the workshop will walk you through deploying the database layer of the three-tier architecture.

**Learning Objectives:**

- Deploy Database Layer
- Subnet Groups
- Multi-AZ Database

**Subnet Groups**

- Navigate to the RDS dashboard in the AWS console and click on Subnet groups on the left-hand side. Click Create DB subnet group.
- Give your subnet group a name, description, and choose the VPC we created.
- When adding subnets, make sure to add the subnets we created in each availability zone specifically for our database layer. You may have to navigate back to the VPC dashboard and check to make sure you're selecting the correct subnet IDs.

**Database Deployment**

- Navigate to Databases on the left-hand side of the RDS dashboard and click Create database.
- We'll now go through several configuration steps. Start with a Standard create for this MySQL-Compatible Amazon Aurora database. Leave the rest of the defaults in the Engine options as default.
- Under the Templates section choose Dev/Test since this isn't being used for production at the moment. Under Settings set a username and password of your choice and note them down since we'll be using password authentication to access our database.
- Next, under Availability and durability change the option to create an Aurora Replica or reader node in a different availability zone. Under Connectivity, set the VPC, choose the subnet group we created earlier, and select no for public access.
- Set the security group we created for the database layer, make sure password authentication is selected as our authentication choice, and create the database.
- When your database is provisioned, you should see a reader and writer instance in the database subnets of each availability zone. Note down the writer endpoint for your database for later use.

# Part 3: App Tier Instance Deployment.

In this section of our workshop we will create an EC2 instance for our app layer and make all necessary software configurations so that the app can run. The app layer consists of a Node.js application that will run on port 4000. We will also configure our database with some data and tables.

- Learning Objectives:
- Create App Tier Instance
- Configure Software Stack
- Configure Database Schema
- Test DB connectivity

**App Instance Deployment.**

- Navigate to the EC2 service dashboard and click on Instances on the left-hand side. Then, click Launch Instances.
- Select the first Amazon Linux 2 AMI.
- We'll be using the free tier eligible T.2 micro instance type. Select that and click Next: Configure Instance Details.
- When configuring the instance details, make sure to select to correct Network, subnet, and IAM role we created. Note that this is the app layer, so use one of the private subnets we created for this layer.
- We'll be keeping the defaults for storage so click next twice. When you get to the tag screen input a Name as a key and call the instance AppLayer. It's a good idea to tag your instances so you can easily keep track of what each instance was created for. Click Next: Configure Security Group.
- Earlier we created a security group for our private app layer instances, so go ahead and select that in this next section. Then click Review and Launch. Ignore the warning about connecting to port 22- we don't need to do that.
- When you get to the Review Instance Launch page, review the details you configured and click Launch. You'll see a pop up about creating a key pair. Since we are using Systems Manager Session Manager to connect to the instance, proceed without a keypair. Click Launch.
- You'll be taken to a page where you can click launch instance, and you'll see the instance you just launched.

**Connect to Instance.**

- Navigate to your list of running EC2 Instances by clicking on Instances on the left hand side of the EC2 dashboard. When the instance state is running, connect to your instance by clicking the checkmark box to the left of the instance, and click the connect button on the top right corner of the dashboard.Select the Session Manager tab, and click connect. This will open a new browser tab for you.
*NOTE: If you get a message saying that you cannot connect via session manager, then check that your instances can route to your NAT gateways and verify that you gave the necessary permissions on the IAM role for the Ec2 instance.*
- When you first connect to your instance like this, you will be logged in as ssm-user which is the default user. Switch to ec2-user by executing the following command in the browser terminal:
  
   `sudo -su ec2-user`

- Let’s take this moment to make sure that we are able to reach the internet via our NAT gateways. If your network is configured correctly up till this point, you should be able to ping the Google DNS servers:

  `ping 8.8.8.8`
  
- You should see a transmission of packets. Stop it by pressing cntrl c.

NOTE: If you can’t reach the internet then you need to double-check your route tables and subnet associations to verify if traffic is being routed to your NAT gateway!

**Configure Database.**

- Start by downloading the MySQL CLI:
  `sudo yum install mysql -y`
`wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm`
`sudo yum localinstall mysql57-community-release-el7-8.noarch.rpm`
`sudo yum install mysql-community-server`
`sudo rpm --importhttps://repo.mysql.com/RPM-GPG-KEY-MYSQL-2022`
`sudo yum install mysql`

- Initiate your DB connection with your Aurora RDS writer endpoint. In the following command, replace the RDS writer endpoint and the username, and then execute it in the browser terminal:
  `mysql -h CHANGE-TO-YOUR-RDS-ENDPOINT -u CHANGE-TO-USER-NAME -p`

  `mysql -h database-1-instance-1.c0ophmqpt70s.us-east-1.rds.amazonaws.com -u admin -p`

- You will then be prompted to type in your password. Once you input the password and hit enter, you should now be connected to your database.

NOTE: If you cannot reach your database, check your credentials and security groups.

- Create a database called webappdb with the following command using the MySQL CLI:

   `CREATE DATABASE webappdb`

- You can verify that it was created correctly with the following command:

   `SHAOW DATABASE`

- Create a data table by first navigating to the database we just created:

   `USE webappdb`

- Then, create the following transactions table by executing this create table command:

  `CREATE TABLE IF NOT EXISTS transactions(id INT NOT NULL
   AUTO_INCREMENT, amount DECIMAL(10,2), description
   VARCHAR(100), PRIMARY KEY(id))`

- Verify the table was created:

  `SHOW TABLES`

- Insert data into table for use/testing later:

  `INSERT INTO transactions (amount,description) VALUES ('400','groceries')`

- Verify that your data was added by executing the following command:
  
  `SELECT * FROM transactions`
  
- When finished, just type exit and hit enter to exit the MySQL client.    






















