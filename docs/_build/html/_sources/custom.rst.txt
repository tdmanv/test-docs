.. _custom:

Installing Custom Applications
==============================

While we have a sample application deployed for you to experiment
with, we encourage you to deploy any of your own applications, whether
they be custom container images or images downloaded from
DockerHub. The only current restriction is that they need to be
deployed into the ``default`` namespace.

Applications can be deployed via the use of either ``kubectl`` (and
the provided configuration file that was emailed to you), via the
Kubernetes dashboard available at
``https://your-testdrive-url.example.com/dashboard/``, or via Helm.

Once you have deployed your own applications, please feel free to
create policies that cover that application and bring the system back
into compliance, run manual restores, test restores, etc.


Using Helm
----------

The test cluster has Helm's Tiller installed in the "default" namespace.
Helm can be used to install and manage applications once ``kubectl``
is appropriately configured. Since Tiller is installed in the default
namespace, one of the following two approaches is needed:

* Exporting an environment variable: ``export TILLER_NAMESPACE=default``.
* Executing ``helm`` with the ``--tiller-namespace default`` flag.

Helm example
############

The below example shows how one can install MongoDB through the use of
a Helm chart.

.. code-block:: bash

  export TILLER_NAMESPACE=default
  helm install stable/mongodb-replicaset

Note that this is a a dynamically scalable MongoDB replica set using
Kubernetes StatefulSets and Init Containers. We will also be modifying
this Helm chart later to provide application-consistent snapshots
using Kanister, a Kasten framework.

.. spelling::
   Init
   Kanister
