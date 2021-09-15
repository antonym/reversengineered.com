+++
title = "Leveraging GitLab for CI CD Deployments of OpenStack"
author = "antonym"
date = "2019-08-13"
description = "Leveraging GitLab for CI/CD Deployments of OpenStack"
tags = [
    "openstack",
    "gitlab",
    "ci/cd"
]
+++

This blog post is going to cover a way to set up Continuous Integration and Continuous Deployments (CI/CD) of your OpenStack cluster and is easily adaptable for various types of deployments like openstack-ansible or kolla-ansible.

An important part of any OpenStack deployment is the configuration for the environment. Once you have your ideal configuration set up and have production loads running, it's important to ensure your environment is deployed and updated in a consistent and controlled way. By utilizing version control, you can track all changes made to your configuration and ensure that new changes can be validated and approved by your peers and also know what previous changes have deployed successfully. Then by having a common method of deployment once those changes have merged in, you can ensure the environment is deployed the same way every time.

# Enter Gitlab

The way we'll do this is by leveraging [Gitlab][1] and it's all in one functionality to do version control with git and CI/CD automation.

We'll store the configurations in a git repository and encrypt the secrets with Ansible Vault. Then we will add a .gitlab-ci.yml to that repository that will add the functionality to do actions when merge requests come in or merges occur.

We will then set up a gitlab runner to the deployment host within the environment you want to have automatically deploy to. Gitlab will pass the job over to that deployment host and run the deployment and configuration changes. Then it'll gather up all the logs and save them as artifacts so that the deployment is fully logged and can be debugged if needed.

# Configure your OpenStack Environment

We'll make the assumption you've already done a fresh install and have your environment the way you want it. If you new to OpenStack or just getting started, these are some of the popular tools for deploying OpenStack:

openstack-ansible - <https://docs.openstack.org/openstack-ansible/latest/>  
kolla-ansible - <https://docs.openstack.org/kolla-ansible/latest/>

# Set up GitLab

You can either use a self hosted Gitlab server or the publicly hosted gitlab.com. Either will work, you just need to make sure that whatever host you are using for deployment has access to the internet as it will need to be able to talk to Gitlab to retrieve jobs.

### Create your configuration repo

    mkdir ~/my-openstack-configs/osa-configs
    cp -R /etc/openstack_deploy/* ~/my-openstack-configs/osa-configs/

### Encrypt your credentials

    # create a vault password file
    openssl rand -base64 40 > ~/.vault_pass.txt
    # encrypt your user_secrets if OSA, or passwords.yml if kolla
    ansible-vault encrypt ~/my-openstack-configs/osa-configs/user_secrets.yml --vault-password-file ~/.vault_pass.txt
    # check your file and ensure it&#039;s encrypted after running

### Set up your git repo

    cd ~/my-openstack-configs
    git init
    git remote add origin git@gitlab.com:your_repo/my-openstack-configs.git
    git add .
    git commit -m "Initial commit"
    git push -u origin master

Your configuration repo is now setup! Now to automate deployments!

### Set up your .gitlab-ci.yml

First we'll create a .gitlab-ci.yml that contains the Deployment job:

    stages:
      - deploy
    variables:
      ANSIBLE_VAULT_PASSWORD_FILE: /root/.vault_pass.txt
    include:
      - local: .gitlab-ci-deploy.yml
    
    #########################
    ## Deployment Pipeline ##
    #########################
    
    deploy_dc1:
      stage: deploy
      image: ubuntu:18.04
      extends: .deploy_script
      variables:
        DATACENTER: dc1
        OSA_RELEASE: stable/stein
      environment:
        name: production
      when: manual
      only:
      - master
      artifacts:
        paths:
          - artifacts/
        expire_in: 1 week 
      tags:
        - acme_dc1
    
    #############################
    ## End Deployment Pipeline ##
    #############################

### Set up .gitlab-ci-deploy.yml

Then we'll create a .gitlab-ci-deploy.yml that contains a lot of the common deployment scripting. This gets included in the original .gitlab-ci.yml. It's broken out into another file to keep the original pipeline file a bit cleaner. This can also be put into scripts and called with a script but showing it here as an example.

    .deploy_script:
      before_script:
        # install ssh-agent
        - &#039;which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )&#039;
        # run ssh-agent
        - eval $(ssh-agent -s)
        - export SSH_PRIVATE_KEY=${DATACENTER}_SSH_PRIVATE_KEY
        - echo "${!SSH_PRIVATE_KEY}" | tr -d &#039;\r&#039; | ssh-add - > /dev/null
        - mkdir -p ~/.ssh
        - chmod 700 ~/.ssh
        - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
        - ln -s $CI_PROJECT_DIR/osa-configs/ /etc/openstack_deploy
        - echo ${ANSIBLE_VAULT_PASSWORD} > /root/.vault_pass.txt
      script:
        - apt update
        - apt install -y python-pip git
        - git clone --branch $OSA_RELEASE https://github.com/openstack/openstack-ansible /opt/openstack-ansible
        - cd /opt/openstack-ansible
        - scripts/bootstrap-ansible.sh
        - cd /opt/openstack-ansible/playbooks
        - ansible-vault decrypt /etc/openstack_deploy/user*secret*.yml
        - openstack-ansible setup-hosts.yml
        - openstack-ansible setup-infrastructure.yml
        - openstack-ansible setup-openstack.yml
      after_script:
        - cp -R /openstack/log ${CI_PROJECT_DIR}/artifacts
        - ansible-vault encrypt /etc/openstack_deploy/user*secret*.yml
        - rm /root/.vault_pass.txt

### CI/CD Repo Settings to Tune on Gitlab Repo

The following settings are need to ensure CI/CD has enough time to run and so it can inject the SSH private key into the docker image during deployment run so that it can talk to the other hosts.

  * Add DATACENTER\_SSH\_PRIVATE_KEYS variable to GitLab Settings CI/CD variables
  * Increase Job Timeout from 1h to 8h (or desired timeout for larger environments) in CI/CD Settings, General Pipeline settings of configuration repo
  * Set contents of .vault\_pass.txt to ANSIBLE\_VAULT_PASSWORD so that the passwords can be decrypted during the job.

### Setup gitlab runner on your deployment host

You can choose to have the runner execute on the host directly with SSH or have it run all scripting in docker. I'm using docker in this example as it keeps the environment clean and uncluttered. The examples also currently build up the docker from scratch each time, but this can be optimized as well by capturing the docker image with the dependencies and configurations needed and then using that newly created image.

#### Install docker to deployment host

```
apt install docker.io
systemctl enable docker
systemctl start docker
```

#### Install gitlab runner

More information about gitlab runner is here: <https://docs.gitlab.com/runner/>

```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner
systemctl enable gitlab-runner
systemctl start gitlab-runner
```

#### Configure Gitlab runner

```    
# configure gitlab runner, headless options also available to skip user input, easy to automate
gitlab-runner register
# Server: https://gitlab.com or location of self hosted gitlab server
# Token: Found from repo&#039;s CI/CD Runners section in settings
# Description: Set description, i.e. "dc1 production runner" 
# Tag: tag used by gitlabci file to send job to runner, i.e. acme_dc1.  This is how you ensure it&#039;s listening to the right deploy job
# Type of deployer, docker or ssh (Please enter the executor: custom, docker, parallels, ssh, docker+machine, kubernetes, docker-ssh, shell, virtualbox, docker-ssh+machine)
```

Once setup, the runner will listen to the Gitlab Server set and action on any jobs requesting the tag on the token you configured the gitlab-runner for. The runner can be shutdown when out of a maintenance window to ensure deploys don't happen outside of a window if so desired.

From there changes merged in will kick off the pipeline and pause before deployment. Pressing the play button on the deployment will send the job to the runner and kick off the deployment.

 [1]: https://gitlab.com "Gitlab"