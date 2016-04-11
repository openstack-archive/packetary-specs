..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================================
Build binary kernel module packages from DKMS packages
======================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/packetary/+spec/dkms-to-binary-modules-rebuild

The document describes a proposal for building binary kernel module
packages from corresponding DKMS packages (shipped in the DKMS format).
Using pre-built binary kernel module packages in the bootstrap image could
significantly reduce the size of the image and reveals potential build
issues.

--------------------
Problem description
--------------------

The key strength of `Dynamic Kernel Module Support (DKMS)`_
is the ability to rebuild the required kernel module for a different version
of kernels. But there is a drawback of installing DKMS kernel modules into
bootstrap. DKMS requires additional packages like ``linux-headers`` and a
tool-chain building to be installed, which unnecessarily oversizes
the bootstrap.

DKMS allows to build a particular driver for various versions of the Linux
kernel. However, kernel ABI (binary API) may change, what makes the
given driver source code incompatible with the kernel. The driver could
invoke non-existed functions or data structures etc.
Rebuilding binary module packages from DKMS would have helped to reveal
potential issues which may be encountered installing a particular DKMS
package.

In conclusion, implementing the functionality for rebuilding binary kernel
module packages from DKMS packages will allow reducing size of bootstrap images
(to compare with including DKMS modules case) and catch the potential faults
in building/installing DKMS modules on bootstrap.

Workflow of using the feature
=============================

The following workflow is expected for using this feature.

* Create dedicated repository for keeping the binary kernel modules
  (in form of .deb or .rpm packages). The repository could be placed
  on the Fuel master node next to the other repositories.

* Re-build required DKMS kernel module packages and save the produced
  binary kernel module packages in the created repository mentioned above.

* Update the content of the repository for the binary kernel modules (mentioned
  above) and rebuild the bootstrap with required binary kernel module packages.

Rebuilding kernel module binaries shall be done prior building the bootstrap.
Each time, when the new version of the kernel is updated on a bootstrap,
the binary kernel modules shall be rebuilt against the new kernel as well.

The details about creating repository and building new bootstrap could be
found in the `document`_.

The case of building a few bootstraps with different kernel version is
possible. The repository will contain the packages for different kernel
version, e.g.

  * i40e_1.3.47-3.13.0-77-generic_x86_64.deb
  * i40e_1.3.47-3.13.0-75-generic_x86_64.deb
  * i40e_1.3.47-3.13.0-83-generic_x86_64.deb

A user shall specify exact version of the required kernel module package
(e.g. i40e_1.3.47-3.13.0-77-generic) for building bootstrap with the
corresponding kernel version (3.13.0-77-generic).

----------------
Proposed changes
----------------

#. The `Packetary`_ shall be extended with the additional functionality
   enabling a user to build binary kernel module packages from corresponding
   DKMS module packages. 
#. The possible implementation of the functionality could be following:

    Packetary dkms2bin --dkms <dkms-package> --out-dir <output-dir> \
    --kernel <kernel version> --debug 

    Where:
    <dkms-package>   - path to the DKMS package list required to be rebuilt
    <out-dir>        - folder to save outcome kernel binary modules
    <kernel version> - kernel version against which the kernel module should
    be rebuilt;
   
#. The error messages shall be provided in case of fault when building
   a binary kernel module package from corresponding DKMS package failed.
   The --debug option shall provide access to the chroot environment for
   analyzing the cause of possible fault. (It means the chroot should not
   be removed after the build).
#. The output binary kernel module naming schema could be different for
   CentOS and Ubuntu packages, but the module shall contain at least the
   module version and kernel version (against which it was built) in its
   name. For example, moudule i40e version 1.3.47 built against 3.13.0-79
   kernel may be named as i40e_1.3.47-3.13.0-79.deb 

See the `Separate MOS from Linux repos`_ specification for details.


Example of user steps, required to build

.. code-block:: bash

  $ Packetary dkms2bin --dkms i40e-dkms-1.3.47~ --out-dir /var/www/nailgun/repo/dkms2bin-repo

  ... creating chroot
  ... installing DKMS packages, building
  ... exporting kernel binary module packages into the --out-dir

The documentation shall be extended with the new command description.

------------
Alternatives
------------

There is a document describing the steps to `rebuild DKMS manually`_, but
it would better to have a tool to simplify work.

--------------
Implementation
--------------



Assignee(s)
===========

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Mandatory design review:
  <launchpad-id or None>


Work Items
==========

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.


Dependencies
============

* Include specific references to specs and/or blueprints in Packetary,
  or in other projects, that this one either depends on or is related to.

* Does this feature require any new library dependencies or code otherwise not
  included in Packetary? Or does it depend on a specific version of library?


----------
References
----------

.. _`Dynamic Kernel Module Support (DKMS)`: https://help.ubuntu.com//community/DKMS
.. _`document`: https://docs.mirantis.com/openstack/fuel/fuel-8.0/fuel-install-guide.html#bootstrap-inject-driver
.. _`Packetary`: https://wiki.openstack.org/wiki/Packetary
.. _`rebuild DKMS manually`: http://docs.openstack.org/developer/fuel-docs/devdocs/develop/custom-bootstrap-node.html#adding-dkms-kernel-modules-into-bootstrap-ubuntu
.. _`Separate MOS from Linux repos`: https://review.openstack.org/gitweb?p=openstack/fuel-specs.git;a=blob;f=specs/6.1/separate-mos-from-linux.rst;h=b9031b4971ae68377d9ffeea75fe0095bad61b3b;hb=refs/heads/master
