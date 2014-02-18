neutron
===================================

#### Table of Contents

1. [Overview - What is the neutron module?](#overview)
2. [Module Description - What does the module do?](#module-description)
3. [Setup - Tha basics of getting started with neutron.](#setup)
4. [Implementation - An under-the-hood peek at what the module is doing.](#implementation)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)
7. [Contributors - Those with commits](#contributors)
8. [Release Notes - Notes on the most recent updates to the module](#release-notes)

Overview
--------

The neutron module is a part of [Stackforge](https://github.com/stackforge), an effort by the Openstack infrastructure team to provide continuous integration testing and code review for Openstack and Openstack community projects not part of the core software. The module itself is used to flexibly configure and manage the newtork service for Openstack.

Module Description
------------------

The neutron module is an attempt to make Puppet capable of managing the entirety of neutron. This includes manifests to provision such things as keystone endpoints, RPC configurations specific to neutron, database connections, and network driver plugins. Types are shipped as part of the neutron module to assist in manipulation of the Openstack configuration files.

This module is tested in combination with other modules needed to build and leverage an entire Openstack installation. These modules can be found, all pulled together in the [openstack module](https://github.com/stackforge/puppet-openstack).

Setup
-----

**What the neutron module affects:**

* [Neutron](https://wiki.openstack.org/wiki/Neutron), the network service for Openstack.

### Installing neutron

    puppet module install puppetlabs/neutron

### Beginning with neutron

To utilize the neutron module's functionality you will need to declare multiple resources. The following is a modified excerpt from the [openstack module](httpd://github.com/stackforge/puppet-openstack). It provides an example of setting up an Open vSwitch neutron installation. This is not an exhaustive list of all the components needed. We recommend that you consult and understand the [openstack module](https://github.com/stackforge/puppet-openstack) and the [core openstack](http://docs.openstack.org) documentation to assist you in understanding the available deployment options.

```puppet
# enable the neutron service
class { '::neutron':
    enabled         => true,
    bind_host       => '127.0.0.1',
    rabbit_host     => '127.0.0.1',
    rabbit_user     => 'neutron',
    rabbit_password => 'rabbit_secret',
    verbose         => false,
    debug           => false,
}

# configure authentication
class { 'neutron::server':
    auth_host       => '127.0.0.1', # the keystone host address
    auth_password   => 'keystone_neutron_secret',
    sql_connection  => 'mysql://neutron:neutron_sql_secret@127.0.0.1/neutron?charset=utf8',
}

# enable the Open VSwitch plugin server
class { 'neutron::plugins::ovs':
    tenant_network_type => 'gre',
    network_vlan_ranges => 'physnet:1000:2000',
}
```

Other neutron network drivers include:

* dhcp,
* metadata,
* and l3.

Nova will also need to be configured to connect to the neutron service. Setting up the `nova::network::neutron` class sets
the `network_api_class` parameter in nova to use neutron instead of nova-network.

```puppet
class { 'nova::network::neutron':
  neutron_admin_password  => 'neutron_admin_secret',
}
```


The `examples` directory also provides a quick tutorial on how to use this module.

Implementation
--------------

### neutron

neutron is a combination of Puppet manifest and ruby code to deliver configuration and extra functionality through *types* and *providers*.


Limitations
-----------

This module supports the following neutron plugins:

* Open vSwitch
* linuxbridge
* cisco-neutron

The following platforms are supported:

* Ubuntu 12.04 (Precise)
* Debian (Wheezy)
* RHEL 6
* Fedora 18

* The Neutron Openstack service depends on a sqlalchemy database. If you are using puppetlabs-mysql to achieve this, there is a parameter called mysql_module that can be used to swap between the two supported versions: 0.9 and 2.2. This is needed because the puppetlabs-mysql module was rewritten and the custom type names have changed between versions.
Development
-----------

The puppet-openstack modules follow the Openstack development model. Developer documentation for the entire puppet-openstack project is at:

* https://wiki.openstack.org/wiki/Puppet-openstack#Developerdocumentation

Contributors
------------
The github [contributor graph](https://github.com/stackforge/puppet-neutron/graphs/contributors).

Release Notes
-------------

**2.2.0**

* Improved documentation.
* Added syslog support.
* Added quantum-plugin-cisco package resource.
* Various lint and bug fixes.
