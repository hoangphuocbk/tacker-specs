..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode


================================================
Support containerized VNF with Kubernetes as VIM
================================================

This proposal describes the plan to add Kubernetes as VIM in Tacker, so Tacker can support cloud native services
through Kubernetes plugin. As 

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
               |                 Infrastructure         |
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
containerized VNF and mixing scenarios between container and VM  in VNFFG. 





Alternatives
------------



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
