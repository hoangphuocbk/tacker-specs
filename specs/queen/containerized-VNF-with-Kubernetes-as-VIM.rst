..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode


=================================================
Kubernetes as VIM in Tacker
=================================================
Disscusion document: [#first]_


This proposal describes the plan to add Kubernetes as VIM in Tacker, so Tacker can support cloud
native applications through Python Kubernetes client. OpenStack and Kubernetes will be used as
VIMs for Virtual machine and Container based VNFs respectively. This feature further be used to
create Kubernetes type of containerized VNF (c-VNF) and also hybrid cloud deployments of VM and
Container based VNF, NS.

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

Currently Tacker only supports OpenStack as VIM, that means VNFs are created by virtual machines.
In some Telco scenario, virtualized network services need to quickly react with the change such as
updating, respawning from failure, scaling, migrating. VM-based VNF may not be a good solution,
instead, other solutions such as container should be used. Other hand, containerized VNFs are
lightweight, small footprint and lower use of system resources, they improve operational efficiency
and reduce operational costs.

Kubernets is an open source project for automating deployment, scaling and management of
containerized applications. K8s also provides scheduling/deploying a group of related containers,
self-healing features by using service discovery and continuous monitoring. Although it is not yet
suitable for all VNF cases, it is one of the more mature container orchestration engine (COE).
Currently, Kubernetes is chosen as COE in Container4NFV project (OPNFV) [#second]_.

Proposed change
===============

Kubernetes as VIM

This proposal is based on current status of available upstream projects (OpenStack, Kubernetes,
Kuryr, etc) to support containerized VNFs in Tacker. Kuryr-kubernetes will be used as networking
between containers and VMs. However, Tacker doesn't manage Kubernetes cluster or care about where
cluster is deployed (on Magnum or bare-metal), Tacker just need their information about Kubernetes
clusters and registers Kubernetes as its VIM. Deploying DPDK, SR-IOV, multiple networking or
storage technologies for container (Kubernetes) should be role of other projects, such as
Container4NFV in OPNFV.

*OpenStack VIM configuration change*

Currently, when create VIM, the default type of VIM is openstack. This spec will add vim_type to
vim-config.yaml file to specify which type of VIM.

.. code-block:: ini

  auth_url: 'http://127.0.0.1/identity'
  username: 'admin'
  password: 'password'
  project_name: 'demo'
  project_domain_name: 'Default'
  user_domain_name: 'Default'
  vim_type: 'openstack'

*Sample configuration file for creating Kubernetes VIM*

.. code-block:: ini

  auth_url: 'https://192.168.122.50:6443'
  bearer_token: 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9'
  ssl_ca_cert: None
  vim_type: 'kubernetes'

VIM configuration use basic authentication is not supported in kubeadm-way installation. Due to
this bug [#third]_, Container4NFV project are focusing on this installation, Tacker can support
both two types of configuration, or leave it for the near future.

.. code-block:: ini

  auth_url: 'https://192.168.122.50:6443'
  user: 'kubernetes account'
  password: 'password'
  vim_type: 'kubernetes'

*Add Kubernetes HTTP client for managing c-VNF life cycle* 

For managing kubernetes type of c-VNF, Tacker will implement Python Kubernetes client [#fourth]_.
Through Kubernetes client, user can create Pod, Deployment, Horizontal Pod Autoscaling or Service
in Kubernetes VIMenvironment.

Kubernetes HTTP client will be introduced in Tacker, it implement Kubernetes Python Client.

*Assumptions*

When Kubernetes as VIM is deployed, user can create c-VNF from directly Kubernetes template.

.. code-block:: console
  tacker vnf-create --kubernetes-template k8s-vnf-template --vim-id VIM-id sampleVNF

Currently, Tacker uses  OASIS Tosca VNF standards to define VNF. Kubernetes environment uses their
template to define their resources like Pod, Deployment, Horizontal Pod Autoscaling or Service.
Translating from Tosca template to Kubernetes template is needed when Kubernetes is chosen as VIM.
In OPNFV, there are project Parser [#fifth]_, they intend to provide tosca2kube, but it is not
completed for now.


Alternatives
------------
There are some other options of implementing containerized VNF in Tacker.

1. Magnum

Magnum is a service to make COE such as Kubernetes, Docker Swarm, Apache Mesos. Considering Magnum
will stitch containerized VNF as nested containers (Container inside VM). In this proposal, we
abstract registering Kubernetes as VIM, therefore the Kubernetes clusters can be deployed on VMs
(Magnum) or bare-metal.

2. Zun

Zun could be used, but it is not mature at that time.

3. Docker

Directly use Dockerfile to create a VNF in Docker, but we can not limit the resources of each VNF by
using Dockerfile. Otherwise, Docker only focuses on CRUD container on each machine, we need the
orchestration tools for scheduling and managing containers on multiple hosts.

4. Multus-CNI [#sixth]_

For multiple networking in Kubernetes, Multus-CNI can be one solution. Currently Kuryr-Kubernetes
doesn't support it. So Multus-CNI will be considered in the future. Kubernetes also has plan for
multiple networking [#seventh]_.

Data model impact
-----------------

*New K8s-VIM auth database*

Because VimAuth database is defined for OpenStack VIM. Tacker need to create new database to store
data about Kubernetes VIM.

::

 +---------------------------------------------------------------------------+
 |Attribute     |Type   |Access  |Default   |Validation/ |Description        |
 |Name          |       |        |Value     |Consersion  |                   |
 +---------------------------------------------------------------------------+
 |vim_id        |string |RO, All |generated |N/A         |                   |
 |              |(UUID) |        |          |            |                   |
 +---------------------------------------------------------------------------+
 |auth_url      |string |RW, All |''        |string      |                   |
 |              |       |        |          |            |                   |
 +---------------------------------------------------------------------------+
 |bearer_token  |string |RW, All |''        |string      |                   |
 |              |       |        |          |            |                   |
 +---------------------------------------------------------------------------+
 |ssl_ca_cert   |string |RW, All |''        |string      |                   |
 |              |       |        |          |            |                   |
 +---------------------------------------------------------------------------+
 |vim_type      |string |RW, All |''        |string      |                   |
 |              |       |        |          |            |                   |
 +---------------------------------------------------------------------------+

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
.. [#third] https://github.com/kubernetes/kubernetes/issues/35536
.. [#fourth] https://github.com/kubernetes-incubator/client-python
.. [#fifth] https://wiki.opnfv.org/display/parser/Parser
.. [#sixth] https://github.com/Intel-Corp/multus-cni
.. [#seventh] https://docs.google.com/document/d/1TW3P4c8auWwYy-w_5afIPDcGNLK3LZf0m14943eVfVg/edit?ts=58877ea7#

