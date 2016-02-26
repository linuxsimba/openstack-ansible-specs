Virtual AIO using Vagrant
#################################
:date: 2015-02-25 22:00
:tags: aio, vagrant

Create a Virtual All-in-One(AIO) using Vagrant. This blueprint covers how this
will be achieved using the Vagrant Virtualbox and Libvirt Providers.

  * https://blueprints.launchpad.net/openstack-ansible/+spec/deploy-with-vagrant

Problem description
===================

Currently to learn or experiment with the Openstack-Ansible deployer, one has to
execute AIO scripts on bare-metal. This blueprint proposes using Vagrant to
virtually deploy the AIO setup. This can help improve development and
testing of the Openstack-Ansible deployer.

Proposed change
===============
The core of the feature is two VagrantFiles, one for the VirtualBox Vagrant
provider and the second for the Libvirt Vagrant provider. It is possible to
use just one VagrantFile. For readability reasons, it is better to have
separate VagrantFiles for each provider.

Software and hardware requirements are as follows:

------------------------+-------------------------------------------------+
|                        | VirtualBox Provider    | Libvirt Provider      |
+========================+========================+=======================+
| RAM                    | 8GB                    | 8GB                   |
+------------------------+------------------------+-----------------------+
| Disk                   | 50GB                   | 50GB                  |
+------------------------+------------------------+-----------------------+
| Supported Hypervisor   | Windows, Mac, Linux    | Linux                 |
+------------------------+------------------------------------------------+
| Vagrant Box            | `HashiCorp Trusty64`_  | `Ubuntu Cloud Image`_ |
+------------------------+------------------------------------------------+

.. _HashiCorp Trusty64: https://atlas.hashicorp.com/ubuntu/boxes/trusty64
.. _Ubuntu Cloud Image: https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box


A set of Ansible Playbooks initiated by the Vagrant Ansible Provisioner
do the following:

-  Configure VM bridges and ethernet interfaces as described in the above
topology
-  Install the Deployer Host as a LXC container in the VM and sets up the LXC
   networking
-  Installs Openstack-Ansible  on the Deploy LXC. For the first version,
a static copy of /etc/openstack_deploy/user_config.yml is placed on the deploy
LXC. In the future, it may be less brittle to modify
the ``/etc/openstack/user_config.yml`` dynamically. Perhaps using the
``lineinfile`` Ansible module. This file appears to be dependent of the
Openstack-Ansible version. For example the Kilo version of
the ``user_config.yml`` is a little different than the liberty version.


The AIO topology design will allow the user to add additional Virtual nodes
to the setup to simulate an Openstack bare metal cluster. But this is not
the scope of this blueprint.

This blueprint currently does not cover Cinder/Swift/Ceph support. Contributions
will be kindly welcome in this area, before the code is completed.

The project file structure is as follows

- ``VagrantFile.libvirt``: VagrantFile for the Libvirt Provider
- ``VagrantFile.vitualbox``: VagrantFile for the VirtualBox provider
- ``aio_base_setup.yml``: Creates the Deploy Server LXC and pre-configures
the Vagrant VM as a Controller/Compute Node. The pre-configuration
includes creating the necessary bridges, like ``br-vxlan`` and ``br-vlan``.
- ``deployerserver.yml``: Clones the Openstack-Ansible repo to the Deploy LXC.
Creates the openstack-ansible executable. Adds an Apt-cache to the deploy
server LXC. Any LXC or VM after this that attempts to apt-get anything,
will go through the APT-cacher. It also creates a user_secret.yml file
with the simple password of 'password123'. This could be defined as a variable
in the VagrantFile.
- ``templates/openstack_user_config.yml.j2``: This is a modified copy of
the etc/openstack_deploy script located in the repo. It is a brittle solution
dependent on someone continuously modifying it each time a new OpenStack release
is done. Would be better to find a more intelligent and dynamic way of
creating a Vagrant AIO compatible user configuration file.

The user order of operations would be:

- ``git clone`` the openstack-ansible Repo
- cd to ``openstack/aio/vagrant``
- Go over the README that describes the requirements for Vagrant to work
- In the VagrantFile enter the Openstack Release the user wishes to deploy.
The variable options will be presented in the comment above the variable.
This blueprint is to build Vagrant AIO support for Liberty and Mitaka.
- Type  ``vagrant up``. This will run the VagrantFile and deploy the VM with the
deploy server LXC.
- Type ``vagrant ssh aio``. This will redirect the user into the deploy server
lxc container and place the user in the openstack-ansible git repo home
directory.
- The user now has the option to modify the default Vagrant AIO openstack_deploy
options as needed.
- Type ``openstack-ansible haproxy-install.yml`` to install HAProxy on the VM.
- Type ``openstack-ansible setup-everything.yml`` to deploy begin the AIO setup
to complete the setup.

Alternatives
------------

Have not seen a good alternative to prescriptively build a virtual
single or multi-node setups of Openstack. Vagrant provides a simple push button
approach, i.e "vagrant up", once the basic Vagrant requirements are met.



Playbook/Role impact
--------------------
It is ideal to intelligently modify ``openstack_user_config.yml``
after it is copied to the ``/etc/openstack_deploy`` location, not just
overwrite it for the virtual AIO setup. This is done fairly easily for
the ``/etc/openstack_deploy/user_secrets.yml`` file. For the
first version of this blueprint, overwriting the openstack_user_config.yml is
the current behavior. I would recommend the
default ``openstack_user_config.yml`` be modified and not overwriting. I
welcome contributions in this area.


Upgrade impact
--------------

The current way of deploying the vagrant AIO user configuration is brittle.
Using an upgraded
Contributions on how to deal with this, welcome.

Security impact
---------------

The Vagrant setup, applies the password "password123" for all passwords
to a modified copy of the ``/etc/openstack/user_secrets.yml``. This has no
security impact to the core repo.


Performance impact
------------------
The virtual AIO will be significantly slower using the Vagrant Virtualbox
Provider because Virtualbox does not support
_`nested Virtualization <https://www.virtualbox.org/ticket/4032>`_.

Libvirt using KVM does not have this limitation, therefore the performance is
better?


End user impact
---------------
None. The virtual AIO is for testing and development


Deployer impact
---------------

There are no additions to the core deployer code. Vagrant Ansible scripts
will modify the ``/etc/openstack_deploy/user_secrets.yml``,
``/etc/openstack_deploy/user_config.yml``
after the files are copied to the /etc/openstack_deploy location.


Developer impact
----------------

None. This project does not affect any of the core code.

Dependencies
------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <stanley@linuxsimba.com> (IRC: skamithi)


Work items
----------

* Create the Virtualbox Provider VagrantFile, without any Ansible provisioning
scripts
* Create the Libvirt Provider VagrantFile, without any Ansible provisioning
scripts
* Create the aio_base_setup.yml provisioning script.
* Create the deployserver provisioning script
Testing
=======



Documentation impact
====================

Virtual AIO Setup Documentation can be part of the AIO documentation.

References
==========

| `Vagrant`_
| `Vagrant-Libvirt`_

.. _Vagrant: https://www.vagrantup.com/
.. _Vagrant-Libvirt: https://github.com/pradels/vagrant-libvirt
