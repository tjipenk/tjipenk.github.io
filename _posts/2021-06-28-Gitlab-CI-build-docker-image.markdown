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
\
Alpine doesn't have java installed by default - command `java -version` would fail. We will create Dockerfile to create new docker image based on Alpine with `openjdk11` installation. After building and pushing image to repo, we will use it in another job and run command `java -version` which should run successfully.
\
\
Creating new image is not the only option. You can use base Alpine image in your pipeline job, then in `script` section of the job install java with regular linux command. It will also work. Just keep in mind that that our example is simple. Real world scenario could include multiple software installation and configuration. You wouldn't want to run it every time pipeline is trigged. Time is precious in CI/CD pipeline. It's more efficient to build image at first and rebuild it only if Docker file changes.
\
\
{% gist filip5114/0dfddd04868c870927e074d37677b517 %}
<p class="bottom-caption">Dockerfile for alpine with openjdk installation</p>

Our Dockerfile couldn't be more simple:
- `FROM alpine:3` - use Alpine image with tag 3 as base image
- `RUN apk add openjdk11` - run command to install openjdk11
- `CMD ["/bin/sh"]` - specifies what command to run in container

You can try build the image locally and test if java is available:
- `docker build -t alpine-java .` - build docker image and tag it alpine-java
- `docker run --rm alpine-java java -version` - run container with alpine-java image and execute `java -version` command

![Local Dockerfile test](../assets/2021-06-28-Gitlab-CI-build-docker-image/local_dockerfile_test.png)
<p class="bottom-caption">Local Dockerfile test</p>

Dockerfile is ready. Push it to Gitlab repo or create Dockerfile directly in Gitlab.


Create Gitlab CI pipeline (.gitlab-ci.yml)
=======================

We will now create Gitlab CI pipeline and there are two options we could use:
1. Create a `.gitlab-ci.yml` file in the root of the repository
2. Use Gitlab CI/CD editor (in Gitlab, `CI/CD -> Editor`)

Option 1 is probably used more often, especially in project using a git branch strategy.
\
Option 2 is more than enough for our scenario and I'll go with it.

![Create New CI/CD Pipeline](../assets/2021-06-28-Gitlab-CI-build-docker-image/new_pipeline.png)
<p class="bottom-caption">Create New CI/CD Pipeline</p>

Click on `Create new CI/CD pipeline`. 
\
You will directed to editor with an example pipeline. We can use it, but we only need `stage` section including `build` and `test` stages. Remove the rest of the pipline and we can start creating jobs.

Build and push docker image
----------------------------------

Use docker image
----------------------------------

Run pipeline
====================================

