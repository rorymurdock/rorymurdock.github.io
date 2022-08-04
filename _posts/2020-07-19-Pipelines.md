---
layout: post
title:  "CI/CD Pipelines 1"
author: "Rory Murdock"
tags: Lab
---

Automating development with CI/CD

# Pipelines

![gif](https://media.giphy.com/media/DHM45OU8VEKDVivmQ8/giphy.gif)

This post was adapted from [my presentation](https://docs.google.com/presentation/d/1kD-DYPOJkp_EWeozYjWs5yTSsF7w6AbjIzmN-gT9u5Y/edit?usp=sharing) on Pipelines for MacAdmins

![Screenshot]({{ site.url }}/assets/img/pipelines/cicd.png)

[ref: medium.com](https://medium.com/tark-technologies/advantages-of-devops-for-business-166c436e3a7)

A CI/CD pipeline is an automated process that allows developers to automatically build, test, and deploy their code. In this post we will cover continuous testing wherein once code has been committed to our git repo [automatic code tests]({% post_url 2019-11-22-Automatic-Code-Testing %}) can be run across different versions and environments as well as staging a build to ensure everything.

There are many different pipeline offerings from [GitHub Actions](https://github.com/features/actions), [GitLab Pipelines](https://docs.gitlab.com/ee/ci/pipelines/), [Travis CI](https://travis-ci.org/), [Bamboo](https://www.atlassian.com/software/bamboo), [Azure Pipelines](https://azure.microsoft.com/en-au/services/devops/pipelines/), [GCP Cloud Build](https://cloud.google.com/docs/ci-cd), [AWS Code Pipeline](https://aws.amazon.com/codepipeline/) and many more. GitHub actions is an easy one to get started with due to it's tight integration however the referenced presentation predates GitHub actions so let's use Travis CI.

![Screenshot]({{ site.url }}/assets/img/pipelines/overview.png)

The way it works is when you commit some code to git it triggers a webhook to start the pipeline. The pipeline then pulls the code you just commit and runs the tasks set out in the pipeline. You can configure what tasks to run in a YAML file usually, it's also possible to define the task in your pipeline of choice but keeping it in YAML is best. Let's get started.

First create your git repo

![Screenshot]({{ site.url }}/assets/img/pipelines/create_repo.png)

Then clone it

![Screenshot]({{ site.url }}/assets/img/pipelines/new_repo.png)

![Screenshot]({{ site.url }}/assets/img/pipelines/git_clone.png)

First create `.travis.yml` and paste in the following, it's a very simple example, there's a lot more you can do with [advanced configuration](https://blog.travis-ci.com/2019-08-07-extensive-python-testing-on-travis-ci)

```yaml
language: python
python:
  - "3.5"
  - "3.6"

# command to install dependencies
install:
  - python -m pip install -r requirements.txt
  - python -m pip install pytest-cov coveralls

# command to run tests
script:
  - python setup.py -apikey $api_key
  - python -m pytest Tests/ --cov=./ -v
after_success:
  - coveralls
```

Let's go through it:

```yaml
language: python
```

We'll be using python

```yaml
python:
  - "3.5"
  - "3.6"
```

Testing with python version 3.5 and 3.6

```yaml
install:
  - python -m pip install -r requirements.txt
  - python -m pip install pytest-cov coveralls
```

Run the commands to install required packages and additional pytest requirements

```yaml
script:
  - python setup.py -apikey $api_key
  - python -m pytest Tests/ --cov=./ -v
```

Run the script to configure the API key ad the run `pytest` to start the tests

```yaml
after_success:
  - coveralls
```

After `pytest` has run submit the coverage report to [coveralls](https://coveralls.io/) -  [Coverage refresher]({% post_url 2019-11-23-Code-Coverage %})

Login to your travis account, add your github account, and then enable testing on your repo

![Screenshot]({{ site.url }}/assets/img/pipelines/travis_setup.png)

In the case we have an API key, now we could publish this in our YAML file but then the whole world could see it, people regularly monitor public git repos for API keys and passwords that are accidentally leaked. There's two ways you can add sensitive secrets to your pipeline.

* [Add an environment variable](https://docs.travis-ci.com/user/environment-variables/#defining-variables-in-repository-settings)
* [Encrypt the secret and then add to the YAML file](https://docs.travis-ci.com/user/encryption-keys/)

I prefer adding them to the environment variables but either way works.

![Screenshot]({{ site.url }}/assets/img/pipelines/travis_env.png)

Next you can add a coverage test with [coveralls](https://coveralls.io/), just sign up and add your repo

![Screenshot]({{ site.url }}/assets/img/pipelines/coveralls.png)

add the webhook

![Screenshot]({{ site.url }}/assets/img/pipelines/coveralls_webhook.png)

We've already covered automatic testing with PyTest [refresher]({% post_url 2019-11-22-Automatic-Code-Testing %})

There's two different modules in the test code, one for the REST API and one for the [Transport NSW API](https://opendata.transport.nsw.gov.au/developers/api-explorer) you can clone it yourself from [here](https://github.com/rorymurdock/Lightning-Talk-Pipelines)

`REST.py`, `Transport.py`, `Tests/REST_test.py`, and `Tests/Transport_test.py` are the main files we're interested in. They each have basic functions and the tests have full coverage.

Commit all files from the example repo your repo

![Screenshot]({{ site.url }}/assets/img/pipelines/commit.png)

Your commit will call the Travis CI webhook and the test will start automatically

![Screenshot]({{ site.url }}/assets/img/pipelines/test_run.png)

![Screenshot]({{ site.url }}/assets/img/pipelines/test_output.png)

You should protect your main branch so that code can not be pushed directly to it, a branch has to be made, and then a PR will run and build, if the build is successful and it has good coverage only then will it be able to be merge to main.

![Screenshot]({{ site.url }}/assets/img/pipelines/main_protect.png)

![Screenshot]({{ site.url }}/assets/img/pipelines/branch.png)

Make your changes and commit it to your branch

![Screenshot]({{ site.url }}/assets/img/pipelines/commit_branch.png)

Your tests should pass

![Screenshot]({{ site.url }}/assets/img/pipelines/build_success.png)

![Screenshot]({{ site.url }}/assets/img/pipelines/coverage_pass.png)

Go to your new branch and open a PR

![Screenshot]({{ site.url }}/assets/img/pipelines/create_pr.png)

![Screenshot]({{ site.url }}/assets/img/pipelines/open_pr.png)

Once you've opened the pull request you'll see all the pending jobs

![Screenshot]({{ site.url }}/assets/img/pipelines/pr_progress.png)

Once all of the jobs have completed, and passed you'll be able to complete the merge, close the pull request and delete the branch

![Screenshot]({{ site.url }}/assets/img/pipelines/merge_pr.png)

and that's it, with continuous testing whenever you commit code it will be automatically tested and coverage checked. You can tie into other services such as Codeclimate for further testing.

![gif](https://media.giphy.com/media/tn3kTJo4P4y1G/giphy.gif)
