.. _kanister:

Kanister: Application-Level Data Management
===========================================


While it is possible to both protect, restore, and migrate
applications at the storage volume level, this is not always desirable
for distributed data services that might be eventually
consistent. There are also a number of advantages that can be obtained
from operating on data at the application level. This includes faster
incremental copies, strongly consistent point-in-time snapshots, the
ability to rescale replicas across clusters, and greater cross-cloud
portability. Further, in a world where developers are increasingly
responsible for the data services used in their application stacks, we
believe that there should still be an explicit separation of concerns
with developers having the ability to customize their data lifecycle
but operations having full global visibility to meet business and
legal requirements.

As a response to these concerns, we are introducing Kanister, an
open-source application-level data management framework. The follow
sections describe Kanister's architecture with examples and shows how
it can be used and extended.

Finally, it is important to note that the data lifecycle workflow or
interface does not change when Kanister is used. The same protection,
restore, and migration workflows already explored will work
transparently with Kanister.


.. contents:: Kanister Overview
  :local:

Kanister Design
---------------

Design Goals
++++++++++++

The design of Kanister was driven by five main goals:

#. **Application-Level**: Performing data life management at the
   application-level is the emerging sweet spot given the increasing
   distributed nature of cloud-native data services (e.g., NoSQL systems
   such as MongoDB).

#. **Ops Focused, Dev Friendly**: A lifecycle management solution
   should solve operations needs including global monitoring,
   visibility, and compliance with business requirements while allowing
   self-service and providing standard implementations of common tasks.

#. **Mechanism vs. Policy**: There should be a clean separation of
   mechanism (actual tasks or actions) vs. the policy (scheduling,
   selection, etc.). This will allow for faster independent evolution on
   both sides.

#. **Clean Encapsulation**: There should be a standard well-defined
   API for common lifecycle management tasks. This enables pluggable
   implementations for diverse and evolving data services.

#. **Extensible**: Any solution must be easy to customize or extend
   for custom data services and non-standard requirements. Extensibility
   should be enabled for both developers and operations.

Components
++++++++++

   .. image:: ./_static/kanister_arch.png

The Kanister framework, based on the Kubernetes Operator framework, is
composed of three main components:

#. **Blueprint**: A blueprint that defines the actions that need to be
   performed for each lifecycle stage (e.g., backup data, mask data,
   etc.).

#. **(Optional) Sidecar Container**: For applications that do not come
   with all required tools, a Kanister sidecar container can be added to
   unlock new and requirement functionality (e.g, upload to an object
   store).

#. **Driver**: Kanister is driven by a driver that could be a
   standalone per-application controller, a CLI tool, or a global
   controller embedded into the Kasten platform.

A reference to the blueprint and sidecar are combined in the
application description (e.g., via a Helm Chart, Deployment,
StatefulSet, etc.) while the drive resides as an application either in
the container platform or potentially even external to it.


Installing Kanister-Enabled Applications
----------------------------------------

Before we dive into how Kanister works and can be extended, it is
helpful to see how easy it can be to use. For example, to deploy
MongoDB, an application we have integrated with Kanister, one would
simply use ``Helm`` to run the following command.

.. code-block:: console

   $ helm repo add kasten https://charts.kasten.io/
   $ helm install kasten/kanister-mongodb-replicaset \
             --name my-release \
             --set kanister.cloud_api_key="${AWS_ACCESS_KEY_ID}" \
             --set kanister.cloud_api_secret="${AWS_SECRET_ACCESS_KEY}" \
             --set kanister.bucket_name="${KANISTER_BUCKET_NAME}"

The above Helm Chart, described in more detail below, adds a sidecar
container that can backup data to an S3-compliant object store and
also restore from it. If you are using Google Container Engine (GKE),
you can follow the `Google Cloud Storage documentation
<https://cloud.google.com/storage/docs/migrating#keys>`_ to obtain
AWS S3-API compatible access and secret keys.


Extending Applications with Kanister
------------------------------------

As shown in the below example, extending an application to use
Kanister is usually simple. In this case, we extended the `public
MongoDB ReplicaSet Helm Chart
<https://github.com/kubernetes/charts/tree/master/stable/mongodb-replicaset/>`_
(v1.4.1).

The most minimal change required, as seen in the first part of the
below diff, is an annotation so that the Kanister driver can both
recognize the Kanister-enabled application as well as determine the
blueprint that should be used.

Second, as this application template did not contain tools to
efficiently backup and restore MongoDB data, we add a sidecar
container to the chart. The sidecar container contains `tools to
upload and download artifacts to and from an object store
<https://aws.amazon.com/cli//>`_ and the `Percona MongoDB Consistent
Backup Tool
<https://github.com/Percona-Lab/mongodb_consistent_backup/>`_. This
sidecar container (and Helm chart) is additionally configured to read
secrets it will use to authenticate against the object store.



.. code-block:: diff

 @@ -1,6 +1,8 @@
  apiVersion: apps/v1beta1
  kind: StatefulSet
  metadata:
 +  annotations:
 +    alpha.kanister.io/blueprint: mongo
    labels:
      app: {{ template "name" . }}
      chart: {{ .Chart.Name }}-{{ .Chart.Version }}
 @@ -74,6 +76,23 @@
                readOnly: true
            {{- end }}
        containers:
 +        - name: kanister-sidecar
 +          image: kastenio/kanister-mongodb-sidecar
 +          command: ["tail"]
 +          args:
 +          - "-f"
 +          - "/dev/null"
 +          env:
 +            - name: AWS_ACCESS_KEY_ID
 +              valueFrom:
 +                secretKeyRef:
 +                  name: {{ template "fullname" . }}-aws-creds
 +                  key: aws_access_key_id
 +            - name: AWS_SECRET_ACCESS_KEY
 +              valueFrom:
 +                secretKeyRef:
 +                  name: {{ template "fullname" . }}-aws-creds
 +                  key: aws_secret_access_key
          - name: {{ template "name" . }}
            image: "{{ .Values.image.name }}:{{ .Values.image.tag }}"
            imagePullPolicy: "{{ .Values.image.pullPolicy }}"


Blueprint Definition
--------------------

As mentioned above, a blueprint defines the actions that need to be
performed for each lifecycle stage. Blueprints can be versioned,
namespaced, and operator or developer provided.

More concretely, a blueprint consists of multiple top-level actions
(e.g., backup, restore, migrate, mask data, etc.) where each action
works on a Kubernetes-level application object (e.g., StatefulSet).

In turn, each action is composed of multiple named phases. Each phase
performs some distinctive task and can be templated based on the
metadata found in the application object. Examples of phases include
extracting data from a data service, uploading extracted data to an
object store, or even scaling a StatefulSet up and down. Phases are
executed sequentially and each phase is only invoked if the previous
phase completed successfully.

To illustrate this, we show a simplified version of the MongoDB
blueprint below. In particular, we will look at the backup action and
the two phases associated with it. The full blueprint can be found
:doc:`here </blueprint>`.  The first phase, named
``takeConsistentBackup`` and starting at line 5, is templated and logs
into the Kanister sidecar of the MongoDB StatefulSet it is running
against. It then executes the ``mongodb-consistent-backup`` tool to
generate a database backup. This backup is used in the next phase,
named ``pushToBlobStore`` and starting at line 19, that copies that
artifact over to the previously configured object storage bucket.


.. code-block:: yaml
  :linenos:
  :caption: Backup Action of a MongoDB Blueprint (Simplified)

  actions:
    backup:
      type: StatefulSet
      phases:
      - func: KubeExec
        name: takeConsistentBackup
        args:
          - "{{ .StatefulSet.Namespace }}"
          - "{{ index .StatefulSet.Pods 0 }}"
          - kanister-sidecar
          - bash
          - -c
          - |
            ...
            mongodb-consistent-backup -H ${primary} -P 27017 -n mongo-backup \
              -l /var/lib/mongodb-consistent-backup
      - func: KubeExec
        name: pushToBlobStore
        args:
          - "{{ .StatefulSet.Namespace }}"
          - "{{ index .StatefulSet.Pods 0 }}"
          - kanister-sidecar
          - bash
          - -c
          - |
            base=$(basename "{{ .Artifact.stdout }}")-$(date +"%s")
            file="{{ .Artifact.stdout }}/rs0.tar"
            cloud_path="s3://${KANISTER_BUCKET}/backups/{{ .StatefulSet.Name }}/${base}/rs0.tar"
            aws s3 cp ${file} ${cloud_path} > /dev/null


Adding Custom Blueprints to Kasten
----------------------------------

While we have the ability to add custom blueprints to the Kasten
system, the API is currently still in flux. If this is something you
would like to explore, please reach out and we will be happy to work
with you to enable it in your cluster.

.. toctree::
   :hidden:

   blueprint


.. spelling::
   Kanister
   versioned
   namespaced
   diff
   templated
   metadata
