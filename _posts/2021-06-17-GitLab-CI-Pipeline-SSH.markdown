---
title: "GitLab CI Pipeline. Run Script via SSH to Remote Server"
layout: post
date: 2021-06-18 12:40
image: /assets/images/markdown.jpg
headerImage: false
tag:
- devops
- gitlab
- ssh
- ci/cd
category: blog
author: karolfilipczuk
description: How to setup SSH connection from GitLab to remote server and run script in pipeline.
---
![Photo by Jantine Doornbos on Unsplash](https://cdn-images-1.medium.com/max/1600/0*DzEDp3kusk5qSsnS)
<p class="bottom-caption">Photo by <a href="https://unsplash.com/@jantined">Jantine Doornbos</a> on <a href="https://unsplash.com">Unsplash</a></p>

When you think about deploying to remote server, SSH is first network protocol which comes to your mind. Adding on top GitLab CI/CD will let you take advantage of automation. To use GitLab CI/CD pipeline together with SSH connections it is necessary to firstly configure GitLab and I would like to show you how to configure it and run simple script.

Prerequisite
============

* GitLab account
* Remote server (I’m using Azure’s Linux VM)

Agenda
======

1. Create new GitLab project
2. Create and add SSH keys
3. Create and run GitLab CI/CD pipeline

Create new GitLab project
=========================

As a first step we will create GitLab project.  
Login into GitLab and navigate to `New project -> Create from template -> Pages/Plain HTML -> Use template`. Give it a project name and hit `Create project`. This will create a simple plain html project.

![Pages/Plain HTML project](../assets/2021-06-17-GitLab-CI-Pipeline-SSH/pages_html_project.png)
<p class="bottom-caption">Pages/Plain HTML project</p>

The template cretaed README.md file, initial `.gitlab-ci.yml` and public directory with `index.html` and `style.css` files.

Create and add SSH keys
=======================

We already have an example project, now we need to create SSH keys. They will be used to connect to our remote server. Each time GitLab CI/CD pipeline is running, it is using GitLab Runner.

GitLab Runner is an application which task is to run jobs in GitLab CI/CD pipeline. GitLab Runner can be installed by yourself on your infrastructure or you can leverage Shared Runners maintained by GitLab. You have 400 minutes per month for free from GitLab. We will use Shared Runner, since they are rady to use out-of-the-box and there is no configuration needed for our example. We need to setup SSH keys in a way that job run by Shared Runner will be able to access our remote server over SSH connection.

Create SSH key
--------------

You can create new SSH key in any environment, even your local environment. When you create new SSH key, you will receive two keys: private and public. It is important that GitLab have private key and your remote server has public key. That is why it doesn’t matter where you create keys, it only matters to share them accordingly with GitLab and remote server.

I have linux virtual machine on Azure and will use it for the purpose of this post. I’ll create new ssh key using the VM. GitLab recommendation is to create SSH key type ED25519, which is more secure than RSA. To create new key run `ssh-keygen -t ed25519 -C "GitLab SSH key"`. Text after `-C` option is a comment and you can change it.

![Create SSH key ed25519](../assets/2021-06-17-GitLab-CI-Pipeline-SSH/create_ssh_ed25519.png)
<p class="bottom-caption">Create SSH key ed25519</p>

The key will be created in default directory which for linux is `/home/<user>/.ssh`. Do not specify passphrase, otherwise it will be cumbersome for GitLab CI/CD pipeline. You should have two new files in `.ssh` directory:

* `id_ed25519` — private key
* `id_ed25519.pub` — public key

Add private key as GitLab Variable
----------------------------------

Copy content of private key and go back to GitLab project. Navigate to `Settings -> CI/CD -> Variables -> Expand -> Add Variable`. GitLab’s variable is a key-value pair. Name key `SSH_PRIVATE_KEY` and paste private key in value field. Click `Add Variable`.

Add two more variables:

* `SSH_USER` — name of the user on the remote server
* `VM_IPADDRESS` — IP address of remote server

![Added variables](../assets/2021-06-17-GitLab-CI-Pipeline-SSH/added_variables.png)
<p class="bottom-caption">Added variables</p>

Add public key to remote server
-------------------------------

Copy content of public key and go back to remote server. Login as the same user which you have specified in `SSH_USER` GitLab’s variable. If you don’t have yet this user, it is time to create it.

Navigate to `/home/<username>/.ssh`. If directory `.ssh` doesn’t exist, then create it. Paste the public key into `authorized_keys` file. If you don’t have `authorized_keys` file, create it. Here is screenshot from my VM (which I have deleted before posting, so they are useless now).

![Authorized keys setup](../assets/2021-06-17-GitLab-CI-Pipeline-SSH/authorized_keys.png)
<p class="bottom-caption">Authorized keys setup</p>

Create and run GitLab CI/CD pipeline
====================================

It’s time to create GitLab CI/CD pipeline. We want to achieve two goals using SSH: log remote server’s hostname and create an example file in user’s home directory.

The pipeline is defined in `.gitlab-ci.yml` and we have two option to create/edit:

1. Directly in GitLab project in web browser, we can edit `.gitlab-ci.yml` and commit changes
2. Clone repository, edit `.gitlab-ci.yml` in your favorite code editor, commit changes and push it to GitLab

I will go with option number 2, it’s more proper way to handle `.gitlab-ci.yml`.

You can clone repository using command `git clone <repo_address>` and repo address you can find in GitLab repository by clicking `Clone` button.  
After cloning open already existing `.gitlab-ci.yml` which was created as part of the Pages/Plain HTML template.

{% gist filip5114/3423562933cf40c983e7428e398c87f6 %}
<p class="bottom-caption">Original .gitlab-ci.yml</p>

We need to add `before_script` section and update `script` section.

{% gist filip5114/2bc3984db989e35d585b4149ff04475e %}
<p class="bottom-caption">Final .gitlab-ci.yml</p>

`.gitlab-ci.yml` defines pipeline. It uses docker image `alpine:latest` to run jobs defined in pipeline. We only have one job `pages`.

The job run in `stage: deploy`. We didn’t define any stages, but we have 5 default stages to use: `.pre, build, test, deploy, .post`. It doesn't matter in our case, since our pipeline at the moment is simple and doesn’t require setting up stages.

Then we have `before_script` which is pretty self-explanatory and will run before `script` command. Let’s explain script line by line:
* `command -v ssh-agent > /dev/null || (apk add --update openssh)` — checks if ssh-agent is already installed and if not, then install it
* `eval $(ssh-agent -v)` — starts ssh-agent
* `echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add —` — adds ssh private key stored in variable `SSH_PRIAVTE_KEY` to agent store
* `mkdir -p ~/.ssh` and `chmod 700 ~/.ssh` — creates `.ssh` directory and assign correct permissions
* `ssh-keyscan $VM_IPADDRESS >> ~/.ssh/known_hosts` — checks public key on remotes server using IP address stored in `VM_IPADDRESS` variable and add it to known hosts. It is protecting from men-in-the-middle attack and is necessary to work, otherwise the job will fail.
* `chmod 644 ~/.ssh/known_hosts` — assign correct permissions
* For more information refer to GitLab docs [here](https://docs.gitlab.com/ee/ci/ssh_keys/)

`script` is where our actual code to execute is defined. We simply want to print hostname to job log and then create an example file on remote host.  
`ssh $SSH_USER@$IP_ADDRESS "hostname && echo 'Welcome!!!' > welcome.txt"` will connect over SSH as user specified in `SSH_USER` variable to remote server, then run command `hostname` which will print hostname and echo `Welcome!!!` to file `welcome.txt` which will be created on remote server in `SSH_USER` home directory.

`artifacts` specify which artifacts to use in deployment. We are not using it in our example.

`only` specify that the job should be only run if any change is pushed into `master` branch in repository.

After making changes, we need to commit them and push to repository.

![Add, commit and push changes to repository](../assets/2021-06-17-GitLab-CI-Pipeline-SSH/add_commit_push_changes.png)
*Add, commit and push changes to repository*

Once the change is pushed into master branch the GitLab CI/CD will be trigged. Navigate to `CI/CD -> Pipelines` and you should see pipeline in status running.

![Running pipeline](../assets/2021-06-17-GitLab-CI-Pipeline-SSH/running_pipeline.png)
<p class="bottom-caption">Running pipeline</p>

Click on it and the click on job `pages` to see logs.

![Pipeline's job logs](../assets/2021-06-17-GitLab-CI-Pipeline-SSH/pipeline_job_logs.png)
<p class="bottom-caption">Pipeline's job logs</p>

Job should finish quickly, in my case it took 16 seconds. The last line shows that job was run successfully. Line 51 shows `script` part from `.gitlab-ci.yml` and in line 52 we can see remote server hostname which is exactly what we wanted to achieve. Check your remote server, you will find `welcome.txt` there.

That’s it! We have successfully created new GitLab project, setup SSH connection to remote server and created simple GitLab CI/CD pipeline to run script via SSH to the remote server.  
Thanks for reading.
