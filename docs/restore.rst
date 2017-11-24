.. _restore:

Restoring Applications
======================

Once applications have been protected via a policy or a manual action,
it is possible to restore then in-place or clone them into a different
namespace.

.. contents:: Restore Overview
  :local:


Restoring Existing Applications
--------------------------------

Restoring an application is accomplished via the Namespaces page while
restoring a workload can be accomplished via the Workloads page. Using
Namespaces, as an example, one needs to simply click to expose the
dropdown and select ``Restore Namespace``

   .. image:: ./_static/namespaces_manual.png

At that point, you will have an option to pick a Restore Point, a
grouped collection of data artifacts belonging to the application, to
restore from. This view also easily distinguishes manually generated
restore points from automated policy-generated ones.

   .. image:: ./_static/namespaces_restore.png

Selecting a restore point will bring up a side-panel containing more
details on the restore point.

   .. image:: ./_static/namespaces_restore_panel.png

Restoring Deleted Applications
------------------------------

The process of restoring a deleted namespace or workload is nearly
identical to the above process. The only difference is that, by
default, removed applications are not shown on the Namespaces or
Workloads page. To discover them, you simply need to filter and
select ``Removed``.

   .. image:: ./_static/restore_deleted_filter.png

Once the filter is in effect, you will see namespaces that K10 has
previously protected but no longer exist. These can now be restored by
clicking on the dropdown.

   .. image:: ./_static/restore_deleted_selection.png


Cloning Into A New Namespace
----------------------------

By default, the system will attempt to restore the application into
the original namespace the restore point was created from. However, as
the above image shows, the target namespace can be changed and new
namespaces can even be created at this point. In particular, this
functionality can be used to clone an application or workload for
debugging or test/dev purposes.

The Restore Process
-------------------

Once you click ``Restore``, the system will automatically recreate the
entire application stack into the selected namespace. This not only
includes the data associated with the original application but also
the versioned container images.

.. note:: Restore can take a few minutes as this depends on the amount
  of data captured by the restore point. The restore time is dominated
  by how long it takes to rehydrate this data in the underlying cloud
  provider followed by recreating the application containers. Please
  be patient.

After restore completes, you will be able to go back to your
application and verify that the state was restored to what existed at
the time the restore point was obtained.


.. spelling::
  namespaces
  Namespaces
  dev
  versioned
