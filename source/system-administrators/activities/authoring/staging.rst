:is-up-to-date: True

.. index:: Crafter CMS Intermediate Publishing Target, Staging

.. _staging:

==========================================
Crafter CMS Intermediate Publishing Target
==========================================

Crafter CMS supports an intermediate publishing target to publish to, named ``staging``, where the site can be fully exercised with regards to system and integration points.  Staging allows testing of your site.  Once validated in staging, items can be pushed live.  ``staging``  precedes ``live`` .

When staging is setup, the **Request Publish** and **Approve for Publish** dialogs gives the user the option to select where to publish to:

.. image:: /_static/images/system-admin/staging-publish-option.png
    :width: 75 %
    :align: center
    :alt: System Administrator - Staging Setup for Publish dialogs

|

Users with access to the following widgets in the dashboard will see a column named **Publishing Target** indicating which target an item has been/would be published to: ``Items Waiting For Approval``, ``Approved Scheduled Items`` and ``Recently Published``

.. image:: /_static/images/system-admin/staging-dashboard.png
    :width: 75 %
    :align: center
    :alt: System Administrator - Dashboard

Notice the pages published to ``staging`` and ``live`` with the same time in the above image.  When the intermediate publishing target is setup, all items will go through ``staging`` before going to ``live``.  So, if a user approves an item to be published directly to live, it will be published to ``staging`` first, then onto ``live``.


------------------------------------------------------
Setting Up the Intermediate Publishing Target per Site
------------------------------------------------------

The intermediate publishing target is setup per site and by default is not enabled.  To enable the intermediate publishing target for your site, click on |siteConfig| from the bottom of the **Sidebar**, then click on **Configuration** and select **Site Configuration** from the dropdown list.

Under the **<published-repository>** tags, set **<enable-staging-environment>** to ``true``.  You can also change the default names for your staging environment and your live environment repos here.

   .. code-block:: xml
       :linenos:

       <published-repository>
          <enable-staging-environment>true</enable-staging-environment>
          <staging-environment>staging</staging-environment>
          <live-environment>live</live-environment>
       </published-repository>

|

To view your site with respect to ``staging``, we need to set it up for delivery.  To setup your site for ``staging``, run the ``init-site`` script and follow the instructions in :ref:`setup-site-for-delivery` using the staging branch in the options.

    .. code-block:: bash

        ./init-site.sh -b staging mysite-staging /path/to/published/repo

|

---------------------------------
Setting Default Repo Branch Names
---------------------------------

Crafter CMS allows you to set default branch names for ``live`` and ``staging`` when not using the *environment-config.xml*.  In your Authoring installation, go to ``shared/classes/crafter/studio/extension`` and open the file ``studio-config-override.yaml``. The current default names are **live** and **staging**.  To change the default names, uncomment and change the defaults set:

.. code-block:: yaml
   :caption: shared/classes/crafter/studio/extension/studio-config-override.yaml
   :linenos:

   # If not using environment-config.xml, environments are configured here
   # Git repository branch for the `live` environment, default "live"
   # studio.repo.published.live: live
   # Git repository branch for the `staging` environment, default "staging"
   # studio.repo.published.staging: staging

.. note::

   Studio uses the properties described above (``studio-config-override.yaml`` file) when staging is enabled and the names for ``<live-environment/>`` and ``<staging-environment/>`` are not present in the Site Configuration (*site-config.xml* file).
   When staging is disabled, the published environment name is read from the Environment Configuration (*environment-config.xml* file).