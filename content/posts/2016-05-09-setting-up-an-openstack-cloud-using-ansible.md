+++
title = "Setting up an OpenStack Cloud using Ansible"
author = "antonym"
date = "2016-05-09"
description = "Setting up an OpenStack Cloud using Ansible"
tags = [
    "debian",
    "howto",
    "openstack",
    "linux",
    "networking",
    "nova"
]
+++

I use [Ansible][1] and [OpenStack][2] quite a bit on a daily basis, so I wanted to check out the work the community has done with the [openstack-ansible][3] project and get an OpenStack environment set up in my lab. I encourage you to read through the [documentation][4] as it is really detailed. Let's do this!

![img][5] 

### My Lab Environment

My setup consists of:

    4 x Dell PowerEdge R720s with 128GB of RAM
    Quad 10G Intel NICs
    Cisco Nexus 3k switches
    

I set aside one of the nodes for deployment and the other three were going to be used as targets. openstack-ansible currently supports Ubuntu 14.04 LTS (Trusty) so the first order of business was to install the OS to the servers. Future support for 16.04 LTS (xenial) and CentOS 7 may be coming down at some point as well.

### Setting up Networking

Once the OS was installed, the first thing to do was to set up the initial networking config in /etc/network/interfaces. For my setup, I'll be assigning networks to vlans for my setup.



Add some initial packages on the target host and enable some modules:

    apt-get install bridge-utils debootstrap ifenslave ifenslave-2.6 \
    lsof lvm2 ntp ntpdate openssh-server sudo tcpdump vlan
    echo 'bonding' >> /etc/modules
    echo '8021q' >> /etc/modules
    

Drop your interfaces file onto all of your hosts you'll be deploying and reboot them to apply the changes so that they set up all of the bridges for your containers and instances. In my example this configuration sets up dual bonds, VLANs, and bridges that OpenStack Ansible will plug everything into.

### Initial Bootstrap

You'll want to use one server as the deployment host so log into that server, check out openstack-ansible, and run the initial Ansible bootstrap:

    git clone https://github.com/openstack/openstack-ansible.git /opt/openstack-ansible
    cd /opt/openstack-ansible
    git checkout stable/mitaka
    scripts/bootstrap-ansible.sh
    

The bootstrap-ansible.sh script will generate keys so make sure to copy the contents of the public key file on the deployment host to the /root/.ssh/authorized_keys file on each target host.

Copy the example openstack_deploy directory to /etc/:

    cp -R /opt/openstack-ansible/etc/openstack_deploy /etc/openstack_deploy
    cp /etc/openstack_deploy/openstack_user_config.yml.example /etc/openstack_deploy/openstack_user_config.yml
    

Modify the openstack\_user\_config.yml for the settings you want. You'll need to specify which servers you want each role to do. The openstack\_user\_config.yml is pretty well commented and provides lots of docs to get started.

My config:



If you have enough memory and CPU on the hosts, you can also reuse the infrastructure nodes as compute_nodes to avoid having to set up dedicated nodes for compute.

### Credentials

    cd /opt/openstack-ansible/scripts
    python pw-token-gen.py --file /etc/openstack_deploy/user_secrets.yml
    

### User Variables

We'll start with just the basics for now to get operational. Make sure to enable at least a few options in the /etc/openstack\_deploy/user\_variables.yml otherwise it will have a hard time assembling the variables (these haven't made it into the mitaka stable branch yet):

    ## Debug and Verbose options.
    debug: false
    verbose: false
    

### Run the playbooks

    cd /opt/openstack-ansible/playbooks
    openstack-ansible setup-hosts.yml
    openstack-ansible haproxy-install.yml
    openstack-ansible setup-infrastructure.yml
    openstack-ansible setup-openstack.yml
    

If there are no errors, then the initial cluster should be setup. The playbooks are all idempotent so you can rerun them at anytime.

### Using the Cluster

Once these playbooks complete, you should have a functional OpenStack Cluster. To get started, you can log into Horizon with either the external vip IP you set up in openstack\_user\_config.yml or by hitting the server directly.

You'll use the user name "admin" and the password will be in your /etc/openstack\_deploy/user\_secrets.yml file that you generated earlier:

    grep keystone_auth_admin_password /etc/openstack_deploy/user_secrets.yml
    keystone_auth_admin_password: 4lkwtwtpmasldfqsdf
    

Each target node will have a utility container that you can ssh into to grab the openstack client credentials or run the client from the container. You can find it by doing an:

    root@osa-node1:~# lxc-ls | grep -i util
    node1_utility_container-860a6cd9
    root@osa-node1:~# ssh root@node1_utility_container-860a6cd9
    Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-85-generic x86_64)
    root@node1-utility-container-860a6cd9:~# openstack server list
    +--------------------------------------+-------------+--------+----------------------+
    | ID                                   | Name        | Status | Networks             |
    +--------------------------------------+-------------+--------+----------------------+
    | 1b7f1a7f-db87-47fe-a884-c66875ceed00 | my-instance | ACTIVE | Public=192.168.20.165|
    +--------------------------------------+-------------+--------+----------------------+
    

### Create and Setup Your Network

In Horizon under the System tab, select networks and then "+Create network". The main thing to note is depending on the network you are setting up, make sure to specify that type in the Physical Network box as well. In my case, I set up a vlan network, so I made sure to set:

    Name: Public
    Project: admin
    Provider Network Type: VLAN
    Physical Network: vlan
    Admin State: UP
    

Once the network is created, click on the Network Name and click "+Create Subnet". Add your:

    Subnet Name: VLAN_854
    Network Address: 10.127.95.0/24
    Gateway IP: 10.127.95.1
    Allocation Pools: <Start Address>,<End Address>
    DNS Name Servers: <DNS Servers>
    

### Add Images to Glance

You'll need to add some images to get up and running. You can find a list of supported images that include Cloud-Init [here][6].

    Name: Image Name
    Image Source: Image Location
    Image Location: Enter in URL of Cloud Image
    Format: QCOW2, RAW, or whatever the image format may be
    Architecture: x86_64 or whatever hardware you might be using
    Public: Checked
    

### Security Groups

By default security groups are enabled, so you'll want to enable some ingress rules like SSH and ICMP by default so you can connect to your instance.

### Start an Instance

Under the instances tab, click "Launch Instance". Fill in your desired options, including boot from image, add any keypairs you might want, and make sure to select the Security Group you set up previously. You'll also want to make sure you are plugged into the right network as well. Once all of those things are set up, you should be able to launch the instance and attempt to connect to it.

### Things to Note

#### Cluster Recovery

The target hosts are in a DB cluster so if you need to reboot them, make sure to stagger them, so that the cluster doesn't fail. If you find the DB is not coming up, you can run the galera-bootstrap playbook which should bring the cluster back up ([docs][7]):

    openstack-ansible galera-install.yml --tags galera-bootstrap
    

If you run into any issues running through this, please let me know in the comments or ping me in #openstack-ansible on Freenode as antonym.

 [1]: https://www.ansible.com/
 [2]: http://www.openstack.org/
 [3]: https://github.com/openstack/openstack-ansible
 [4]: http://docs.openstack.org/developer/openstack-ansible/install-guide/index.html
 [5]: https://c1.staticflickr.com/3/2403/2083277924_32a8f36177_b.jpg
 [6]: http://docs.openstack.org/image-guide/obtain-images.html
 [7]: http://docs.openstack.org/developer/openstack-ansible/mitaka/install-guide/ops-galera-recovery.html