+++
title = "Upgrading Kolla Ansible for Deploying OpenStack"
author = "antonym"
date = "2019-05-10"
description = "Upgrading Kolla Ansible for Deploying OpenStack"
tags = [
    "openstack",
    "kolla",
    "ansible"
]
+++

If you are already running an environment with Kolla and want to upgrade it to the next release of OpenStack, it\'s pretty easy and quick to do. First you\'ll want to determine the version of the next version you\'ll want to deploy to and install the kolla-ansible for that version of OpenStack. You can view the releases of kolla-ansible here: <https://releases.openstack.org/teams/kolla.html>

For example:

Stein (8.x): 8.0.0.rc1  
Rocky (7.x): 7.0.2  
Queens (6.x): 6.2.1

So say you are running Rocky (7.x) and want to upgrade to Stein (8.x). You\'ll want to:

    pip install --upgrade kolla-ansible==8.0.0

You will now have the latest kolla-ansible code to do the upgrade to Stein. Upgrades need to be ran incrementally which means major version to major version. It is not a good idea to skip versions as you may miss things like needed database migrations for the environment.

Next we will need to update the configuration and inventory files to ensure we have all of the latest values from the new version of kolla-ansible. The files you need are the globals.yml, passwords.yml, and inventory files. Their locations are:

# CentOS

/usr/share/kolla-ansible/etc_examples/kolla/{globals.yml,passwords.yml} # configuration files  
/usr/share/kolla-ansible/ansible/inventory/{aio,multinode} # inventory

# Ubuntu

/usr/local/share/kolla-ansible/etc_examples/kolla/{globals.yml,passwords.yml} # configuration files  
/usr/local/share/kolla-ansible/ansible/inventory/{aio,multinode} # inventory

I would recommend starting with those files and porting any deviations or changes to your environment into those files so that you do not run into any unexpected issues. Each release can have some deviations in inventory, globals or passwords so it is important to redo the configurations for each release so that you don not run into unexpected issues when doing the upgrade.

Make sure to set the openstack_release in globals.yml to the version of kolla-ansible you want to deploy:

    openstack_release: 8.0.0.rc1

You will also need to make sure you have the latest list of passwords by merging the new list of passwords with the old list:

    mv /etc/kolla/passwords.yml passwords.yml.old
    cp kolla-ansible/etc/kolla/passwords.yml passwords.yml.new
    kolla-genpwd -p passwords.yml.new
    kolla-mergepwd --old passwords.yml.old --new passwords.yml.new --final /etc/kolla/passwords.yml

Once you have the configurations set up, you can begin the process of pulling down the images.

    kolla-ansible -i multinode pull

This will identify all images that need to be pulled down to hosts and prestage the images on the machines. By prestaging the docker images down the host, we can ensure we can rapidly upgrade the environment and reduce downtime. Once the images are pulled down we can begin the process of upgrading.

    kolla-ansible -i multinode upgrade

This will begin the process of restarting the docker containers one by one and handle the upgrading from one image to the next. At the end of the upgrade, all containters will be upgraded from rocky to stein and the environment will be available for use. The upgrade per node is usually pretty quick as it is just reloading a new image and doing things like migrating database during each container restart. Because all of the images have been pregenerated, deployment and upgrading can be very fast reducing downtime and maintenance times for large scale environments.

Official upgrade documentation can be found here:  
<https://docs.openstack.org/kolla-ansible/latest/user/operating-kolla.html>