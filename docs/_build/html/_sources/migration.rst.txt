.. _migration:

Migrating Applications
======================

.. contents:: Migration Overview
  :local:

Introduction
------------

The ability to move an application across clusters is an extremely
powerful feature that enables a variety of use cases including
Disaster Recovery (DR), Test/Dev with realistic data sets, and
performance testing in isolated environments.

In particular, the K10 platform is being built to support
application migration in a variety of different and overlapping
contexts:

* **Cross-Namespace**: The entire application stack can be migrated
  across different namespaces in the same cluster (covered in
  :doc:`restoring applications </restore>`).

* **Cross-Cluster**: The application stack is migrated across
  non-federated Kubernetes clusters.

* **Cross-Account**: Migration can additionally be enabled across
  clusters running in different accounts (e.g., AWS accounts) or
  projects (e.g., Google Cloud projects).

* **Cross-Region**: Migration can further be enabled across different
  regions of the same cloud provider (e.g., US-East to US-West).

* **Cross-Cloud**: Finally, migration can be enabled across different
  cloud providers (e.g., AWS to Azure).


Migration Configuration
-----------------------

Some additional infrastructure configuration is needed before
migration between two clusters can be enabled.

* **Required**: Object storage configuration
* **Use-Case Specific**: Cross-account and Kanister configuration


Object Storage Configuration
++++++++++++++++++++++++++++

For two non-federated clusters to share data, we need to create an
object storage bucket (e.g., ``k10-migrate``) that is owned and
writable by the role or account the source K10 deployment is running
as. The bucket also needs to be readable by the role or account of the
destination deployment. This bucket is used to store metadata about
the application restore points that have been selected for migration
between clusters.

Further, if the source and destination clusters are in different
regions of the cloud provider, it is **essential** that this shared
object storage bucket is created in the same region as the destination
cluster as we use that to determine where artifacts (e.g., snapshot
copies) should be copied.

Cross-Account Configuration
+++++++++++++++++++++++++++

The following configuration is **only** required if cross-account
migration is needed.

AWS
###

If cross-account volume migration is desired, K10 needs an IAM user
created within the destination account (e.g., ``k10migrate``). The
`AWS account ID
<http://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html>`_
is also needed. During the creation of the migration profile, the
username should be specified as ``<AWS Account ID>/<User Name>``

In an AWS environment, K10 grants access to snapshots stored in the
source account to the destination account. If the destination cluster
is in a different region, K10 will make a cross-region copy of the
underlying EBS snapshots before sharing access.


Google Cloud
############

Currently, due to underlying provider limitations, migration is only
supported among different clusters in the same project and therefore
no additional configuration is required.

Kanister Configuration
++++++++++++++++++++++

The :doc:`Kanister framework </kanister>`, described later, stores
application-level artifacts in an object store. If Kanister artifacts
are migrated across clusters, the destination cluster currently needs
read access to the all object storage buckets that are being used to
store artifacts.

.. note:: The above requirement for the Kanister object storage bucket
  to be shared will go away in an upcoming release in favor of only
  using the object storage bucket configured above for migration.

Import/Export Profiles
----------------------

To enable the export and import of applications, you are required to
create a profile on each side of the transfer. Profile creation can be
accessed from the ``Config Profiles`` icon in the top-right corner of
the dashboard.

Export Profile
++++++++++++++

To move data to another cluster, simply click ``Create New Profile``
on the profiles page and select ``Export``. We recommend using the
target cluster name as the profile name to make it easier to access
later.

As mentioned earlier, exporting data requires an object storage
location. You are therefore required to pick an object storage
provider, a region for the bucket, and the bucket name. If a bucket
with the specific name does not exist, it will be created in the
specified region.

   .. image:: ./_static/config_profile_export.png

When you hit create, the config profile will be created and you will
see a profile similar to the following:

   .. image:: ./_static/config_profile_example.png


Import Profile
++++++++++++++

To import data into a cluster, we also need to create a import
profile. Currently, this is just used to guide policy creation and
requires no parameters but we do recommend using the source cluster
name as the profile name.

   .. image:: ./_static/config_profile_import.png


Exporting Applications
----------------------

The workflow for exporting applications into another cluster is very
similar to the workflow followed for :doc:`restoring applications
</restore>`. You simply go over to the Namespaces page, click on the
dropdown, and select ``Export Namespace``.

   .. image:: ./_static/namespaces_manual.png

After a restore point is selected, you will be presented with
information about the application snapshot being migrated (e.g., time,
originating policy, artifacts) and, more importantly, the option to
configure the destination by selecting a migration profile.

   .. image:: ./_static/migration_export.png

A migration profile, as defined above, specifies where the exported
data and metadata will be stored. This profile contains the name of
the cluster that we are migrating data to and the location of an
object storage bucket (e.g., using AWS S3, Google Cloud Storage, or
any other S3-compliant object store) to export the needed data.

Simply select any of the migration profiles you might have previously
created and then click on ``Export``. After confirming the export, you
will be presented with a confirmation dialog. It is **essential** that
you copy the text block presented here because it will be required to
securely create an import policy on the destination cluster.

   .. image:: ./_static/migration_textblock.png


Importing Applications
----------------------

Importing an application snapshot is again very similar to the policy
workflow for :doc:`protecting applications </protect>`. From the
``Policies`` page, simply select ``Create a New Policy``.

   .. image:: ./_static/migration_import_policy.png

To import applications, you need to select ``Import App Data`` for the
action. While you need to also specify a frequency for the import,
this simply controls how often to check the shared object storage
location. If data within the object store is not refreshed at the same
frequency, no duplicated work will be performed.

You will also need to paste the text block presented by the
destination cluster during export to allow us to create the data
export/import relationship for this application across these
clusters. Finally, similar to the export workflow, the permissions
profile also need to be selected but, as mentioned earlier, the
destination cluster only needs read and list permissions on all the
data in the shared object store.

After the ``Create Policy`` button is clicked, the system will start
scheduling jobs based on the policy definition. If new data is
detected, it will be imported into the cluster, automatically
associated with the application stack already running, and be made
available as a restore point. The normal workflow for :doc:`restoring
applications </restore>` can be followed and, when given the choice,
simply select a restore point that is tagged with the policy name used
above to create the import policy.


.. spelling::
   Kanister
   Apps
   namespaces
   metadata
   username
   Namespaces
   config
