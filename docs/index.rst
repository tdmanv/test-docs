.. K10 documentation master file, created by
   sphinx-quickstart on Thu May 25 15:27:06 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Documentation Overview
======================

Kasten's K10 platform is a unique application-centric data management
solution for all public and private Kubernetes deployments that
balances the needs of operators and developers making it easier to
build, deploy, and manage stateful containerized applications at
scale.

Through a top-down application focus, K10 provides the right balance
and separation of concerns between developers and operators. In
particular, the current release of the platform focuses on the
following four use cases:

|

   .. image:: ./_static/index_usecases.jpg

|

**Test/Dev Environments**
  K10 supports the safe and automated export of data from production
  to test and dev environments with data masking support. In addition,
  it also provides the ability to clone applications and namespaces to
  help with debugging and workspace suspend/resume workloads.

**Cloud Migration**
  To support disaster-recovery and ultimately cross-cloud portability,
  K10 allows for the seamless migration of applications with their
  data across non-federated Kubernetes clusters. These clusters can be
  under the same account, in different accounts, or even different
  accounts across different cloud provider regions.

**Backup and Recovery**
  The K10 platform supports protecting applications both at the volume
  level and, through the use of an open-source framework called
  Kanister, at the application/data-service level. Based on
  label-based criteria, K10 can protect applications today against
  accidental and malicious data loss.

**Disaster Recovery**
  To assist in the fast recovery from site or cluster outages, K10
  supports simple orchestrated recovery that can be full or partial.


Quickstart
----------

If you want a high-level introduction to the system, we recommend
starting with the :ref:`quickstart` guide.

If you are using this documentation for our test drive system, we
recommend you get started with the :ref:`testdrive` documentation
first.

Guided Walkthrough
------------------

For a more detailed documentation-driven walkthrough of the system,
the recommended sequence of actions is:

#. :ref:`install`
#. :ref:`overview`
#. :ref:`protect`
#. :ref:`restore`
#. :ref:`migration`
#. :ref:`kanister`

Support
=======

You can file bug reports or get help by sending email to our `support <support@kasten.io>`_
alias. You can also call at any time for either support or
feedback: +1-412-370-2852.


.. toctree::
   :hidden:

   self
   quickstart
   install
   overview
   protect
   restore
   migration
   kanister
   restrictions
   testdrive

.. spelling::
   dev
   Kanister
   namespaces
