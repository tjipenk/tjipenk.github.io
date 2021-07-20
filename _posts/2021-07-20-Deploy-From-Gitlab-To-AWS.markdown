---
title: "Deploy from Gitlab to AWS EC2"
layout: post
date: 2021-07-20 17:30
image: /assets/gitlab-aws/gitlab-aws-deploy.png
headerImage: false
tag:
- devops
- gitlab
- aws
- ci/cd
- ec2
category: blog
author: karolfilipczuk
description: Use Gitlab CI pipeline for AWS EC2 deployment.
---
![Gitlab to AWS diagram](../assets/gitlab-aws/gitlab-aws-deploy.png)
<p class="bottom-caption">Deploy from Gitlab to AWS</p>

Nowadays everything is hosted in a cloud which make sense. Anybody can create environment from scratch in a blink of an eye, cloud provides flexibility and scalability, cloud providers make sure you have plenty of choice in terms of resoruces and they take over more and more maintenance duties from you. While all this is true, it is common to leverage cloud and at the same time use what you already been using until now.
\
\
In this blog post, I'll show you how to use Gitlab to deploy application on AWS.
\
\
We will provision EC2 instance in AWS, install NGINX and check that defult NGINX webpage is accessible and visible in web browser. Afterwards we will use CI/CD pipeline combining Gitlab CI and AWS Codedeploy to deploy another web page in the place of default NGINX web page.

Prerequisite
============

* Gitlab account
* AWS account

Agenda
======

1. Create Gitlab Pages/Plain HTML project
2. Create AWS resources
3. Create programmatic user in AWS
4. Create pipeline and deployment package
5. Summary

Create Gitlab Pages/Plain HTML project
=========================
Let's use one of the available templates to create new Gitlab project.
Click **`New project`** button and then **`From template -> Pages/Plain HTML`**.
\
This will create new project with example html and css file. 

Create AWS resources
=========================
Now let's setup necessary resources in AWS. You can choose two ways to provision resources: Cloudformation template or manual provision in AWS console. Luckly for you, I have already prepeared [Cloudformation template](https://gist.github.com/filip5114/bba06f31e3e1bb3e5a6e9e5672b46c46) which will provision following resources:
- VPC
- Internet Gateway associated with VPC
- Subnet
- Route Table 
	- with route to Internet Gateway
- Key Pair
- Security Group 
	- with ingress connection on ports 80
- EC2
	- with installed nginx, Codedeploy and Session Manager agents 
	- with custom instance role (alllows connection via Session Manager and Get, List operations on our S3 bucket)
- S3 Bucket with versioning enabled
- Codedeploy:
	- Role (uses managed policy AWSCodeDeployRole)
	- App
	- Group

Go to **`Cloudformation -> Create Stack -> With new resources (standard)`**. Choose **`Template is ready`** and upload the template.
\
\
![Create stack from existing template](../assets/gitlab-aws/createStack.png)
<p class="bottom-caption">Create stack from existing template</p>
Next set parameters defined in Cloudformation template.
- `Stack name` - any name
- `App name` - any name, will be part of tag for each resource
- `BucketName` - must be unique in AWS, only lowercase letters and number are allowed
- `SourceIpIngerss` - CIDR notation, will be used for security group ingress rules. Put you IP address if you want to restrict access. I'm using default, EC2 is accessible for evryone on port 80.
\
\
![Stack Parameters](../assets/gitlab-aws/stackParameters.png)
<p class="bottom-caption">Stack Parameters</p>
Remember to tick checkbox for Capabilities which is necessary in case you have IAM role defined in template.
\
![Stack Capabilities](../assets/gitlab-aws/stackCapabilities.png)
<p class="bottom-caption">Stack Capabilities</p>

Stack creation should take around 4 mins. Afterwards all mentioned resources should be already available. You should see last event with logical ID of your stack name in status **`CREATE_COMPLETE`**.
\
![Stack Events](../assets/gitlab-aws/stackEvents.png)
<p class="bottom-caption">Stack Events</p>

Go to **`Resoruce`** tab, find EC2 instance and click on hyperlink which will take you to instance view. Mark the EC2 instance and click connect.
\
![EC2 Connect](../assets/gitlab-aws/ec2Connect.png)
<p class="bottom-caption">EC2 Connect</p>

Then open **`Session Manager`** tab. The connect button should be active thanks to Cloudformation template which has handled enabling Session Manager connection. Session Manager connection allows to securly connect without SSH keys or bastion host to EC2 instance. 
\
\
Cloudformation template has done two steps to enable Session Manager connect:
- created instance role with necessary policies and attached it to the EC2
- installed Session Manager agent in EC2 instance using user-data
\
\
![EC2 Session Manager](../assets/gitlab-aws/ec2SessionManager.png)
<p class="bottom-caption">EC2 Session Manager</p>
Connect to EC2 and run following commands to check that Codedeploy and Nginx are installed and running:
```
sudo su - ec2-user
systemctl status nginx
systemctl status codedeploy-agent.service
```
![EC2 Connect](../assets/gitlab-aws/ec2Commands.png)
<p class="bottom-caption">EC2 Commands</p>
\
Check that default Nginx web page is accessible using EC2 Public IP address. Remember to copy/paste address. Using open address URL will default to https protocol, but Nginx page is available on http protocol.
![EC2 Public IP](../assets/gitlab-aws/ec2PublicId.png)
<p class="bottom-caption">EC2 Public IP</p>
\
You should see Nginx defult welocome web page.
![Nginx defualt web page](../assets/gitlab-aws/nginxDefaultPage.png)
<p class="bottom-caption">Nginx default web page</p>

We also needed Codedeploy Application and Deployment Group which were created by stack. Naviagte to **`Codedeploy -> Deploy -> Applications`** and you should see defined application.
\
![Codedeploy Application](../assets/gitlab-aws/codedeployApp.png)
<p class="bottom-caption">Codedeploy Application</p>

Click on **`Application Name`**, then on **`Deployment Group Name`** to check the details such as IAM Role assigned and which EC2 instances are targeted by the deployment group.
\
![Codedeploy Deployment Group](../assets/gitlab-aws/codedeployDeploymentGroup.png)
<p class="bottom-caption">Codedeploy Deployment Group</p>

Both Gitlab and AWS are done and ready. The last part is to integrate Gitlab with AWS to replace default Nginx page with Gitlab page created from Plain/HTML Pages template.

Create programmatic user in AWS
=========================
We need to have a way to acccess AWS resources directly from Gitlab CI pipeline. We do that using AWS programmatic user.
\
\
Go to `IAM -> Users -> Add Users`. Name the user and make sure you have checked **`Programmatic access`** checkbox. Then **`Attach exisiting policies directly`** and choose **`AmazonS3FullAccess`** and **`AWSCodeDeployDeployerAccess`**.

>WARNING: Those permissions give more access than necessery. For the real world scenarios please adjust permissions for your needs.

Create the user and note down **`Access Key ID`** and **`Secret Access Key`**. They can't be retrived again.
![Programmatic User Keys](../assets/gitlab-aws/userKeys.png)
<p class="bottom-caption">Programmatic User Keys</p>

Setup programmatic user in Gitlab
=========================
AWS part is done. We can now switch back to Gitlab.
\
Gitlab allows us to use AWS programmatic user in pipeline. Configuring the user on Gitlab side comes down to adding 3 variables.
\
\
Go to **`Settings -> CI/CD -> Variables`**.
\
Add variables:
- **`AWS_ACCESS_KEY_ID`** -  Your Access key ID
- **`AWS_SECRET_ACCESS_KEY`**	- Your Secret access key
- **`AWS_DEFAULT_REGION`** - Your region code (the region you have used to creat Cloudformation stack e.g. us-east-1)
\
\
>Make sure to use the exact names for variables. They are special variables and Gitlab is awere that they should be use as credentials for AWS.

![Gitlab AWS Variables](../assets/gitlab-aws/gitlabVars.png)
<p class="bottom-caption">Gitlab AWS Variables</p>

Create pipeline and deployment package
========================
Last missing piece is the Gitlab pipeline.
\
\
Default **`.gitlab-ci.yml`** comes with project template. We need to adjust it for our case, but before that we need to add few things to make our deplyment artifatcs compatible with AWS Codedeploy. Codedeploy is expecting a ZIP file containing:
- deployment artifacts
- appspec.yml ([appspec reference](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html))
- any script necessary to be executed before/during/after deployment

Deployment artifacts include two files: html and css stored in **`public`** directory.
**`appspec.yml`** we need to create and place in **`public`** directory. Create it and copy the code:
```
version: 0.0
os: linux
files:
  - source: /index.html
    destination: /usr/share/nginx/html/
  - source: /style.css
    destination: /usr/share/nginx/html/
hooks:
  BeforeInstall:
    - location: scripts/remove.sh
      timeout: 300
      runas: root
```
**`files`** section defines what files (source) should be copied to what destination.
\
**`hooks`** seciton define what script should be run in which stage. We only run script to remove old web page from Nginx before deploying artifacts.
\
\
Then create a **`remove.sh`** script as defined in **`appspec.yml`** in **`public/scripts/remove.sh`** and copy the code:
```
#!/bin/bash
rm -rf /usr/share/nginx/html/*
```
Finaly update **`.gitlab-ci.yml`** with the code:
```
image: registry.gitlab.com/gitlab-org/cloud-deploy/aws-base:latest

pages:
  stage: deploy
  script:
  - aws deploy push --application-name myApp --s3-location s3://namek999/artifacts.zip --source public/
  - aws deploy create-deployment --application-name myApp --deployment-group-name myApp-DeploymentGroup --s3-location bucket=namek999,bundleType=zip,key=artifacts.zip
  artifacts:
    paths:
    - public
  only:
  - master
```
Few changes were made:
- Change image to **`registry.gitlab.com/gitlab-org/cloud-deploy/aws-base:latest`**
	- It's special docker image maintained by Gitlab which has installed AWS CLI
- **`aws deploy push --application-name myApp --s3-location s3://namek777/artifacts.zip --source public/`**
	- create a zip from `public/` directory, assing deployment to application and push the zip file to S3 bucket
	- replace applciation name and S3 bucket name (you can find them in Cloudformation stack, Outputs tab)
- **`aws deploy create-deployment --application-name myApp --deployment-group-name myApp-DeploymentGroup --s3-location bucket=namek777,bundleType=zip,key=artifacts.zip`**
	- creates deployment in Codedeploy for specified application and deployment group
	- points to location of the zip deployment
	- replace application name, deployment group name and S3 bucket name (you can find them in Cloudformation stack, Outputs tab)

Commit **`.gitlab-ci.yml`** and go to **`CI/CD -> Pipelines`**. The pipeline will be trigged automatically after commit.
\
After it is done, go back to AWS **`Codedeploy -> Deploy -> Deployments`** and you should see completed deployment.
\
\
![Codedeploy Deployment](../assets/gitlab-aws/codedeployDeployment.png)
<p class="bottom-caption">Codedeploy Deployment</p>

Everything looks ok, then check what is now being served by Nginx. Navigate to EC2 public IP in web browser and you shoudl now see different web page.
\
\
![Gitlab Page](../assets/gitlab-aws/gitlabPage.png)
<p class="bottom-caption">Gitlab Page</p>


Summary
====================================
We have created pipeline which inetgeared Gitlab with AWS deployment.
For successful deployment from Gitlab to AWS you need:
- AWS programmatic user
- S3 Bucket for storing deployment packages
- Codedeploy Application and Deployment Group 
- Complete necessary setup to be compatible with Codedeploy (appspec.yml and scripts)

Thanks for reading!