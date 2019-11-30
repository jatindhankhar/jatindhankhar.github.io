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

The repository in question is a tier 1 service powering a critical part of the infrastructure so we needed to minimize the downtime and make sure deployments were not blocked during the migration

# Why?

We were using Bitbucket to host the code and it worked fine for basic use cases but we needed to support automated PR reviews to assist the code reviews.  
There were many features that were available on Github but not Bitbucket

Some of them were

1. [Draft pull requests](https://github.blog/2019-02-14-introducing-draft-pull-requests/)
2. [Explicit reviewer request changes](https://help.github.com/en/articles/about-pull-request-reviews)
3. [Collapsing files and marking files as already viewed](https://help.github.com/en/articles/reviewing-proposed-changes-in-a-pull-request#marking-a-file-as-viewed)

Also, let's just say, Bitbucket's UI wasn't pleasing to many. (_cough_ No native syntax highlighting, and no way to hide comments _cough_)

We are using GoCD as CI pipeline along with [Pronto](https://github.com/prontolabs/pronto "Pronto") and it also didn't support Bitbucket's API properly. So we listed down the reasons why we needed to migrate to Github. Also rest of the organisation was already on Github, so it wasn't that hard to convince people. Overall it was much easier to integrate new stuff with Github's API than to deal with the Bitbucket's API.

# How?

#### Plan

The first step was to clone the repo from Bitbucket to Github and do periodic syncs.  
During this whole process, we turned off branch protection for your holy branch (in our case it is `master` and `release`)  
Once a clone of the code was made, we then tested our pipelines and deployments on one of the staging environments.  
After that it was just syncing the new repository periodically with new commits and once we were confident that it will work.  
Disabled writes to the old repository, asked developers to use the new repository and disable writes on the old one.

#### Execution

**First clone**
We followed this guide from Gist to migrate repo [Github Migration guide](https://github.com/aiidateam/aiida-core/wiki/How-to-migrate-from-BitBucket-to-GitHub "https://github.com/aiidateam/aiida-core/wiki/How-to-migrate-from-BitBucket-to-GitHub")

1. Create the new repo on Github, preferably empty without any files, repo with starter files/templates will also work
2. Take a mirror clone of the repo from Bitbucket
3. Push the repo to the newly created Github repo

```bash
git clone --mirror https://bitbucket.org/username/old_repo.git
cd old_repo
git remote set-url --push origin git@github.com:username/new_repo
git push --mirror
```

**Periodic syncs**

```bash
cd old_repo
# Set remote if not already done git remote set-url --push origin git@github.com:username/new_repo

# This pulls update from old repo 
git pull 
# This pushes the sync from old repo to new one 
git push --mirror
```