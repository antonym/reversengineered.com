+++
title = "Keeping Up to Date with Renovate"
author = "antonym"
date = "2022-03-13"
description = "Keeping Up to Date with Renovate"
tags = [
    "renovate",
    "gitlab",
    "ci/cd",
    "terraform",
    "terraform-provider-gitlab",
    "automation"
]
+++

Code dependencies become out of date over time and take valuable time to manage. Updating them becomes a chore as you:

- Identify a change occurred manually
- Parse the CHANGELOG to see what the latest changes are
- Create an MR with the changes
- Validate the pipelines completed successfully and merge the change in

Today most of those steps can be quite manual, leading to code not getting updated unless it's a critical security fix or a new feature is needed.

To help solve this problem, I have been using a tool called [Renovate](https://github.com/renovatebot/renovate) to automate the process of updating code.

## Renovate

Renovate is a bot which will scan repositories, identify dependency changes, and create merge request for each change for you automatically. You can then review the merge request, view the changelog of everything that has changed recently, validate the pipeline completed without any issues and then merge the changes in.

When you are dealing with many repositories, this reduces a large amount of toil and saves you time. It's very easy to set up as a self hosted runner so you have full control.

## Configuration

To get a self hosted runner working with Gitlab, you can start with this [repository](https://gitlab.com/renovate-bot/renovate-runner) and follow the instructions.

Some good things to know on getting Renovate set up:

- The self hosted runner repo will contain config.js that contains these [settings](https://docs.renovatebot.com/self-hosted-configuration/) to configure how Renovate runs on repositories
- Each repository will contain a [renovate.json](https://docs.renovatebot.com/configuration-options/) that controls how updates are ran on the repository
- For changes that may not be picked up, you can use the a [regex manager](https://docs.renovatebot.com/configuration-options/#regexmanagers) to set how the change is picked up

## Other Notable Features with Renovate

- Updates existing MRs if a new version is released
- Can set stability times and only update after a certain number of days has passed
- Docker image builds are easier to keep up to date by switching from `latest` to using a pinned versionThat way, images are rebuilt when new versions are identified and help prevent stagnant images and out of date images.
- If another change is merged in where the version is updated, Renovate will identify the new change and close the MR

## Managing Lots of Repos

If you have a lot of repositories to manage, you can use [Terraform](https://www.terraform.io) with the [Gitlab Provider](https://github.com/gitlabhq/terraform-provider-gitlab) and use the newly introduced support for [managing files](https://github.com/gitlabhq/terraform-provider-gitlab/pull/724) to generate renovate.json for many repositories and manage them from one repository. This makes it a lot easier to manage and update renovate configurations across a large amount of repositories.

## Save Time and Focus on What's Important

By implementing Renovate, you can save yourself a lot of time and focus on what's important while at the same time ensuring your codebase is kept up to date.