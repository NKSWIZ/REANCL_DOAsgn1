# DevOps Assignment 1 - (Candidate-a1NPhi7fgd)

## Problem Statement
  > Use CM tools such as Puppet, Ansible, or Chef to automate the installation of basic Drupal or WordPress. Setup a sample site. Automate the solution using Cloudformation template

This document details the following topics related to the assignment.

  - Solution description & Approach
  - Implementation Detail
  - Assumptions & limitations
  - Future Enhancements

#### Solution description & Approach
 This solution uses the combination of AWS Cloudformation and Puppet to implement the above-mentioned requirement. Cloudformation is used to automate the infrastructure part in AWS and Puppet to automate the installation of the WordPress in an EC2 instance.

#### Implementation Detail 
 ##### Base Image Creation
 Created an instance with ubuntu 16.04 and installed basic puppet modules required to install a WordPress site, with a dependency that the database name, database username, database password, database root password are to be supplied before building and deploying the puppet module. Building and deploying the module are to be done during the initial instance launch, Standalone Puppet approach is used for this implementation. A base image (AMIName - `base_img_for_wps_with_puppet` - AMIID `ami-b04433cf`) is created using this EC2 instance.

##### CloudFormation 
 Cloudformation creates a new VPC, Subnet, Route Tables, Internet Gateway, Routes, ACL, Security Group and spin up an EC2 instance based on the image `ami-b04433cf`. Cloudformation uses three templates one to create the infrastructure (`VPC Stack`) and one to create the EC2 instance (`EC2 Stack`) and creates a configuration file in the EC2 instance which is required to build the puppet module and the main stack (`Main Stack`) to call the other two templates to automate the whole implementation. Cloudformation stack takes these following input parameters. 
  - DBName
  - DBPassword
  - DBRootPassword
  - DBUser
  - InstanceType
  - KeyName
  - SSHLocation
  - SubnetCIDRBlock
  - VPCCIDRBlock
  - SecurityGroupId (required for EC2 template - no required in the UI)
  - SubnetId (required for EC2 template - no required in the UI)

The `Main Stack` takes all the input parameters and passes SSHLocation, VPCCIDRBlock, VPCCIDRBlock to the `VPC Stack`. 
`VPC Stack` creates a new VPC, Public Subnet, InternetGateway, RouteTable, Route, NetworkAcl and InstanceSecurityGroup, it also creates all the necessary routes and basic ACL and security group rules. 
once the infrastructure is creation is completed successfully `VPC Stack` returns VPC ID, SecurityGroupId and SubnetId as output parameters, the `Main Stack` captures these output values and passes DBName, DBPassword, DBRootPassword, DBUser, InstanceType, KeyName, SecurityGroupId and SubnetId to the `EC2 Stack`.
`EC2 Stack` creates an EC2 instance based off the image `ami-b04433cf`, it writes the configuration file `/home/ubuntu/MyModules/wordpress/manifests/conf.pp` using the DBName, DBPassword, DBRootPassword, DBUser parameters. then it runs the commands to build, deploy and execute the Puppet module to deploy the wordpress site. These steps are executed as part of the user data script using bash. `EC2 Stack` returns the WebsiteURL as output which is captured by the `Main Stack` and returned as WordpressSiteURL in stack output, which can be used to access the newly deployed wordpress site.    

#### Assumptions & Limitations
Following assumptions are made during the implementation with some scope limitations 
- This template can only be used in the US-Norther Virginia region
- AMI created is only available in US-Norther Virginia region
- It creates only one public subnet, as the requirement needs an instance to be deployed and need to be accessed in the internet, no private subnet in place as this specific solution doesn't need one. 
- It uses only the T2.nano and T2.micro instance types

#### Future Enhancements
Following improvements can be made to enhance the solution far more generic and reusable 
- Copy the AMI to other region and update the template mapping to point the new images in the regions 
- Update the template to support more instances types
- Use GIT to get the latest Puppet modules to make the solution more dynamic
