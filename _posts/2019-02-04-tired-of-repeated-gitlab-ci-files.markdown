---
title:  "Tired of repeated gitlab-ci files? Includes to the rescue!"
date:   2019-02-04 09:00:00
categories: [DevOps, CI/CD]
tags: [GitLab CI, GitLab]
---

Building pipelines aka Continuous Integration and Continuous Delivery (CI/CD) are not really new buzzwords in the tech industry or as sexy as bitcoin and friends, but I'm still quite excited about the recent release of the GitLab CE 11.7 Version which was released on [22nd January 2019](https://about.gitlab.com/2019/01/22/gitlab-11-7-released/){:target="_blank"}. In this post we will have a look to newly added feature in GitLab CI where we will 

> MAKE THE `GITLAB-CI.YML` DRY AGAIN


## TL;DR
As of GitLab CE version 11.7, you can use include commands to include your tasks from other .yml files.  This is a big deal, because you don‘t have to repeat your self in each of your projects any more.  Especially in projects that consist of a large number of microservices sharing (parts) of the pipeline definition is major win.

## GitLab
Even after the acquisition of Github by Microsoft, I still am a big fan and daily visitor of Github to look for new things around in the tech projects of my interest.

However, since approx. 1.5 years I‘m often confronted with GitLab installations at work and customers to serve as the internally hosted git service and I‘m kind of getting more and more to know about the strength of GitLab and the ecosystem around that, which is actually quite impressive. GitLab helps you as a developer to be more productive where it avoiding switches between for example Jenkins for your pipeline and code repository in Gogs.

## Introduction
Writing pipelines in Gitlab CI is quite an easy task when compared to writing Jenkins pipelines, at least for me. To be honest, I'm still trying to hide my Jenkins stuff that is related to private projects. On the other hand, GitLab CI is quite easy to understand and has great [documentation](https://docs.gitlab.com/ce/ci/yaml/){:target="_blank"} where you can start right away without learning any special programming language (e.g. Groovy).

# Usage of Include Tag
So the central part of this post is the newly available [`include tags`](https://docs.gitlab.com/ce/ci/yaml/#include){:target="_blank"} feature in the Gitlab CE 11.7 version.  Using them, you have the possibility to **include** other CI definitions, therefore allowing you to re-use your `jobs` from other files outside of your main repository. It is also possible to include remote .yaml files from other organizations. In the following, I will present some practical examples and some personal experience with this new feature.

In the following sections we will build the following pipeline:

![pipeline to build in this project]({{ "../images/gilab-ci.002.png" | absolute_url }})

>[Demo project](https://gitlab.com/imalik8088/gitlab-ci-include-test){:target="_blank"}  
>[Shared repo with pipeline job definitions](https://gitlab.com/imalik8088/gitlab-ci-pipeline){:target="_blank"}

### 1. Include snippets from the same repo to build your pipeline

The simplest way to extract your pipeline jobs from the main `.gitlab-ci.yml` is to move out the job definitions to a separate directory inside the same repo. In the following example, I moved the job definitions to the `ci` directory.

```yaml
.
├── .gitlab-ci.yml
├── ci
│   └── local-job.yml
```
The `local-job.yml` file echos some variable and also demonstrates the functionality to override job definitions that have already been defined in the main `gitlab-ci.yml` file, here the `script` and `variables` keys.

```yaml
include:
  - local: '/ci/local-job.yml'

variables:
  OVERRIDE_VAR: "OVERRIDE VALUE IN CI FILE"
  VAR1: "OVERRIDE VAR 1 FROM CI FILE"
  DEBUG: 'TRUE'

local-job:
  script:
    - echo "OVERRIDE FROM SCRIPT BLOCK OF MAIN GITLAB CI"
    - echo $OVERRIDE_VAR  

# --- OUTPUT OF THE JOB
# $ echo "before_script from 'include-local' task"
# before_script from 'include-local' task  
# $ echo "OVERRIDE FROM SCRIPT BLOCK OF MAIN GITLAB CI"
# OVERRIDE FROM SCRIPT BLOCK OF MAIN GITLAB CI  
# $ echo $OVERRIDE_VAR
# OVERRIDE VALUE IN CI FILE
```

### 2. Include a job definition from another repository within the same organization

![example setup in microservice organizations]({{ "../images/gilab-ci.001.jpeg" | absolute_url }})

Having some parts of your build in a separate file can help clean up big pipeline definitions, but especially with multiple repositories this is still too much repetition.  For this use-case, GitLab also allows you to include from a another repository in the same organization:


```yaml
include:
  - project: 'imalik8088/gitlab-ci-pipeline'
  file: '/maven-package.yml'

maven-package:
  artifacts:
    paths:
      - target/gitlab-ci-1.0-SNAPSHOT.jar  
```

This includes a file called `maven-package.yml` from the same GitLab organization but a **different** repository. Here again, you are able to change the default behavior by overriding the blocks or variables. This job definition will **expand** the original job definition, where we are defining a project-specific artifact where a .jar by the java application is been used to define it as an artifact in the context of GitLab. This comes in quite handy when you don't want any defaults but project-specific definition.


### 3. Include a job definition from a URL

The third and last option we will look at, is to include a job definition file via an arbitrary URL.  This is even more powerful than the previous options, because you are free to query any http endpoint that returns the file.
Here is an example of that:

```yaml
include:
  - remote: 'https://gitlab.com/imalik8088/gitlab-ci-pipeline/raw/master/build-docker.yml'

build-docker:
  variables:
    IMAGE_NAME: gitlab-ci-java
  dependencies:
    - maven-package
```

In this job definition, we are building a docker image. You have to define usually some extra code if you want to use docker in your CI (better known as Docker-in-Docker). This ceremony is now hidden inside an external job definition.  The main steps can still be overridden which leads to more concise pipelines where the boilerplate can be hidden behind the scenes. In this example, we are depending on the `maven-package` job where the artifact is transferred to this job definition and is used to build the docker image including the compile, test .jar from a previous stage.

### Notes
* One thing to note here is, that although you are defining your stages in the remote job definition you have to list all of the stage in the main .gitlab-ci file which defines the order in which jobs will be executed.
* I have included in the demo project a [gitlab ci file](https://gitlab.com/imalik8088/gitlab-ci-include-test/blob/master/old-without-includes.yml) from a personal project were its about **125 lines** of code and lot of repetitions. We can imagine how this gitlab ci file can benefit from the new feature described in this post and almost every organization have repetitive jobs like build, test, coverage, build docker images, deploy, etc.

## Conclusion
Gitlab and Gitlab CI have been build in the era of cloud-native applications and the whole integration of GitLab and CI/CD makes the whole DevOps lifecycle a breeze. With the newly added feature to include jobs definition from a local definition, a central repository or remote URL, we can finally get rid of all the repetition in our build definitions. 

_Thanks for the review [Markus](https://twitter.com/markus1189)!_