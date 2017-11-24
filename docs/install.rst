.. _install:

K10: Install
============

If you are deploying the Kasten platform in your own Kubernetes
cluster in AWS or Google (GKE), the below instructions will help.

.. contents:: Installation Overview
  :local:

Prerequisites
-------------

* You need the `Helm package manager <https://helm.sh/>`_
* Just like our :doc:`Kanister </kanister>` charts, you need to add
  the Kasten Helm repository to your local setup

.. code-block:: console

   $ helm repo add kasten https://charts.kasten.io/


Installing on AWS
-----------------

To install on AWS, you need to define two environment variables that
specify your access key id and secret access key.
After doing so, just run the following command to install K10,
the Kasten platform.

.. code-block:: console

   $ helm install kasten/k10 --name=k10 --namespace=kasten-io \
             --set k10_secrets.aws_access_key_id="${AWS_ACCESS_KEY_ID}" \
             --set k10_secrets.aws_secret_access_key="${AWS_SECRET_ACCESS_KEY}"

In particular, the above AWS access keys should have the following
policy permissions.


EC2 (EBS) Permissions
+++++++++++++++++++++++++

The following permissions are needed by Kasten to operate on EBS, AWS
EC2's underlying block storage solution

.. literalinclude:: ../pipelines/manifests/test_drive/aws_ec2_k10_minimal_policy.json
  :language: javascript


Optional S3 Permissions
+++++++++++++++++++++++

If you are going to migrate applications between different clusters,
you will need an S3 bucket for it. The credentials used above should
have the following permissions on the bucket.

.. note:: A future release will allow a completely different set of
  credentials to be used for accessing the S3 object store.

.. literalinclude:: ../pipelines/manifests/test_drive/aws_s3_bucket_acess_template.json
  :language: javascript

Installing on GKE
-----------------

A `GCP Service Account (SA)
<https://cloud.google.com/compute/docs/access/service-accounts>`_
automatically gets created with every GKE cluster.  This SA can be
accessed within the GKE cluster to perform actions on GCP resources
and is the simplest way to run the Kasten platform.  If the SA is
correctly configured (see more below), just run the following command
to install K10:

.. code-block:: console

   $ helm install kasten/k10 --name=k10 --namespace=kasten-io

Service Account Scope
+++++++++++++++++++++

By default, the default service account scope is sufficient for
K10. However, for cross-cluster application migration and K10 DR, we
also do need permissions to access Google Cloud Storage (GCS)
(``https://www.googleapis.com/auth/devstorage.read_write`` or,
alternatively, ``storage-full`` can also be used as an alias). The
complete scope listing is as follows:

.. code-block:: console

  compute-rw,
  storage-full,
  cloud-platform,
  https://www.googleapis.com/auth/trace.append,
  https://www.googleapis.com/auth/service.management.readonly,
  https://www.googleapis.com/auth/servicecontrol,
  https://www.googleapis.com/auth/monitoring.write,
  https://www.googleapis.com/auth/logging.write,

If the cluster was not originally created with the additional
credential, it is technically possible to modify the scope following
`these instructions
<https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances#changeserviceaccountandscopes>`_.
However, given the need to restart instances, that is not
recommended. As documented below, we recommend creating a separate
service account with the correct permissions.

Alternatively, if a new cluster is being created, the modified scope
can be set at that time. For example, it can be passed to the
``gcloud`` command via the ``--scope`` parameter.

Using a Separate Service Account
++++++++++++++++++++++++++++++++


Creating a new Service Account
##############################

K10 requires a newly created service account to contain the following
roles:

.. code-block:: console

  roles/compute.storageAdmin
  roles/storage.admin

The following steps should be used to create the service account and
add the required permissions:

.. code-block:: console

  myproject=$(gcloud config get-value core/project)
  gcloud iam service-accounts create k10-test-sa --display-name "K10 Service Account"
  k10saemail=$(gcloud iam service-accounts list --filter "k10-test-sa" --format="value(email)")
  gcloud iam service-accounts keys create --iam-account=${k10saemail} k10-sa-key.json
  gcloud projects add-iam-policy-binding ${myproject} --member serviceAccount:${k10saemail} --role roles/compute.storageAdmin
  gcloud projects add-iam-policy-binding ${myproject} --member serviceAccount:${k10saemail} --role roles/storage.admin


Installing K10 with the new Service Account
###########################################

For security reasons, ``helm`` does not allow the use of a file
outside the chart folder. To therefore use the ``k10-sa-key.json``
generated above, you need to fetch the chart, unpack it, and then
install with a pointer to the newly created credentials.

.. code-block:: console

  helm fetch kasten/k10 --devel --untar
  cp k10-sa-key.json k10/
  helm install k10/ --name=k10 --namespace=kasten-io --set k10_secrets.google_api_key_path=k10-sa-key.json


Kasten Dashboard Access
-----------------------

In order to access :doc:`Kasten Dashboard </overview>`, an `ingress
controller <https://goo.gl/C2MCYp>`_ needs to be deployed into the
Kubernetes cluster.  The Kasten Helm chart will deploy the
`nginx ingress <https://kubeapps.com/charts/stable/nginx-ingress>`_
and configure it with the NodePort service.  With the default
configuration, the :doc:`Kasten Dashboard </overview>` is secured and
can only be accessed via a user who is already authenticated using
``kubectl`` (see below).

If your cluster already has deployed an ingress controller, you can
disable the nginx ingress deployment by specifying the following
option to the ``helm`` command:

.. code-block:: console

    --set nginx.enable=false

To configure nginx ingress to be exposed through the default
LoadBalacer, please use the following option with ``helm``:

.. code-block:: console

    --set nginx.controller.service.type=LoadBalacer


Accessing via ``kubectl``
+++++++++++++++++++++++++

Run the following command to have kubectl establish a proxy connection
to the ingress controller service

.. code-block:: console

    kubectl proxy

The dashboard will be available at:
``http://localhost:8001/api/v1/namespaces/kasten-io/services/k10-nginx-controller:http/proxy/``

Accessing via a LoadBalacer
+++++++++++++++++++++++++++

To access the dashboard via the LoadBalancer, you have to first find
the LoadBalancer's public DNS/IP address:

.. code-block:: console

  kubectl --namespace kasten-io get services k10-nginx-controller \
     -o jsonpath='{.status.loadBalancer.ingress[].hostname}'

The K10 dashboard will be available at the root of the DNS or IP
address returned from the above command.

.. note:: Currently, directly viewing the dashboard via the
  LoadBalancer is only protected by Basic-Auth.

.. spelling::
   nginx
   kubectl
   Auth
