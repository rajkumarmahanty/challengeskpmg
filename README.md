# challengeskpmg
Challenges from KMPG Interview

Challenges 1 - A 3-tier environment is a common setup. Use a tool of your choosing/familiarity create these resources. Please remember we will not be judged on the outcome but more focusing on the approach, style and reproducibility.

AWS Cloud Native Tool - Cloud Formation Template 

Enviroment : 3 Tier - Web Application with Autoscaling and Availability Set enabled.

Architecture Diagram :



CloudFormation Template Structure and Explanation :


How to Deploy the CFT Templates for deployment of the 3 Tier - WebApp having NGNIX and mySql

Steps :

1. Get started and deploy this into my AWS account
2. You can launch this CloudFormation stack in your account:

Example using AWS CLI Command :

First Download this code into your workstation, make your own changes and make the prerequisites updates.
Your AWS account must have one VPC available to be created in the selected region.
Create Amazon EC2 key pair
Install a domain in Route 53.
Install a certificate (in your selected region & also one in us-east-1)
Next install AWS CLI in your workstation.

Upload the files in the "infrastructure" directory into to your own S3 bucket.

Eg. aws s3 cp --recursive infrastructure/ s3://cf-templates-19sg5y0d6d084-ap-southeast-1/
You can run the master.yaml file from your workstation.
Broken down into 2-Stages to avoid too much time consuming and a single process.
Run Stage1 first before running Stage2, since Stage2 require export variable
from Stage1. If you don't want to create Cloudfront, then you can avoid Stage2.

Stage1 (~ 25 - 35 minutes)
===========================
To create a environment :
aws cloudformation create-stack \
--stack-name <env> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-project1//master.yaml

To update a environment :
aws cloudformation update-stack \
--stack-name <env> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-project1//master.yaml

To delete a environment :
aws cloudformation delete-stack --stack-name <env>

<env> - Note :stack-name that can be used are (dev, staging, prod)


Stage2 (~ 35 - 45 minutes)
===========================
To create a environment :
aws cloudformation create-stack \
--stack-name <envCDN> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-project1//infrastructure//webapp-cdn.yaml

To update a environment :
aws cloudformation update-stack \
--stack-name <envCDN> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-project1//infrastructure//webapp-cdn.yaml

To delete a environment :
aws cloudformation delete-stack --stack-name <envCDN>

<envCDN> - Note :stack-name that can be used are (devCDN, stagingCDN, prodCDN)


Example :
aws cloudformation create-stack \
--stack-name dev \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-project1//master.yaml

aws cloudformation create-stack \
--stack-name devCDN \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//cloudformation-project1//infrastructure//webapp-cdn.yaml
	
Adjust the Auto Scaling parameters for ECS hosts and services
The Auto Scaling group scaling policy provided by default launches and maintains a cluster of hosts distributed across two Availability Zones (min: 2, max: 2, desired: 2).

As well as configuring Auto Scaling for the ECS hosts (your pool of compute), you can also configure scaling each individual ECS service. This can be useful if you want to run more instances of each container/task depending on the load or time of day (or a custom CloudWatch metric). To do this, you need to create AWS::ApplicationAutoScaling::ScalingPolicy within your service template.

Deploy multiple environments (e.g., dev, staging, production)
Deploy another CloudFormation stack from the same set of templates to create a new environment. The stack name provided when deploying the stack is prefixed to all taggable resources (e.g., EC2 instances, VPCs, etc.) so you can distinguish the different environment resources in the AWS Management Console.

Change the VPC or subnet IP ranges
This set of templates deploys the following network design:

Item	CIDR Range	Usable IPs	Description
VPC	10.0.0.0/16	65,536	The whole range used for the VPC and all subnets
Public Subnet 1	10.0.1.0/24	251	The public subnet in the first Availability Zone
Public Subnet 2	10.0.2.0/24	251	The public subnet in the second Availability Zone
Private Subnet 1	10.0.3.0/24	251	The private subnet in the first Availability Zone
Private Subnet 2	10.0.4.0/24	251	The private subnet in the second Availability Zone
You can adjust the following section of the master.yaml template:

# Update Domain Name
PMHostedZone:
  Default: "kasturicookies.com"
  Description: "Enter an existing Hosted Zone."
  Type: "String"

# Update Sub-domain
# Update Auto Scaling parameters (MIN,MAX,Desired)
dev:
  ASMIN: '2'
  ASMAX: '2'
  ASDES: '2'
  WEBDOMAIN: "dev.kasturicookies.com"
  CDNDOMAIN: "devel.kasturicookies.com"

staging:
  ASMIN: '2'
  ASMAX: '2'
  ASDES: '2'
  WEBDOMAIN: "staging.kasturicookies.com"
  CDNDOMAIN: "static.kasturicookies.com"

prod:
  ASMIN: '2'
  ASMAX: '5'
  ASDES: '2'
  WEBDOMAIN: "www.kasturicookies.com"
  CDNDOMAIN: "cdn.kasturicookies.com"

# Update Uploaded SSL ARN
CertARN: "arn:aws:acm:us-east-1:370888776060:certificate/eec1f4f2-2632-4d20-bd8a-fbfbcdb15920"

# CIDR ranges
VPC:
  Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub ${TemplateLocation}/infrastructure/webapp-vpc.yaml
      Parameters:
        PMServerEnv:          !Ref "PMServerEnv"
        PMVpcCIDR:            10.0.0.0/16
        PMPublicSubnet1CIDR:  10.0.1.0/24
        PMPublicSubnet2CIDR:  10.0.2.0/24
        PMPrivateSubnet1CIDR: 10.0.3.0/24
        PMPrivateSubnet2CIDR: 10.0.4.0/24

# DB Config
MyRDS:
  Type: "AWS::CloudFormation::Stack"
  DependsOn:
  - "MySecurityGroup"
  Properties:
    TemplateURL: !Sub "${PMTemplateURL}/webapp-rds.yaml"
    TimeoutInMinutes: '5'
    Parameters:
      DatabaseUser: "startupadmin"
      DatabasePassword: "xxxxxxxx"
      DatabaseName: !Sub "${AWS::StackName}db"
      DatabaseSize: '5'
      DatabaseEngine: "mysql"
      DatabaseInstanceClass: "db.t2.micro"

