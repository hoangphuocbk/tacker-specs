..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode


=================================================
Support containerized VNF using Kubernetes as VIM
=================================================
Disscusion document: [#first]_


This proposal describes the plan to add Kubernetes as VIM in Tacker, so Tacker can support cloud native services
through Kubernetes plugin. OpenStack and Kubernetes will be used as VIMs for VM and Container based VNF respectively.
The plan narrowly focus on basic life cycle of c-VNF in Tacker (CRUD).

.. code-block:: console

                        +----------------------------------------+
                        |Tacker(NFVO/VNFM)                       |
               +------+ |         +-------------------+          | +-----------+
               | Heat +-----------+   Infra drivers   +------------> Kubernetes|
               |client| |         |                   |          | |   Client  |
               +---+--+ |         +-------------------+          | +------+----+
                   |    |                                        |        |
                   |    +----------------------------------------+        |
                   |                                                      |
                   |    +----------------------------------------+        |
                   |    |VIM                                     |        |
                   |    | +--------------+ +-------------------+ |        |
                   |    | |              | |                   | |        |
                   |    | |   OpenStack  | |    Kubernetes     <----------+
                   +------>(VM+based VNF)| |      clusters     | |
                        | |              | |(Containerized VNF)| |
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
container orchestration engine (COE). Currently, Kubernetes is chosen as COE in Container4NFV project (OPNFV) [#second]_. 

Proposed change
===============

1. Kubernetes as VIM

This proposal is based on current status of available upstream projects (OpenStack, Kubernetes, Kuryr, etc) to support
containerized VNF in Tacker. Kuryr-kubernetes will be used as networking between containers and VMs. However, Tacker
doesn't manage Kubernetes cluster or care about where cluster is deployed (on Magnum or bare metal), Tacker just need
information about Kubernetes clusters and registers Kubernetes as its VIM. Deploying DPDK, SR-IOV, multiple networking
or storage technologies for container (Kubernetes) should be role of other projects, such as Container4NFV in OPNFV.

2. VNFM

There will be 2 options in VNF management:

* New VNFM, that can be called as containerized VNFM. It seperates with the the old VNFM, because monitor, policy actions
  and management drivers will be different in Kubernetes environment.

* Add Kubernetes client [#third]_ as infra driver in the existing VNFM in Tacker.

3. VNF data-model changes

When apply Kubernetes as VIM, there will be 2 types of VNF, container-based and VM-based VNF. For now, Kubernetes can
not create service function chainning (SFC) between containerized VNFs, therefore creating VNFFG should rely on OpenStack
VIM. This proposal make changes and reuse networking-sfc feature to create SFC between VM-based and container-based VNFs,
not only CRUD seperate c-VNF.

4. Containerized VNF definition

Currently, Tacker use  OASIS Tosca VNF standards to define VNF. Kubernetes environment use their template to define objects
like pod, deployment, service, etc. Translating from Tosca template to Kubernetes template is needed when Kubernetes is
choosed as VIM. In OPNFV, there are project Parser [#fourth]_, they intend to provide tosca2kube, but it is not completed for now. 


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

4. Multus-CNI [#fifth]_

For multiple networking in Kubernetes, Multus-CNI can be one solution. But currently Kuryr-Kubernetes doesn't support it. So
Multus-CNI will be considered in the future. Kubernetes also has plan for multiple networking [#sixth]_.

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
  
  Trinath Somanchi <trinath.somanchi@nxp.com>
  
  Xuan Jia <jiaxuan@chinamobile.com>

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
.. [#first] https://docs.google.com/document/d/1zhJxoMc-_nFop8q2aB2mSjXZ_bjMQq1Ju9_P9ppV_Vo/edit#
.. [#second] https://wiki.opnfv.org/display/OpenRetriever/Container4NFV
.. [#third] https://github.com/kubernetes-incubator/client-python
.. [#fourth] https://wiki.opnfv.org/display/parser/Parser
.. [#fifth] https://github.com/Intel-Corp/multus-cni
.. [#sixth] https://docs.google.com/document/d/1TW3P4c8auWwYy-w_5afIPDcGNLK3LZf0m14943eVfVg/edit?ts=58877ea7#
