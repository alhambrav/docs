.. index:: API; Crafter Studio

.. _crafter-studio-api:

==================
Crafter Studio API
==================

------------------
Content Management
------------------

.. toctree::
	:maxdepth: 1

	content/create

---------------
User Management
---------------

.. toctree::
	:maxdepth: 1

	user/create
	user/get
	user/list
	user/list-by-org
	user/list-by-project
	user/update
	user/delete
	user/enable
	user/disable
	user/status

----------------
Group Management
----------------

.. toctree::
	:maxdepth: 1

	group/create
	group/get
	group/get-by-org
	group/get-users
	group/update
	group/delete
	group/add-user
	group/remove-user

---------------
Role Management
---------------

.. toctree::
   :maxdepth: 1

	role/create
	role/get
	role/get-by-project
	role/get-users
	role/update
	role/delete
	role/add-user
	role/remove-user

------------------
Project Management
------------------

.. toctree::
	:maxdepth: 1

	project/create
	project/get
	project/list
	project/list-by-org
	project/list-by-user

---------------
Repo Management
---------------

.. toctree::
   :maxdepth: 1

   repo/sync-from-repo
   repo/rebuild-database


--------
Security
--------

^^^^^^^^^^^^^^
Authentication
^^^^^^^^^^^^^^

.. toctree::
   :maxdepth: 1

	security/authentication/login
	security/authentication/logout
	security/authentication/validate-session
	security/authentication/forgot-password
	security/authentication/validate-token
	security/authentication/set-password
	security/authentication/change-password
	security/authentication/reset-password

^^^^^^^^^^^^^
Authorization
^^^^^^^^^^^^^

Policy
^^^^^^

.. toctree::
   :maxdepth: 1

	security/authorization/policy/create
	security/authorization/policy/get
	security/authorization/policy/list
	security/authorization/policy/list-by-org
	security/authorization/policy/list-users
	security/authorization/policy/update
	security/authorization/policy/delete

-----
Audit
-----

.. toctree::
	:maxdepth: 1

	audit/system
	audit/organization
	audit/project

----------
Monitoring
----------

.. toctree::
   :maxdepth: 1

   monitor/version
   monitor/status
   monitor/memory

----
CMIS
----

.. toctree::
   :maxdepth: 1

   cmis/list
   cmis/search
   cmis/clone

-------
Publish
-------

.. toctree::
   :maxdepth: 1

	publish/status
	publish/start
	publish/stop

--------
Activity
--------

.. toctree::
   :maxdepth: 1

   activity/get-user-activity
   activity/post-activity

---------
Clipboard
---------

.. toctree::
   :maxdepth: 1

   clipboard/copy-item
   clipboard/cut-item
   clipboard/get-items
   clipboard/paste-item

----------
Dependency
----------
.. toctree::
   :maxdepth: 1

   dependency/get-dependant
   dependency/get-dependencies

----------
Deployment
----------
.. toctree::
   :maxdepth: 1

   deployment/bulk-golive
   deployment/get-available-publishing-channels
   deployment/get-deployment-history
   deployment/get-scheduled-items

-------
Preview
-------
.. toctree::
   :maxdepth: 1

   preview/sync-site

---------------
Content / Asset
---------------

.. toctree::
   :maxdepth: 1

   content/change-content-type
   content/content-exists
   content/create-folder
   content/delete-content
   content/get-content
   content/get-content-at-path
   content/get-content-type
   content/get-content-types
   content/get-item
   content/get-item-orders
   content/get-item-states
   content/get-item-versions
   content/get-items-tree
   content/reorder-items
   content/revert-content
   content/unlock-content

------
Server
------
.. toctree::
   :maxdepth: 1

   server/get-available-languages
   server/get-loggers
   server/get-ui-resource-override
   server/set-logger-state
