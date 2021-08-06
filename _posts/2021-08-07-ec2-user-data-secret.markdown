---
title: "How to safely use sensitive data at EC2 launch?"
layout: post
date: 2021-08-06 09:00
image: /assets/secret-ec2-user-data/nicolas-hippert-J4eTN9GqhzI-unsplash.jpg
headerImage: false
tag:
- devops
- aws
- ec2
- secret
category: blog
author: karolfilipczuk
description: Safely usage of sensitive data (secrets) using Secret Manager at EC2 lanuch.
---
![Photo by Nicolas HIPPERT on Unsplash](../assets/secret-ec2-user-data/nicolas-hippert-J4eTN9GqhzI-unsplash.jpg)
<p class="bottom-caption">Photo by <a href="https://unsplash.com/@nhippert?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Nicolas HIPPERT</a> on <a href="https://unsplash.com/s/photos/safe?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></p>

Recently, I had a task to automate startup of custom systemd services when new EC2 is launched. User data script is great way to make any necessary changes in the system at EC2 launch. User data script is run as root, gives you pretty much all permissions you need. 

The only problem I have encountered was with one of the systemd service, which requires a password to be started. Since user data script can be retrieved from launched EC2 and the result of the script is stored in logs, it's against security best practices to put password as plain text in the script. The problem can be solved with AWS Secret Manager and AWS CLI command to retrieve secret. 

Prerequisite
============
* AWS account

Agenda
======

1. Create Secret in Secret Manger
2. Create EC2 Instance Role with Secret Read Access policy
3. Create EC2 instance and retrieve secret value at launch
4. Make it useful
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
> Remember to replace resource's ARN with your secret's ARN.

Click `Next: Tags -> Next: Review`, name the policy `SecretReadAccess` and hit `Create Policy`.

Go back to web browser tab with role creation process. Hit refresh arrows, search for `SecretReadAccess`, mark it and click `Next: Tags -> Next: Review`, name it `EC2InstanceRole` and hit `Create role`.
![Choose newly create role](../assets/secret-ec2-user-data/role_policy.png)
<p class="bottom-caption">Choose newly create role</p>

Create EC2 instance and retrieve secret value at launch
=========================
Alright! We have everything prepared to launch the instance and safely retrieve secret at launch using user-data.

Go to `EC2 -> Instances -> Launch instances`. Choose `Amazon Linux 2 AMI`, then `t2.micro` instance type and go to `Configuration Instance Details`.

Make sure that `Auto-assign Public IP` is enabled and choose the newly created role in `IAM role` field.
![Auto-assign Public IP and IAM role configuration](../assets/secret-ec2-user-data/ec2-ip-role.png)
<p class="bottom-caption">Auto-assign Public IP and IAM role configuration</p>

Scroll to the bottom where `Advanced Details` can be found and locate `User data` field. User data is a script which is executed at launch of EC2. It's commonly used to install any necessary software and sometimes you may want to use passwords, keys or any other sensitive data in user data. You should never put sensitive data directly in user data, since it can be visible in logs. Solution is to store sensitive data in Secret Manager and retrieve the secret value in user data.
\
Copy the script into user data. It will retrieve key/values from the secret.
```
#!/bin/bash
aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret
```
<p class="bottom-caption">User data to retrieve secrets key/values</p>

> Remember to replace region and secret-id parameters with your values.

Click `Review and Launch -> Launch` (add new key pair if you don't have existing one).
Go to the `EC2 -> Instances` and wait until the instance will be in `Running` state.
Now login to then instance. Click on instance, `Connect -> EC2 Instant Connect -> Connect`.
The logs of user data script can be found in `/var/log/cloud-init-output.log`
![Secret retrieved](../assets/secret-ec2-user-data/secret_get_1.png)
<p class="bottom-caption">Secret retrieved</p>

As you can see, we have successfully retrieved secret from Secret Manager. We have only printed the secret to prove that it can be done. Real world scenario would rather only use the secret, so it wouldn't be printed in logs.

Make it useful
=========================
Although we have successfully retrieved secret, it's not really useful in raw format. The most interesting part of the secret is value from key/value pair. I'll show you few user data scripts which will make more out of the secret retrieval.

### Get only secret string as text

```
#!/bin/bash
aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret --query SecretString --output text
```

Additional options `--query` and `--output` allows to only retrieve `SecretString` instead of all data returned by AWS CLI.

![Secret string](../assets/secret-ec2-user-data/secret_get_2.png)
<p class="bottom-caption">Secret string</p>


### Get value of key/value pair
```
#!/bin/bash
yum update -y
yum install -y jq
aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret --query SecretString --output text | jq -r .mySecretKey
```

Unfortunately, there is no easy way to extract value from key/value pairs in pure bash. You can always write a script which will achieve it, but it is much simpler to use [`jq`](https://stedolan.github.io/jq/). We use `yum` to update repositories and install `jq`. Then `jq` can be used on the secret to retrieve value from key/value pair by specifying the key.

![Value of key/value pair](../assets/secret-ec2-user-data/secret_get_3.png)
<p class="bottom-caption">Value of key/value pair</p>

### Get value of key/value pair and store it in environment variable

```
#!/bin/bash
yum update -y
yum install -y jq
myValue=$(aws secretsmanager get-secret-value --region us-east-1 --secret-id MySecret --query SecretString --output text | jq -r .mySecretKey)
echo $myValue > /var/log/echoSecret.txt
```

Until now, we have simply printed the retrieved secret. This time, the secret is stored in an environment variable and will not be visible in the script's log. To check that in fact environment variable has the desired value, I added command to `echo` the variable into `/var/log/echoSecret.txt`.

![Logs without secret printed](../assets/secret-ec2-user-data/secret_get_4_1.png)
<p class="bottom-caption">Logs without secret printed</p>

![Secret stored in file](../assets/secret-ec2-user-data/secret_get_4_2.png)
<p class="bottom-caption">Seceret stored in file</p>



Summary
====================================
Today we have successfully created new secret in AWS Secret Manager, granted secrets read access for EC2 instance and read secret at EC2 launch with user data script. The approach can be useful for using sensitive data at EC2 launch, for example: password/key for Linux systemd services.

Thanks for reading!