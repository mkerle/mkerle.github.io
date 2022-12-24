---
layout: single
title:  "Deploying a Jekyll Static Site to AWS Amplify"
date:   2022-12-24 16:30:00 +1000
toc: true
categories: aws docker ecr container amplify jekyll
---

## Introduction

This is a guide on how to deploy a Jekyll static site to AWS Amplify.

## Pre-Requisites

Have a git repository with a jekyll static site.  The static site does not need to be in the root of the directory structure as we can point Amplify to look in a subdirectory of the repo.

## AWS Build Image

To deploy a Jekyll site, AWS Amplify can use standard AWS images with Jekyll packages installed however the default version used may be older then the version required by your project.

A [custom docker image][my-aws-docker-guide] can be built and uploaded to get more recent jekyll versions that will be required.

An example custom docker image for Jekyll 3.1 is available at https://github.com/mkerle/docker-images/tree/main/aws/jekyll-amplify

## Deploy Site

In AWS go to `Amplify`.

1. Create a new app and select `Host web app`
1. Select `GitHub`.  You will then have to authorise access for AWS your GitHub account.
1. Select the repo with your Jekyll content you want to host.  Ensure that the correct branch is selected.  If you Jekyll content is not in the root directory then specify the path by enabled `Connecting a monorepo? Pick a folder`.
1. Modify the build settings.  AWS should figure out that it was a Jekyll site and if not maybe incorrect path was specified for the `monorepo` option.
1. In `Advanced settings` specify the URL to custom docker image if required.  The URL can be obtained from `AWS ECR`.
1. You can specify any package overrides in the `Live packages updates` section.

Once you click `Save and deploy` AWS will provision the server and clone your git repo and perform the tasks in the build settings.  If successful the site will be deployed otherwise you will need to download the logs to determine what the issue was.



[my-aws-docker-guide]: /aws-docker-ecr/