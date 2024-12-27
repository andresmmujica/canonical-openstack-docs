.. _Network traffic isolation with MAAS:

Network traffic isolation with MAAS
===================================

Canonical OpenStack, in MAAS mode, supports network traffic isolation in a
multi-network environment, where each of these cloud networks is coupled
with specific cloud activity. It does this through the integration of
Juju network spaces.

[note type=“note”] **Network space:** A `Juju <https://juju.is>`__
object that abstracts away the
`OSI <https://en.wikipedia.org/wiki/OSI_model>`__ Layer 3 network
concepts. A space is created on a backing cloud (e.g. MAAS) to represent
one or more subnets. The purpose of a space is to allow for user-defined
network traffic segmentation. [/note]

Traffic isolation is implemented at the discretion of the cloud
architect, where the degree of isolation is dependent upon the number of
subnets used. That is, no isolation results from using a sole subnet
with a single space. Conversely, maximum isolation can be arrived at
with unique subnet-space pairings. The subnet:space mappings are done
within MAAS.

To finish, spaces are mapped to the cloud network names supported by
Sunbeam. The space:network mappings are done at the Sunbeam level.
In the case of an environment consisting of a sole subnet, each cloud
network will be mapped to the same space.

[note type=“note”] **Note:** The
:doc:`Install Canonical OpenStack using Canonical MAAS
</how-to/install/install-canonical-openstack-using-canonical-maas>`
page shows how to use Canonical OpenStack with MAAS. [/note]

Cloud networks
--------------

The cloud network names supported by Sunbeam, their corresponding
traffic types, and examples of such traffic is given here:

+-----------------+----------------------------+------------------------+
| Cloud network   | Traffic type               | Example traffic        |
+=================+============================+========================+
| data            | hypervisor-to-hypervisor   | intra-Project routing  |
|                 | (East-West)                | by OVN/OVS             |
+-----------------+----------------------------+------------------------+
| internal        | control plane              | Nova to RabbitMQ       |
|                 |                            | queries                |
+-----------------+----------------------------+------------------------+
| management      | cloud node management      | Juju                   |
+-----------------+----------------------------+------------------------+
| public          | service API endpoints      | Identity service via   |
|                 |                            | Keystone               |
+-----------------+----------------------------+------------------------+
| storage         | instance-to-storage        | Ceph-based volumes     |
+-----------------+----------------------------+------------------------+
| storage-cluster | storage-to-storage         | Ceph data rebalancing  |
+---------------+------------------------------+------------------------+

There are other types of traffic that don’t necessarily map to the above
cloud networks. They are described below:

+----------------+---------------------------+-------------------------+
| Other          | Traffic type              | Example traffic         |
| networking     |                           |                         |
+================+===========================+=========================+
| “external      | instance-to-external      | instance remote access  |
| networking”    | (North-South)             | over SSH                |
+----------------+---------------------------+-------------------------+
| “private       | instance-to-openstack     | OpenStack internal      |
| networking”    |                           |                         |
+----------------+---------------------------+-------------------------+

Machine access
--------------

Machines in the cloud environment require access to certain cloud
networks.

Node roles
~~~~~~~~~~

Machines identified by their Sunbeam node roles, and their associated
services, must have access to specific cloud networks. These access
requirements are described here:

+-----------------------+-----------------------+-----------------------+
| Node role             | Hosted service        | Cloud network access  |
+=======================+=======================+=======================+
| juju-controller       | Juju controller       | management            |
+-----------------------+-----------------------+-----------------------+
| sunbeam               | ``clusterd`` database | management            |
+-----------------------+-----------------------+-----------------------+
| control               | cloud control plane   | management, internal, |
|                       | services              | public, storage       |
+-----------------------+-----------------------+-----------------------+
| compute               | Nova Compute          | management, internal, |
|                       | (hypervisors)         | data, storage         |
+-----------------------+-----------------------+-----------------------+
| storage               | Ceph                  | management, storage,  |
|                       |                       | storage-cluster       |
+-----------------------+-----------------------+-----------------------+

Client
~~~~~~

The client machine will need access to the ``management`` and ``public``
cloud networks.

It will also need access to the cloud’s external networking in order to
access cloud instances (e.g. over SSH).
