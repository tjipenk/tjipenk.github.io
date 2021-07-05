---
title: "Deploy from Gitlab to AWS"
layout: post
date: 2021-07-05 09:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- devops
- gitlab
- aws
- ci/cd
category: blog
author: karolfilipczuk
description: Use Gitlab CI pipeline for AWS deployment.
---
![Gitlab to AWS diagram](../assets/gitlab-aws/gitlab-aws-deploy.svg)
<p class="bottom-caption">Deploy from Gitlab to AWS</p>

Nowadays everything is hosted in a cloud which make sense: anybody can create environment from scratch in a blink of an eye, cluoud provides flexibility and scalability, cloud providers make sure you have plenty of choice in terms of resoruces and they take more and more maintenance duties from you. While all this is true, it is common to leverage cloud and at the same time use what you already been using until now.
\
In this blog post, I'll show you how to use Gitlab to deploy application on AWS.

Prerequisite
============

* Gitlab account
* AWS account

Agenda
======

1. Create Gitlab blank project
2. Create AWS resources
3. Create Gitlab CI pipeline (.gitlab-ci.yml)
	1. Build and push docker image
	2. Use custom docker image
	3. Be efficient!
4. Summary

Create new GitLab project
=========================


Summary
====================================


Thanks for reading!