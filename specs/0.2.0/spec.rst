..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Build packages from src
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/packetary/+spec/build-packages

Implement package building module in Packetary to provide single application to
solve full range of tasks of packaging and repositories management.


--------------------
Problem description
--------------------

Repository management and packet building is
held in different interfaces that take a long time.
More convenient to to build packages using the same interface.

----------------
Proposed changes
----------------

We propose to implement building scripts to integrate it in
the Packetary and provide a Python application that wraps the
process to create a rpm package, relying on Docker and Mock to build rpm
packages in isolated environment.



------------
Alternatives
------------

* Koji:
  Supports rpm based distributions only
  https://fedoraproject.org/wiki/Koji

* Automated build farm (ABF):
  Supports rpm based distributions only
  http://www.rosalab.ru/products/rosa_abf
  https://abf.io/

* Delorean
  Supports rpm based distributions only
  Supports python packages only
  Requires separate docker image for each supported distribution
  https://github.com/openstack-packages/delorean

* docker-rpm-builder
  Supports rpm based distributions only
  Requires separate docker image for each supported distribution
  https://github.com/alanfranz/docker-rpm-builder

--------------
Implementation
--------------


*     Use standard upstream Linux distro tools to build packages (mock)

*     Every package should be built in a clean and up-to-date buildroot.

*     Package build tool is able to run build stage for different revisions of the same package in parallel on the same host.

*     Packages are built from git repositories with unpacked source (it's not necessary to commit source tarballs into git).


Packager shoud support followins source layouts:

- Source rpm file (.srpm .src.rpm)

- Standard source layout (git project):


  ./source tarball (.tar.*z)

  ./rpm specfile (.spec)

  ./other files related to package (.patch .init etc)

- Openstack source layout (two git projects):

  - source git project:

    ./source

    ./files

    ./and folders

  - spec git project

    ./centos7 (build target name)

    ./centos7/rpm/SOURCES

    ./centos7/rpm/SOURCES/files related to package (.patch .init etc)

    ./centos7/rpm/SPECS

    ./centos7/rpm/SPECS/rpm specfile (.spec)

    This layout should be converted to the standard one before build stage.

    Source tarball should be generated using `git-archive`.


Assignee(s)
===========

Primary assignee:
  Ivan Bogomazov <ibogomazov@mirantis.com>

Other contributors:
  None

Mandatory design review:
  None


Work Items
==========

* Create interface to run docker command from python

* Implement rpm packages build



Dependencies
============

None

----------
References
----------

None

