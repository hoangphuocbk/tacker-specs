..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode


================================================
Support containerized VNF with Kubernetes as VIM
================================================

This proposal describes the plan to add Kubernetes as VIM in Tacker, so Tacker can support cloud native services
through Kubernetes plugin. OpenStack and Kubernetes will be used as VIMs for VM and Container based VNF respectively.
The plan narrowly focus on basic life cycle of c-VNF in Tacker (CRUD).

::

					   +----------------------------------------+
					   |Tacker(NFVO/VNFM)                       |
			  +------+ |         +-------------------+          | +-----------+
			  | Heat +-----------+   Infra drivers   +------------> Kubernetes|
			  |      | |         |                   |          | |   Client  |
			  +------+ |         +-------------------+          | +-----------+
				  |    |                                        |        |
				  |    +----------------------------------------+        |
				  |                                                      |
				  |    +----------------------------------------+        |
				  |    |VIM                                     |        |
				  |    | +--------------+ +-------------------+ |        |
				  |    | |              | |                   | |        |
				  |    | |   OpenStack  | |    Kubernetes     <----------+
				  +------>(VM-based VNF)| |(Containerized VNF)| |
					   | |              | |                   | |
					   | +--------------+ +-------------------+ |
					   | +------------------------------------+ |
					   | |        Neutron network & Kuryr     | |
					   | +------------------------------------+ |
					   +----------------------------------------+

					   +----------------------------------------+
					   |                                        |
					   |             Infrastructure             |
					   |                                        |
					   +----------------------------------------+

					   

Problem description
===================

Currently Tacker only support OpenStack as VIM, that means VNFs are created as virtual machines. In some Telco
scenario, virtualized network services need to quickly react with the change (updating, respawning from failure,
scaling, migrating), VM-based VNF may not be a good solution. Instead, other solutions such as container should
be used. Other hand, containerized VNFs are lightweight, small footprint and lower use of system resources, they
improve operational efficiency and reduce operational costs.

Kubernets is an open source system for automating deployment, scaling and management of containerized applications.
Its strength provide scheduling/deploying a group of related containers, self-healing features by using service
discovery and continuous monitoring. Although it is not yet suitable for all VNF cases, it is one of the more mature
container orchestration engine (COE). Currently, Kubernetes is chosen as COE in OpenRetriever [1] project (OPNFV). 

Proposed change
===============

This proposal is based on current status of available upstream projects (OpenStack, Kubernetes, Kuryr, etc) to support
containerized VNF. To adaptive with the current environment in Tacker, kuryr-kubernetes will be used for networking
between containers and VMs. Kubernetes can not creating SFC chainning between containerized VNFs, therefore creating
VNFFG between VM-based and container-based VNFs should rely on OpenStack. Otherwise currently Tacker only support VNFFG
in the same VIM. So this proposal will raise PoC, Kubernetes as sub VIM of the OpenStack VIM.





Alternatives
------------
There are some other options for implementing containerized VNF in Tacker.

1. Magnum

Magnum is a service to make COE such as Kubernetes, Docker Swarm, Apache Mesos. Considering Magnum will stitch containerized
VNF as nested containers (Container inside VM). In this proposal, we abstract registering Kubernetes as VIM, therefore the
Kubernetes clusters can be deployed on VMs (Magnum) or bare metals.

2. Zun

Zun could be used, but it is not mature at that time.

3. Docker

Directly use Dockerfile to create a VNF in Docker, but we can not limit the resources of each VNF by using Dockerfile.
Otherwise, Docker only focus on CRUD container on each machine, we need the orchestration tools for scheduling and managing
containers on multiple hosts.

4. Multus-CNI

For multiple networking in Kubernetes, Multus-CNI can be one solution. But currently Kuryr-Kubernetes doesn't support it. So
Multus-CNI will be considered in the future.

Data model impact
-----------------


REST API impact
---------------


Security impact
---------------


Notifications impact
--------------------


Other end user impact
---------------------


Performance Impact
------------------


Other deployer impact
---------------------


Developer impact
----------------


Implementation
==============

Assignee(s)
-----------
  Hoang Phuoc <hoangphuocbk2.07@gmail.com>

  Janki Chhatbar <jchhatba@redhat.com>

Work Items
----------


Dependencies
============


Testing
=======


Documentation Impact
====================


References
==========
.. [#f1] https://wiki.opnfv.org/display/OpenRetriever/OpenRetriever
