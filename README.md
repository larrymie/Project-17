# Project-17
**AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 2

![1](https://user-images.githubusercontent.com/91284177/165767533-7a75f2f1-24cd-496a-b642-64f3af5bdb41.png)
I the previous (project 16) i started automating the diagram above by provisioning the VPC and 2 public subnets.  

I now Created 4 private subnets keeping in mind following principles:

Made sure i use variables or length() function to determine the number of AZs
Used variables and cidrsubnet() function to allocate vpc_cidr for subnets
Kept variables and resources in separate files for better code structure and readability
Tagged all the resources i have created so far. Explored how to use format() and count functions to automatically tag subnets with its respective number.

**Tagging

 I added multiple tags as a default set. for example, in my terraform.tfvars file i had my default tags defined
 ![02](https://user-images.githubusercontent.com/91284177/165769442-e256147b-b820-4322-bdcd-1a654fbbed01.png)
Tagged all my resources using the format below in my variables.tf file, The nice thing about this is – anytime i need to make a change to the tags, i will simply do that in one single place (terraform.tfvars) IMAGE 03

![03](https://user-images.githubusercontent.com/91284177/165770045-5d311527-7469-4776-a5ad-65aad9138d92.png)

**Internet Gateways & format() function

I created an Internet Gateway in a separate Terraform file internet_gateway.tf IMAGE 04

![04](https://user-images.githubusercontent.com/91284177/165770841-7f6bb415-5c10-47c8-863d-8c747e0080b1.png)

**NAT Gateways

I created 1 NAT Gateways and 1 Elastic IP (EIP) addresses in a new file called natgateway.tf. IMAGE 05
![05](https://user-images.githubusercontent.com/91284177/165771623-bc36fa3c-638e-4c2d-bf17-646137837311.png)

**AWS ROUTES

I created a file called route_tables.tf and use it to create routes for both public and private subnets, created the below resources. Ensured they are properly tagged. IMAGE 06

![06](https://user-images.githubusercontent.com/91284177/165772109-e085d24c-890b-4e91-ac0c-5b8f01a8a7e6.png)


I  ran terraform plan and terraform apply it added the following resources to AWS in multi-az set up:

– Our main vpc
– 2 Public subnets
– 4 Private subnets
– 1 Internet Gateway
– 1 NAT Gateway
– 1 EIP
– 2 Route tables

Now completed Networking part of AWS set up. I moved on to Compute and Access Control configuration automation using Terraform 

**AWS Identity and Access Management

IAM and Roles
I passed an IAM role to EC2 instances to give them access to some specific resources, so icreated the following:

**Created AssumeRole IMAGE 07
![07](https://user-images.githubusercontent.com/91284177/165773251-d3a06759-351c-40a1-85da-99b24c089215.png)

**Created IAM policy for this role

This is where we i defined a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform action describe applied to EC2 instances: IMAGE 08

![08](https://user-images.githubusercontent.com/91284177/165773901-cc34e274-29bc-4308-ab6f-7df4160b966d.png)

Attached the Policy to the IAM Role

    resource "aws_iam_role_policy_attachment" "test-attach" {
        role       = aws_iam_role.ec2_instance_role.name
        policy_arn = aws_iam_policy.policy.arn
    }
Created an Instance Profile and interpolate the IAM Role
    resource "aws_iam_instance_profile" "ip" {
        name = "aws_instance_profile_test"
        role =  aws_iam_role.ec2_instance_role.name
    }


Now done with Identity and Management part for now, created other resources required.

Resources created afterward
As per our architecture we need to do the following:

**Create Security Groups
Create Target Group for Nginx, WordPress and Tooling
Create certificate from AWS certificate manager
Create an External Application Load Balancer and Internal Application Load Balancer.
create launch template for Bastion, Tooling, Nginx and WordPress
Create an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
Create Elastic Filesystem
Create Relational Database (RDS)
Let us create some Terraform configuration code to accomplish these tasks


**CREATE SECURITY GROUPS

I created all the security groups in a single file, refrenced this security group within each resources that needs it.
I started by creating a file and named it security.tf. wrote script as followed IMAGE 09
![09](https://user-images.githubusercontent.com/91284177/165775664-65f3310d-d8ae-46d3-9752-48fa03a55fe7.png)

**CREATED CERTIFICATE FROM AMAZON CERIFICATE MANAGER

Created cert.tf file and add the following code snippets to it. larrymie.click was created using via aws route53. the certificate and others were generated as seen below IMAGE 10
![10](https://user-images.githubusercontent.com/91284177/165776661-de7f2a49-5f19-4f1c-b3f9-c720dbd436b5.png)


Created an external (Internet facing) Application Load Balancer (ALB) also created a file called alb.tf

First of all, i created the ALB, then created the target group and lastly created the listener rule. IMAGE 11

![11](https://user-images.githubusercontent.com/91284177/165777323-128e4ced-bc64-42b3-bdca-28d825999962.png)

To inform my ALB to where to route the traffic i created a Target Group to point to its targets: IMAGE12

![12](https://user-images.githubusercontent.com/91284177/165777860-1487b97f-3b9a-4b9e-bfcb-6bec8c537608.png)

I then created a Listner for this target Group. IMAGE13

![13](https://user-images.githubusercontent.com/91284177/165778362-73e6796b-8cf3-4cd5-b195-6cdb8f14cffa.png)

Added the following outputs to output.tf to print them on screen IMAGE14

![14](https://user-images.githubusercontent.com/91284177/165778723-842c4593-1ea5-4bc4-8c31-e10ab77221d6.png)

**Created an Internal (Internal) Application Load Balancer (ALB)
 The same concepts imfollowed for external LB was followed with the internal load balancer. IMAGE 15

![15](https://user-images.githubusercontent.com/91284177/165779358-954fadd1-2211-418b-8698-287aec60a832.png)

To inform my ALB to where to route the traffic i need to create a Target Group to point to its targets: IMAGE16

![16](https://user-images.githubusercontent.com/91284177/165779930-b317323a-d6e6-42ff-9e07-6441aef19308.png)

Also created a Listner for this target Group (for Wordpress & Tooling) IMAGE17

![17](https://user-images.githubusercontent.com/91284177/165780499-20cbdaa0-f37f-4c17-8db4-45d2a8e606f9.png)


**CREATING AUTOSCALING GROUPS

Firstly, i created asg-bastion-nginx.tf and sns_topics for 4 autoscaling groups IMAGE 18

![18](https://user-images.githubusercontent.com/91284177/165781196-cb27ea95-0814-432a-9059-ae458596c90b.png)

**Launched template for bastion IMAGE 19
![19](https://user-images.githubusercontent.com/91284177/165782193-f00eb169-b523-4075-ac87-1d855d6e8681.png)


Created asg-wordpress-tooling.tf IMAGE 20

![20](https://user-images.githubusercontent.com/91284177/165782510-a0390558-fc00-41f4-b402-ab06249821d9.png)

**STORAGE AND DATABASE

Created Elastic File System (EFS)
In order to create an EFS i first created a KMS key. IMAGE 21

![21](https://user-images.githubusercontent.com/91284177/165783273-a5fc502b-89e1-4f93-8070-4dbcf1a00022.png)

AWS Key Management Service (KMS) makes it easy to create and manage cryptographic keys and control their use across a wide range of AWS services and applications.

I then created EFS and it mount targets- added the following code to efs.tf. IMAGE 22
![22](https://user-images.githubusercontent.com/91284177/165783775-d85875d2-cb0b-4cf3-8ce5-14974d8ca9ca.png)

Create MySQL RDS using this snippet of code in rds.tf file IMAGE 23
![23](https://user-images.githubusercontent.com/91284177/165784087-6537425e-cdd9-42b8-9576-35d6f56949d1.png)

Before applying Terraform i updated my variable.tf file to avoid Terraform failure. IMAGE 24

![24](https://user-images.githubusercontent.com/91284177/165784895-3b49055e-5fc1-44e4-8872-617bf0326dd2.png)
 
 Also updated my terraform.tfvars file and added the following codes. IMAGE 25
 ![25](https://user-images.githubusercontent.com/91284177/165785357-dc859af1-edba-43dd-b266-2ae09823a227.png)

I finally ran Terraform apply on my terminal, checked in my aws vpc dashboard to confirm if all tyhe resources were successfully provissioned IMAGE 26
 ![26](https://user-images.githubusercontent.com/91284177/165785885-829536a3-f2ec-479e-9f83-bd340cc850d0.png)

 









