# challengeskpmg
Challenges from KMPG Interview

Challenges 1 - A 3-tier environment is a common setup. Use a tool of your choosing/familiarity create these resources. Please remember we will not be judged on the outcome but more focusing on the approach, style and reproducibility.

AWS Cloud Native Tool - Cloud Formation Template 

Enviroment : 3 Tier - Web Application with Autoscaling and Availability Set enabled ,AWS WAF enabled for Web access Protection from DDOS,SQLInjections etc.

### Architecture Diagram :
[!Architecture-Diagram](images/AWS_WebApp_Design - 2.png)

## CloudFormation Template Structure Prerequisites and Explanation :

### Prerequisites :

The Cloudformation Security Group IP address is open by default (testing purpose). Please update the Security Group Access with your own IP Address to ensure your instances security.

Before deploying the templates, you would need the following:

- Your AWS account must have one VPC available to be created in the selected region
- Amazon EC2 key pair
- Installed Domain in Route 53.
- Installed Certificate (in us-east-2 & also one in us-east-1) || if you need any other region then add the appropiate region to the mainbase.yaml file and install the certificate for that region

### Cloud Formation Template Details

| Template | Description |
| --- | --- | 
| [mainbase.yaml](mainbase.yaml) | This is the main master template - deploy it to CloudFormation and it includes all of the nested templates automatically. |
| [infracode-cft/webapp-iam.yaml](infracode-cft/webapp-iam.yaml) | This template creates policy to allow EC2 instance full access to S3 & CloudWatch, and VPC Logs to CloudWatch. |
| [infracode-cft/webapp-s3bucket.yaml](infracode-cft/webapp-s3bucket.yaml) | This template deploys the Backup Data Bucket with security data at rest and archive objects greater than 60 days, ELB logging, and Webhosting static content. |
| [infracode-cft/webapp-vpc.yaml](infracode-cft/webapp-vpc.yaml) | This template deploys a VPC with a pair of public and private subnets spread across two Availability Zones. It deploys an [Internet gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html), with a default route on the public subnets. It deploys 2 [NAT gateways](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-comparison.html), and default routes for them in the private subnets. |
| [infracode-cft/webapp-securitygroup.yaml](infracode-cft/webapp-securitygroup.yaml) | This template contains the [security groups](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html) and [Network ACLs](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) required by the entire stack. |
| [infracode-cft/webapp-rds.yaml](infracode-cft/webapp-rds.yaml) | This template deploys a (Mysql) Relational Database Service. |
| [infracode-cft/webapp-elb-appserver.yaml](infracode-cft/webapp-elb-appserver.yaml) | This template deploys an Application Load Balancer that exposes our PHP APP services. |
| [infracode-cft/webapp-autoscaling-appserver.yaml](infracode-cft/webapp-autoscaling-appserver.yaml) |This template deploys an ECS cluster to the private subnets using an Auto Scaling group. |
| [infracode-cft/webapp-elb-webserver.yaml](infracode-cft/webapp-elb-webserver.yaml) | This template deploys a Webserver Load Balancer that exposes our Nginx Proxy services. |
| [infracode-cft/webapp-autoscaling-webserver.yaml](infracode-cft/webapp-autoscaling-webserver.yaml) | This template deploys an ECS cluster to the private subnets using an Auto Scaling group. |
| [infracode-cft/webapp-cdn.yaml](infracode-cft/webapp-cdn.yaml) | Run this template separately. CDN Deployment is time consuming ~(30-40min to complete deploy). |
| [infracode-cft/webapp-cloudwatch.yaml](infracode-cft/webapp-cloudwatch.yaml) | This template deploys Cloudwatch Alert Service, CPU Utilization Alarm. |
| [infracode-cft/webapp-route53.yaml](infracode-cft/webapp-route53.yaml) | This template deploys Route 53 recordset to update ELB Alias and CNAME CDN. |
| [infracode-cft/webapp-waf.yaml](infracode-cft/webapp-waf.yaml) | This template deploys AWS WAF Secure Automation to deploy AWS WAF web ACL with eight preconfigured. |

## How to Deploy the CFT Templates for deployment of the 3 Tier - WebApp having NGNIX and mySql with AWS WAF Protection

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

Upload the files in the "infracode" directory into to your own S3 bucket.
You can run the mainbase.yaml file from your workstation.

Break the Deployment into 2-Stages : 
Run Stage1 first before running Stage2, since Stage2 require export variable from Stage1. If you don't want to create Cloudfront, then you can avoid Stage2.

Stage1
===========================
To create a environment :
aws cloudformation create-stack \
--stack-name <env> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//challeneskpmg//mainbase.yaml

To update a environment :
aws cloudformation update-stack \
--stack-name <env> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//challeneskpmg//mainbase.yaml

To delete a environment :
aws cloudformation delete-stack --stack-name <env>

<env> - Note :stack-name that can be used are (staging, prod)


Stage2
===========================
To create a environment :
aws cloudformation create-stack \
--stack-name <envCDN> \
--capabilities=CAPABILITY_IAM \
--template-body file:////path_to_template//challeneskpmg//infracode-cft//webapp-cdn.yaml

<envCDN> - Note :stack-name that can be used are (stagingCDN, prodCDN)

Adjust the Auto Scaling parameters for ECS hosts and services
The Auto Scaling group scaling policy provided by default launches and maintains a cluster of hosts distributed across two Availability Zones (min: 2, max: 2, desired: 2).

As well as configuring Auto Scaling for the ECS hosts (your pool of compute), you can also configure scaling each individual ECS service. This can be useful if you want to run more instances of each container/task depending on the load or time of day (or a custom CloudWatch metric). To do this, you need to create AWS::ApplicationAutoScaling::ScalingPolicy within your service template.

Deploy multiple environments (e.g., staging, production)
Deploy another CloudFormation stack from the same set of templates to create a new environment. The stack name provided when deploying the stack is prefixed to all taggable resources (e.g., EC2 instances, VPCs, etc.) so you can distinguish the different environment resources in the AWS Management Console.
  
### VPC Design and Assumptions

Change the VPC or subnet IP ranges ( IP Ranges taken are imaginery and not related to any Enterprise Allocation ) 
This set of templates deploys the following network design:

| Item | CIDR Range | Usable IPs | Description |
| --- | --- | --- | --- |
| VPC | 10.2.0.0/16 | 65,536 | The whole range used for the VPC and all subnets |
| Public Subnet 1 | 10.2.1.0/24 | 251 | The public subnet in the first Availability Zone |
| Public Subnet 2 | 10.2.2.0/24 | 251 | The public subnet in the second Availability Zone |
| Private Subnet 1 | 10.2.3.0/24 | 251 | The private subnet in the first Availability Zone |
| Private Subnet 2 | 10.2.4.0/24 | 251 | The private subnet in the second Availability Zone |

You can adjust the following section of the master.yaml template:

You can update Domain Name ( CH1HostedZone ),Sub-Domain,AutoScaling Parameters,SSL ARN,CIDR Ranges and DB Config in mainbase.yaml as per your need

