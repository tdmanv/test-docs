.. _restrictions:

Restrictions
============

Here is an list of limitations on the current product. The majority of
them will be addressed in an upcoming release.

* Secrets during restores, migration, clones.
* Installing custom Kanister blueprints requires support from
  us. However, if you are a power user, we would be happy to help and
  supply documentation for this.
* Cross-cloud migration only with Kanister
* Cross-project migration within Google currently not supported
* Kanister bucket access for migration

Migration Restrictions
----------------------

The current migration implementation has a few restrictions on use.

* Cross-cloud Migration: Cross-cloud migration is only supported at
  the application-level via :doc:`Kanister </kanister>`. Volume
  migration will be introduced in a later release.

* Export policies: Scheduling exports in a policy-driven automated
  manner is still under development and will be exposed in an upcoming
  release.

.. spelling::

   reprioritized
   architected
   Kanister
