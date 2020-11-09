:is-up-to-date: True

:orphan:

.. _script-sandbox-configuration:

============================
Script Sandbox Configuration
============================

When a script is executed all code is validated against a blacklist of insecure expressions to prevent code that could
compromise the system. When you try to execute a script that contains insecure expressions you will see an error
similar to this:

.. code-block:: none

  UnsupportedOperationException: Insecure call staticMethod java.lang.Runtime getRuntime ...

|

It is recommended to keep the default configuration if possible. However, if access to one or more of the blacklisted expressions
is required, it is possible to override the blacklist configuration. Configuration is global and affects all scripts on the server.

.. warning:: When you allow a script to make an insecure call you should make sure it can only be executed with known
             arguments and **never** with unverified user input.

|

------------------------
Using a custom blacklist
------------------------

Crafter Engine includes a default blacklist that you can find 
`here <https://github.com/craftercms/engine/blob/develop/src/main/resources/crafter/engine/groovy/blacklist>`_. Make sure you review the branch/tag you're using.

To use a custom blacklist follow these steps:

#.  Copy the default blacklist file to your classpath, for example:
    
    ``CRAFTER_HOME/bin/apache-tomcat/shared/classes/crafter/engine/extension/groovy/blacklist``
    
#.  Remove or comment (adding a ``#`` at the beginning of the line) the expressions that your scripts require
#.  Update the :ref:`server-config.properties <engine-configuration-files>` configuration file to load the custom blacklist:
    
    .. code-block:: none
      :caption: ``CRAFTER_HOME/bin/apache-tomcat/shared/classes/crafter/engine/extension/server-config.properties``
    
      # Use a custom blacklist for the sandbox
      crafter.engine.groovy.sandbox.blacklist=classpath:crafter/engine/extension/groovy/blacklist
    
#.  Restart Crafter CMS

Now you can execute the same script without any issues.


-------------------------------
Adding dependencies with Grapes
-------------------------------

If your Groovy code need to use external dependencies you can use Grapes, however, when the Groovy sandbox is enabled
dependencies can only be downloaded during the initial compilation and not during runtime. For this reason it is
required to add an extra parameter ``initClass=false`` in the annotations to prevent them to be copied to the classes:

.. code-block:: groovy
  :caption: Example grapes annotations

  @Grab(group='org.apache.commons', module='commons-pool2', version='2.8.0', initClass=false)
  
  @Grab(value='org.apache.commons:commons-pool2:2.8.0', initClass=false)

|

----------------------------
Disabling the Groovy Sandbox
----------------------------

It is possible to completely disable the Groovy sandbox for all scripts. To disable the sandbox for all sites update the server configuration file :ref:`server-config.properties <engine-configuration-files>`:

.. code-block:: none
  :caption: *CRAFTER_HOME/bin/apache-tomcat/shared/classes/crafter/engine/extension/server-config.properties*

  # Disable the script sandbox for all sites
  crafter.engine.groovy.sandbox.enable=false
