+++
title = "Setting up python-openstackclient with Rackspace"
author = "antonym"
date = "2017-05-10"
description = "Setting up python-openstackclient with Rackspace"
tags = [
    "cloud",
    "howto",
    "openstack"
]
+++

I finally got around to switching to [python-openstackclient][1] after using [python-novaclient][2] and [supernova][3] for a number of years. Here's a quick way to get it set up if you're using [Rackspace][4] as a public cloud [OpenStack][5] provider and want to use multiple clouds or regions from one client.

Install python-openstackclient:

    pip install python-openstackclient
    

Create a directory to hold your cloud config file:

    mkdir -p $HOME/.config/openstack
    

Generate a clouds.yaml file, but make sure to put valid information for logging in (make sure to use your password instead of API key):

    cat <<EOF > $HOME/.config/openstack/clouds.yaml
    clouds:
      rackspace:
        cloud: rackspace
        auth:
          auth_url: 'https://identity.api.rackspacecloud.com/v2.0/'
          project_id: rackspace-account-number
          username: rackspace-account-username
          password: rackspace-account-password
        region_name: DFW,ORD,IAD,LON,SYD,HKG
    EOF
    

When running the openstack client command, you'll need to specify the cloud you want to use, and then specify the region you want to call. By default, if --os-region-name isn't specified, it will use the first entry set in region_name. To list servers in an account:

    openstack --os-cloud rackspace --os-region-name=IAD server list
    

To boot a quick server, first grab the flavor id and image id that you want to boot:

    openstack --os-cloud rackspace --os-region-name=IAD flavor list
    openstack --os-cloud rackspace --os-region-name=IAD image list
    

Then take the values you want and boot the server:

    openstack --os-cloud rackspace --os-region-name=IAD server create --image <image-uuid> --flavor <flavor-name> --key-name my_key mytestserver
    

If for any reason you get an [error][6] that states the following:

    No module named openstack.common
    

Check and see if you have some older Rackspace novaclient pip packages installed and remove them with pip uninstall:

    positron:~ ant$ pip freeze | grep ext
    os-diskconfig-python-novaclient-ext==0.1.2
    os-networksv2-python-novaclient-ext==0.25
    os-virtual-interfacesv2-python-novaclient-ext==0.19
    

For more information on configuration options, check out the [python-openstackclient configuration docs][7].

 [1]: https://github.com/openstack/python-openstackclient "python-openstackclient"
 [2]: https://github.com/openstack/python-novaclient "python-novaclient"
 [3]: https://github.com/major/supernova "supernova"
 [4]: https://www.rackspace.com "Rackspace"
 [5]: https://www.openstack.org "OpenStack"
 [6]: https://bugs.launchpad.net/python-novaclient/+bug/1579157
 [7]: https://docs.openstack.org/developer/python-openstackclient/configuration.html "python-openstackclient configuration docs"