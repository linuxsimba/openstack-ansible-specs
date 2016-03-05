Virtual AIO using Vagrant
#################
:date: 2015-02-25 22:00
:tags: aio, vagrant

Create a Virtual All-in-One(AIO) using Vagrant. This blueprint covers how this
will be achieved using the Vagrant Virtualbox and Libvirt Providers.

  * https://blueprints.launchpad.net/openstack-ansible/+spec/deploy-with-vagrant

Problem description
===================

To hasten develop and testing of Openstack Ansible, this blueprint proposes
using Vagrant to  setup a virtual AIO environment.


Proposed change
===============
Have Vagrant provisioning scripts run through the steps described in
to    `AIO Quickstart Guide`_

.. _AIO Quickstart Guide: http://docs.openstack.org/developer/openstack-ansible/developer-docs/quickstart-aio.html

Virtual image used by Vagrant, called vagrant boxes, can be easily created using
HashiCorp's Packer utility.  A simple example is
as follows:

::

    git clone https://github.com/chef/bento
    cd bento
    packer build -only=virtualbox ubuntu-14.04-amd64.json
    vagrant box add builds/ubuntu-14-04.libvirt.box --name "trusty64"
    # confirm vagrant box addition
    vagrant box list

Prebuilt vagrant boxes can also be found on `atlas.hashicorp.com`_

.. _atlas.hashicorp.com: https://atlas.hashicorp.com/ubuntu/boxes/trusty64

After a Vagrant Box is added to the vagrant box list, set the the following optional
environmental variables to customize the environment:

-  VAGRANT\_AIO\_CPUS: Number of virtual CPUs used. Default is 2.
-  VAGRANT\_AIO\_MEMORY: Amount of memory in MB. Default is 4096
-  VAGRANT\_AIO\_DISK\_SIZE: Root disk size. Default is 80GB
-  VAGRANT\_AIO\_OPENSTACK\_RELEASE: Openstack-ansible release branch to
   use. By default this is none. When set to none, it will attempt to
   install the latest tagged release of openstack-ansible. If set it
   will install the latest tag in the specified branch.
-  VAGRANT\_AIO\_TAG: If set, this will override the
   VAGRANT\_AIO\_OPENSTACK\_RELEASE setting and checkout the repo at the
   tag specified.
-  VAGRANT\_AIO\_STOP\_AFTER\_BOOTSTRAP: If set to 1, then the Vagrant
   provisioner will stop after running
-  BOOTSTRAP\_OPTS: Applies additional BOOTSTRAP\_OPTS as described on
   the quickstart-aio guide.

If no customization is required, the default settings as described above are applied
and the following occurs after ``vagrant up`` is executed.

-  VM is started, based on the CPU and Memory and Disk settings defined
   in the environmental variables
-  After the VM is started, the Vagrant provisioner runs
   ``scripts/bootstrap-ansible.sh``
-  Then the provisioner runs ``scripts/bootstrap-aio.sh``
-  if VAGRANT\_AIO\_STOP\_AFTER\_BOOTSTRAP is set, then the provisioner
   stops here. otherwise it runs ``scripts/run-playbooks.sh``

The  Virtualbox Virtualbox provider is supported on Mac/Linux/Windows.

The Libvirt Vagrant provider is supported only on Linux

Alternatives
------------

None.

Playbook/Role impact
--------------------
None.

Upgrade impact
--------------

None.

Security impact
---------------

None.

Performance impact
------------------
The virtual AIO will be significantly slower using the Vagrant Virtualbox
Provider because Virtualbox does not support
`nested Virtualization <https://www.virtualbox.org/ticket/4032>`_.

Libvirt, using KVM, does not have this limitation, therefore,  the performance
is better.


End user impact
---------------

None. The virtual AIO is for testing and development


Deployer impact
---------------

None


Developer impact
----------------

None.

Dependencies
------------

Requires Vagrant 0.7+

Requires VirtualBox 4.x and higher or Libvirt 1.2.10 and higher


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  http://launchpad.net/~linuxsimba skamithi


Work items
----------

- Create the Virtualbox Provider VagrantFile

- Create the Libvirt Provider VagrantFile


Testing
=======

Manual Testing?  Not sure of a way to automate testing.


Documentation impact
====================

Virtual AIO Setup Documentation can be part of the AIO documentation.

References
==========

| `Vagrant`_
| `Vagrant-Libvirt`_

.. _Vagrant: https://www.vagrantup.com/
.. _Vagrant-Libvirt: https://github.com/pradels/vagrant-libvirt
