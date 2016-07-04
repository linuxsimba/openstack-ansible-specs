Virtual AIO using Vagrant
#########################
:date: 2015-06-30 22:00
:tags: aio, vagrant

Create a Virtual All-in-One(AIO) using Vagrant. This blueprint covers how this will be achieved using the Vagrant Virtualbox or Vagrant Libvirt-KVM Providers.

  * https://blueprints.launchpad.net/openstack-ansible/+spec/deploy-with-vagrant

Problem description
===================

To hasten develop and testing of Openstack-Ansible, this blueprint proposes using Vagrant to setup a virtual AIO environment.


Proposed change
===============
Provide Vagrant provisioning scripts that run through the install steps from the `AIO Quickstart Guide`_

A Virtual image used by Vagrant, called a vagrant box is easily built using the following steps.

* Download `Packer`_. It has MacOS, Linux and Windows support

* Git clone the `chef/bento`_ github repository.

* Build an Ubuntu Vagrant Box with 120GB. Building a custom vagrant box ensures that the vagrant box has the latest operating system updates and proper disk file size is set. Virtualbox and Libvirt-KVM both make use of `sparse file-systems`_ like VDI and Qcow2. A Vagrant Box therefore is less than 2GB when built. The disk can then expand to the set file size. Building a new vagrant box using `chef/bento` is quick. It takes only 10-15 minutes with a 10MB/s internet connection. Below is an example of how to build the Ubuntu 14.04 Server Vagrant box using the chef/bento Packer templates.

* `vagrant-triggers`_ is a Vagrant plugin that provides an easy way to checks the existence of the git release branch or git tag mentioned in the environment variables, before the VM is configured.

Building a Vagrant Box for Virtualbox
-------------------------------------
::

    git clone https://github.com/chef/bento
    cd bento
    packer build -var "disk_size=122880" -var "headless=true" -var "vm_name=aio" -var "box_basename=aio" -only virtualbox-iso ubuntu-14.04-amd64.json 
    vagrant box add builds/ubuntu-14-04.virtualbox.box --name "aio"
    vagrant box list

       aio                (virtualbox, 0)


Building a Vagrant Box for Libvirt-KVM
--------------------------------------
::
  
  git clone https://github.com/chef/bento
  cd bento
  packer build -var "disk_size=122880" -var "headless=true" -var "vm_name=aio" -var "box_basename=aio" -only qemu ubuntu-14.04-amd64.json 
  vagrant box add builds/aio.libvirt.box --name "aio"
  vagrant box list

       aio                (libvirt, 0)




Shell Environment Variables
---------------------------

After a Vagrant box is created, and added the vagrant box list, the following shell
environmental variables are available to customize the Openstack-Ansible AIO install.

(**\*** -  *Mandatory Options*)

-  **\*** ``AIO_OPENSTACK_RELEASE``: The Openstack-Ansible Git release branch to use in the AIO build. By default this is an empty string. If it is not set the AIO_TAG is checked. 
-  **\*** ``AIO_TAG``: Openstack-Ansible Git repo tag. *Use either AIO_OPENSTACK_RELEASE or AIO_TAG*. The Vagrantfile checks if a user has set both and stops to ask the user to unset one of the environment variables.
-  ``AIO_HYPERVISOR``: Can be either "virtualbox" or "libvirt". Default is "virtualbox".
-  ``AIO_VAGRANT_BOX``: Name of the Vagrant box to use. Default is "aio"
-  ``AIO_MEMORY``: Amount of memory in MB. Default is 8192
-  ``AIO_CPUS``: VCPUs consumed by the VM. Default is 4.
-  ``AIO_STOP_AFTER_BOOTSTRAP``: If set to 1, the Vagrant provisioner will stop after bootstrapping AIO. It will not run any playbooks.
- ``AIO_BOOTSTRAP_OPTS``: Applies additional BOOTSTRAP\_OPTS as described in the `AIO Quickstart Guide`_.


Example 1: AIO Build using a release name
------------------------------------------
::

   export AIO_OPENSTACK_RELEASE="stable/mitaka"
   vagrant up

Example 2: AIO Build using a tag
---------------------------------
::
   
   export AIO_TAG="13.0.1"
   vagrant up

Example 3: AIO Build, bootstrap AIO only
----------------------------------------
::

  export AIO_TAG="13.0.1"
  export AIO_STOP_AT_BOOTSTRAP=1
  vagrant up

Example 3: AIO Build, on the libvirt-kvm hypervisor
---------------------------------------------------
::

  export AIO_HYPERVISOR=libvirt
  export AIO_TAG="13.0.1"
  export AIO_STOP_AT_BOOTSTRAP=1
  vagrant up


Example Workflow
-----------------
This section describes the proposed process the user takes from start to finish on a Ubuntu-KVM hypervisor

::

  wget https://releases.hashicorp.com/vagrant/1.8.4/vagrant_1.8.4_x86_64.deb
  dpkg -i vagrant_1.8.4_x86_64.deb
  wget https://releases.hashicorp.com/packer/0.10.1/packer_0.10.1_linux_amd64.zip
  unzip packer_0.10.1_linux_amd64.zip
  cp packer /usr/local/bin/
  chmod 755 /usr/local/bin/packer
  git clone https://github.com/chef/bento
  cd bento
  packer build -var "disk_size=122880" -var "headless=true" -var "vm_name=aio" -var "box_basename=aio" -only qemu ubuntu-14.04-amd64.json 
  vagrant box add builds/aio.libvirt.box --name "aio"
  git clone https://github.com/openstack-ansible/
  cd openstack-ansible/scripts
  vagrant plugin install vagrant-libvirt
  vagrant plugin install vagrant-triggers
  export AIO_HYPERVISOR=libvirt
  export AIO_OPENSTACK_RELEASE=stable/mitaka
  vagrant up

Behind the Scenes
------------------

-  The virtual machine is started, based on the default values in the VagrantFile and shell environment variables set.
-  After the virtual machine is started, the Vagrant provisioner runs ``scripts/bootstrap-ansible.sh``
-  Then the provisioner runs ``scripts/bootstrap-aio.sh``
-  if ``AIO_STOP_AFTER_BOOTSTRAP`` is set, then the provisioner
   stops here. otherwise it runs ``scripts/run-playbooks.sh``

The vagrant-virtualbox provider, installed by default, is supported on Mac/Linux/Windows.

The vagrant-vibvirt Vagrant provider is supported only on Linux

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
AIO will be  slower using the Virtualbox
hypervisor beycause Virtualbox does not support `nested Virtualization <https://www.virtualbox.org/ticket/4032>`_.

Libvirt-KVM, does not have this limitation, therefore, performance is better.


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

Requires Vagrant 0.8+

Requires VirtualBox 4.x and higher or Libvirt 1.2.10 and higher


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  http://launchpad.net/~linuxsimba skamithi


Work items
----------

- Create a VagrantFile that provides virtualbox and libvirt support.


Testing
=======

Manual Testing. Not sure of a way to automate testing.


Documentation impact
====================

Virtual AIO Setup Documentation can be part of the AIO documentation.

References
==========

| `Vagrant`_
| `Vagrant-Libvirt`_

.. _vagrant-triggers: https://github.com/emyl/vagrant-triggers
.. _sparse file-systems: https://en.wikipedia.org/wiki/Sparse_file
.. _chef/bento: https://github.com/chef/bento
.. _AIO Quickstart Guide: http://docs.openstack.org/developer/openstack-ansible/developer-docs/quickstart-aio.html
.. _Packer: https://www.packer.io/downloads.html
.. _Vagrant: https://www.vagrantup.com/
.. _Vagrant-Libvirt: https://github.com/pradels/vagrant-libvirt
