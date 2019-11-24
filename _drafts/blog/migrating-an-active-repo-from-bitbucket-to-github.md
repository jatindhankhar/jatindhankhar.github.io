---
title: Migrating an active repo from Bitbucket to Github
categories: blog
excerpt: Migrating an active repo with continuous pushes from Bitbucket to Github
tags:
- bitbucket
- github
comments: true
description: Migrating an active repo with continuous pushes from Bitbucket to Github

---
# What?

We needed to migrate a high commit frequency repository that was hosted on Bitbucket to Github.

The repository in question is a tier 1 service powering a critical part of the infrastructure so we needed to minimise the downtime and make sure deployments were not blocked during them migration

# Why?

We were using Bitbucket to host the code and it worked fine for basic use cases but we needed to support automated PR reviews along with assisting the said review and let's say, Bitbucket's UI wasn't pleasing to many.

Also we are using GoCD as CI pipeline along with [Pronto](https://github.com/prontolabs/pronto "Pronto") and it also didn't support Bitbucket's API properly. So we listed down the reasons why we needed to migrate to Github. Also rest of the organisation was already on Github, so it wasn't that hard to convince people.

# How?