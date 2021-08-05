---
title: "How to safely use sercrets at EC2 launch?"
layout: post
date: 2021-08-03 17:30
image: /assets/gitlab-aws/gitlab-aws-deploy.png
headerImage: false
tag:
- devops
- aws
- ec2
- secret
category: blog
author: karolfilipczuk
description: Safely usage of secrets at EC2 lanuch.
---
![Gitlab to AWS diagram](../assets/gitlab-aws/gitlab-aws-deploy.png)
<p class="bottom-caption">Secrets</p>

Prerequisite
============
* AWS account

Agenda
======

1. Create Secret in Secret Manger
2. Create AWS resources
3. Create programmatic user in AWS
4. Create pipeline and deployment package
5. Summary

Create a secret in Secret Manager
=========================
Start with creating a secret which we will later on for EC2 instance at launch.
`Secret Manager -> Store a new secret`
Then choose type `Other type of secrets` and specify key/value for new secret.
![New Secret](../assets/secret-ec2-user-data/new_secret.png)
<p class="bottom-caption">New Secret</p>
Click `Next`, then specify name for secret and click `Next` again.
Now you can configure automatic rotation. It's not necessary for this scenario, therefore leave default and click `Next`. Review details and click `Store`. 
You will be taken to screen showing all secrets, in my case it is just the one created now.
![List of stored secrets](../assets/secret-ec2-user-data/secret_list.png)
<p class="bottom-caption">List of stored secrets</p>

Create EC2 Instance Role with Secret Read Access policy
=========================
We already have secret safely stored within AWS Secret Manager. It can't be simply read by any resource. EC2 instance will not be able to read secret unless you will add read permissions to the instance role. 

Go to `IAM -> Roles -> Create Role`. Choose EC2 and click `Next: Permissions`. 
Then click `Create Policy`, it will open in separate web browser tab.
Click on tab `JSON` and paste the policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "secretsmanager:GetSecretValue"
            ],
            "Resource": "arn:aws:secretsmanager:us-east-1:215538436894:secret:MySecret-u7eM2x",
            "Effect": "Allow"
        }    
    ]
}
```
<p class="bottom-caption">Policy granting EC2 permission to read secret</p>
> Remember to replace resoruce's arn with your secret's arn.

Click `Next: Tags -> Next: Review`, name the policy `SecretReadAccess` and hit `Create Policy`.

Go back to web browser tab with role creation process. Hit refresh arrows, search for `SecretReadAccess`, mark it and click `Next: Tags -> Next: Review`, name it `EC2InstanceRole` and hit `Create role`.
![Choose newly create role](../assets/secret-ec2-user-data/role_policy.png)
<p class="bottom-caption">Choose newly create role</p>

Create EC2 instance and retrieve secret value at launch
=========================
Alright! We have everything prepared to launch instance and safely retrive secret at lanuch using user-data.

Go to `EC2 -> Instances -> Launch instances`. Choose `Amazon Linux 2 AMI`, then `t2.micro` instance type and go to `Configuration Instance Details`.

Make sure that `Auto-assign Public IP` is enabled and choose the newly created role in `IAM role` field.
![Auto-assing Public IP and IAM role configuration](../assets/secret-ec2-user-data/ec2-ip-role.png)
<p class="bottom-caption">Auto-assing Public IP and IAM role configuration</p>

Scroll to the bottom where `Advanced Details` can be found and locate `User data` field. User data is a script which is executed at launch of EC2. It's commonly used to install any necessary software and sometimes you may want to use passwords, keys or any other sensitive data in user data. You should never put sensitive data directly in user data, since it can be visible in logs. Solution is to store sensitve data in Secret Manager and retreive the secret value in user data.
\
Copy the script into user data. It will retrieve key/values from the secret.
```
#!/bin/bash
aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret
```
<p class="bottom-caption">User data to retreive secrets key/values</p>

> Remember to replace region and secret-id parameters with your values.

Click `Review and Launch -> Launch` (add new key pair if you don't have exisitng one).
Go to the `EC2 -> Instances` and wait until the instance will be in `Running` state.
Now login to then instance. Click on instance, `Connect -> EC2 Instant Connect -> Connect`.
The logs of user data script can be found in `/var/log/cloud-init-output.log`
![Secret retrieved](../assets/secret-ec2-user-data/secret_get_1.png)
<p class="bottom-caption">Secret retrieved</p>

As you can see, we have successfully retrieved seceret from Secret Manager. We have only printed the secret to proof that it can be done. Real world scenario would rather only use the secret, so it wouldn't be printed in logs.

Make it useful
=========================
Although we have successfully retrieved seceret, it's not really useful in raw format. Most interesting part of the secret is value from key/value pair. I'll show you few user data scripts which will make more out of the secret retrieval.

**Get only secret string as text**

```
#!/bin/bash
aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret --query SecretString --output text
```

Additional options `--query` and `--output` allows to only retrive `SecretString` instead of all data returned by AWS CLI.

![Secret string](../assets/secret-ec2-user-data/secret_get_2.png)
<p class="bottom-caption">Secret string</p>


**Get value of key/value pair**

```
#!/bin/bash
yum update -y
yum install -y jq
aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret --query SecretString --output text | jq -r .mySecretKey
```

Unfortunately there is no easy way to extract value from key/value pairs in pure bash. You can always write a script which will achive it, but it is much simpler to use `jq` (source)[https://stedolan.github.io/jq/]. We use `yum` to update repositores and install `jq`. Then `jq` can be used on the seceret to retrieve value from key/value pair by specifying the key.

![Value of key/value pair](../assets/secret-ec2-user-data/secret_get_3.png)
<p class="bottom-caption">Value of key/value pair</p>

**Get value of key/value pair and store it in environemnt variable**

```
#!/bin/bash
yum update -y
yum install -y jq
myValue=$(aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret --query SecretString --output text | jq -r .mySecretKey)
echo $myValue > /var/log/echoSecret.txt
```

Until now, we have simply printed the retrieved secret. This time the secret is stored in environment variable and will not be visible in script's log. To check that in fact environment variable has the desired value, I added command to `echo` the variabkle into `/var/log/echoSecret.txt`.

![Logs without secret printed](../assets/secret-ec2-user-data/secret_get_4_1.png)
<p class="bottom-caption">Logs without secret printed</p>

![Seceret stored in file](../assets/secret-ec2-user-data/secret_get_4_2.png)
<p class="bottom-caption">Seceret stored in file</p>



Summary
====================================


Thanks for reading!