---
layout: single
title:  "AWS Elastic Container Repository"
date:   2022-12-24 12:30:00 +1000
toc: true
categories: aws docker ecr container
permalink: /aws-docker-ecr/
---

## Introduction

Some AWS applications (e.g. Amplify) may require a custom docker image in order to run some application.  This is a short introduction on how to get a custom container into AWS [ECR (Elastic Container Respository)][ecr-general].

NOTE: that there are 2 versions of ECR, one for [public][pub-ecr] and the other for private repositories.  This guide will focus on public repositories as it is more accessible for free tier.

## Pre-Requisities

### Setup IAM

While this part my seem optional it is worth setting up an IAM user to perform ECR related tasks rather then using root user.

1. Go to AWS IAM as a user capable of creating policies and creating users.
1. Go to `Access Management-> Policies`
1. Click `Create Policy`
    1. Depending on requirements create a policy that allows access to ECR public (either specific registry or all)
    1. The policy should also allow read access of STS resources (Security Token Service)

With the policy setup next create a user group in IAM.

1. Go to `Access Management-> User Groups`
1. Click `Create group`
1. Give the group a name and attach the policy created earlier.

Now the user group is created create a user in IAM to perform these tasks.

1. Go to `Access Management-> Users`
1. Click `Add users`
    1. Give the user a name
    1. For `Select AWS access type` enable both `Access key` and `Password` options.
1. In the permissions screen choose `Add user to group` and then select the group created earlier.
1. Take not of the secret and password.  Password will need to be changed on first login.  The secret will be required in a later step.


### Create Registry

To create a public registry follow the below steps:

1. Login as the IAM user created earlier with permissions to perform ECR tasks.
1. Go to `Elastic Container Registry` in AWS
1. Click `Get Started`.  The create repository screen should appear.
    1. Select `Public` in visibility settings
    1. Give the repository a name
    1. Review other settings and  create the repository.

### Install AWS CLI Tools

Now on your local machine ensure that the [AWS CLI][aws-cli-install] tools are installed.  Use the instructions in the link to install.

With the AWS CLI tools installed we need to configure it with user permissions. Run:

    aws configure

Using the key and secret for the IAM user created earlier.

### Install Docker

On your local machine ensure that docker is installed.

## Create the Docker Image

These instructions are based on two sources:
1. The `View push commands` in ECR
1. The AWS instructions for [creating container images][create-container] on AWS ECS

The process is:
1. Replace `<region>` and `<registry>` with the actual values required. 
>`aws ecr-public get-login-password --region <region> | docker login --username AWS --password-stdin public.ecr.aws/<registry>`
1. Build docker image locally, see instructions further on or [aws docker instructions][create-container].  Start with `Dockerfile` preperation.  
1. Build the image using the below where `<image-name>` is the name to be used for the image.
>`docker build -t <image-name> .` 
1. Tag the docker image using 
>`docker tag <image-name>:latest public.ecr.aws/<registry>/<repository>:latest`
1. Push the image to ECR with 
>`docker push public.ecr.aws/<registry>/<repository>:latest`

### Dockerfile

A AWS base image can be used for the `Dockerfile` as a starting point.  When using `AWS Amplify` the `Dockerfile` can be viewed and then copied as a starting point.  Generally these AWS images and be cleaned up to remove unnessary packages that will not be required.

The below are an overview of the commands from [aws docker instructions][create-container].

1. Create the Dockerfile
> `touch Dockerfile`
1. Edit `Dockerfile` as required.
1. Build the image
> `docker build -t <image-name> .`
1. Check if the image was created correctly.  Should see the image in the output.
> `docker images --filter reference=<image-name>`
1. Run the docker image if required. i.e run commands to confirm `$PATH` and install directories etc.
> `docker run -t -i <image-name> '<cmd>'`

## Cleanup

Ensure that the docker image is removed from AWS ECR if it will not be used for a while to avoid incurring any costs.

    aws ecr delete-repository --repository-name <repository> --region <region> --force


[ecr-general]: https://docs.aws.amazon.com/ecr/index.html
[pub-ecr]: https://docs.aws.amazon.com/AmazonECR/latest/public/what-is-ecr.html
[aws-cli-install]: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
[create-container]: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html