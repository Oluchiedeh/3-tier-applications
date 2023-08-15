# 3-tier-applications

# Part 0: Set up
we will be downloading the code from GitHub and uploading it to S3 so our instances can access it. We will also create an AWS Identity and Access Management EC2 role so we can use AWS Systems Manager Session Manager to connect to our instances securely and without needing to create SSH key pairs.

**Learning Objectives:**
- S3 Bucket Creation
- IAM EC2 Instance Role Creation
- Download Code from GitHub Repository

**Download the Code from GitHub**

Download the code from this repository into your local environment by running the command below. If you don't have git installed, you can just download the zip. Save it somewhere you can easily access.

`git clone https://github.com/aws-samples/aws-three-tier-web-architecture-workshop.git`

**S3 Bucket Creation**
1. Navigate to the S3 service in the AWS console and create a new S3 bucket.
2. Give it a unique name, and then leave all the defaults as in. Make sure to select the region that you intend to run this whole lab in. This bucket is where we will upload our code later.

**IAM EC2 Instance Role Creation**
1. Navigate to the IAM dashboard in the AWS console and create an EC2 role.
2. Select EC2 as the trusted entity.
3. When adding permissions, include the following AWS-managed policies. You can search for them and select them. These policies will allow our instances to download our code from S3 and use Systems Manager Session Manager to securely connect to our instances without SSH keys through the AWS console.

**AmazonSSMManagedInstanceCore**

**AmazonS3ReadOnlyAccess**

4. Give your role a name, and then click Create Role.

# Part 1: Networking and Security





