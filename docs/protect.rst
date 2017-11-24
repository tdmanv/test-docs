.. _protect:

Protecting Applications
=======================

To help with the explanation, we will use the multi-volume `GitLab
<https://gitlab.com>`_ application (available via a Helm chart) as a
running example for this and the following sections of the
documentation.  In particular, we will define policies to protect this
application, see the automated policy kick in, test taking a manual
snapshot, deleting data, and then restoring from one of the previous
snapshots. However, the below documentation also applies to all other
stateful applications you might chose to install and we will call out
other specialized workflows that are possible for a variety of
different deployment scenarios.

.. contents:: Protection Overview
  :local:


Policy-Based Protection
-----------------------

As you can see when you look at the main dashboard, compliance is at
0% because we have no policies defined to cover any installed
applications. To fix this, please click on Namespaces card on the main
dashboard. As documented earlier, this will take you to a page where
you can see all stateful applications in the system and the number of
workloads they are composed of:

   .. image:: ./_static/overview_ns.png

To protect any unmanaged application, simple click ``Create a policy``
and that will take you to the policy creation section:

   .. image:: ./_static/policies_create.png


.. note:: It is not required that a policy be created to operate at
  the namespace level. In fact, as can be seen from the Workloads
  page, policies can be restricted to a particular workload or a
  particular set of selectors that could apply to only a subset of the
  workloads available in a namespace.

Give your policy a name, add Snapshot as an action, and then set the
action frequency (we recommend hourly). Once you pick a frequency, you
have the option of modifying the retention schedule by simply editing
the displayed numbers.

Notice that the policy creation panel automatically selected the
namespace and included all objects in this namespace as this policy
creation was initiated through the Namespaces page. You can click on
``Select Objects`` to obtain more granular control over labels used
within the policy if desired:

   .. image:: ./_static/policies_selectors.png

Through the judicious use of labels within your application and, in
turn as selectors (they are logically ORed together), you can create
wildcard or forward-looking policies that will apply to any addition
of workloads or applications into the given namespace. For example,
using the ``heritage: Tiller`` selector will apply the policy you are
creating to any new ``Helm``-deployed application as that tool
automatically adds that label to any workload it creates.

Once you are done with policy configuration, simply hit ``Create
Policy``!

Viewing Policy Activity
+++++++++++++++++++++++

Once you are back on the main dashboard, if you pay careful attention
you will see both the applications and volumes sections quickly switch
from unmanaged to non-compliant (i.e., a policy covers the objects but
no action has been taken yet). Soon after, they will both switch to
compliant as snapshots get invoked. You can also scroll down on the
page to see the activity, how long each snapshot took, and the
generated artifacts. Your page will now look similar to this:

   .. image:: ./_static/dashboard_compliant.png

More detailed job information can be obtained by clicking on the
in-progress or completed jobs.

This activity can be further seen by going back to the Policies page
and selecting "Snapshots schedule" for the policy you just
created. It will show the following view including the protected
workloads and an easy view to see compliance for a given policy's
schedule.

   .. image:: ./_static/policies_snapshot_schedule.png

At this point, the application is protected and you can now move on to
testing manual snapshots and restores in the next section of the
documentation.


Manual Protection
-----------------

While the previous restore point created right after policy creation
will work, we can also optionally experiment with taking a manual
snapshot. To do so, go back to the dashboard and click on the
Namespaces card. For the selected application, select the dropdown:

   .. image:: ./_static/namespaces_manual.png

and then select ``Manually Protect Namespace``. This starts the
creation of a manual restore point and you can check the progress on
the dashboard. Once the manual snapshot has completed, you can modify
state in the selected application.

Note that you can perform the same manual action on an individual
workload too or even on an application or workload that has no policy
associated with it.

.. spelling::
  Namespaces
  wildcard
  ORed
