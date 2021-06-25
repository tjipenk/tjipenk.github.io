---
title: "GitLab CI Pipeline. Build docker image."
layout: post
date: 2021-06-18 09:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- devops
- gitlab
- docker
- ci/cd
category: blog
author: karolfilipczuk
description: Include docker image build in Gitlab CI Pipeline, push it to Gitlab Repo and use it in another job.
---
![Photo by Jantine Doornbos on Unsplash](https://cdn-images-1.medium.com/max/1600/0*DzEDp3kusk5qSsnS)
<p class="bottom-caption">Photo by <a href="https://unsplash.com/@jantined">Jantine Doornbos</a> on <a href="https://unsplash.com">Unsplash</a></p>

Gitlab allows seamlessly using docker image from public and private hubs. I bet that most of you uses docker executors. All works great and without a hassle until you need to build your own docker image. Fortunately, you can build your docker image automatically in pipeline by leveraging docker-in-docker image build. 
\
I'll show you how to include docker image build in Gitlab CI Pipeline, push it to Gitlab Repo and use it in another job.

Prerequisite
============

* Gitlab account
* Docker basic knowledge

Agenda
======

1. Create Gitlab blank project
2. Create Dockerfile
3. Create Gitlab CI pipeline (.gitlab-ci.yml)
	1. Build and push docker image
	2. Use docker image
4. Run pipeline

Create new GitLab project
=========================

I'll start from scratch. Create new blank project which we will use throughout this blog post. 
Login into GitLab and navigate to `New project -> Create blank project/repository`. 
Give it a project name and hit `Create project`.

![Pages/Plain HTML project](../assets/2021-06-28-Gitlab-CI-build-docker-image/clone_project.png)
<p class="bottom-caption">Blank project</p>

Clone the project and we are ready to go.

Create Dockerfile
=======================

We need to create a Dockerfile, which will be used to build an docker image. Let's use `Alpine Linux` as base image. It is a minimal linux distribution ~5MB. Alpine Linux is great choice when you have specific task to accomplish and you want to use less storage, have fast build times. 
\
apk add openjdk11


Create Gitlab CI pipeline (.gitlab-ci.yml)
=======================

Build and push docker image
----------------------------------

Use docker image
----------------------------------

Run pipeline
====================================

