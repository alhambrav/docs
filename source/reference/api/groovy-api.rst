:is-up-to-date: True
:last-updated: 4.1.0

.. highlight:: groovy
   :linenothreshold: 5

.. index:: Groovy, Groovy API, Custom Services, Services, Controllers, Unit Testing

.. _groovy-api:

==================
Groovy Development
==================
.. contents::
    :local:
    :depth: 2

CrafterCMS supports server-side development with Groovy. By using Groovy, you can create RESTful services, MVC controllers, code that runs before a page or component is rendered, servlet filters, scheduled jobs, and entire backend applications.

----------
Groovy API
----------
CrafterCMS provides a number of useful global variables that can be used in all the different types of scripts available:

.. include:: /includes/global-groovy-variables.rst

There are also several other variables available to scripts that are executed during the scope of a request (REST scripts, controller
scripts, page/component scripts and filter scripts):

.. include:: /includes/request-groovy-variables.rst

All scripts are executed in a sandbox to prevent insecure code from running, to change the configuration see
:ref:`groovy-sandbox-configuration`

To create unit tests for your groovy code, see :ref:`unit-testing-groovy-code`

----------------
Types of Scripts
----------------
There are different types of scripts you can create, depending on the subfolder under Scripts where they're placed. The following are
the ones currently supported:

^^^^^^^^^^^^
REST Scripts
^^^^^^^^^^^^
REST scripts function just like RESTful services. They just need to return the object to serialize
back to the caller. REST scripts must be placed in any folder under Scripts > rest.

A REST script URL has the following format: it starts with /api/1/services, then contains all the folders that are part of the hierarchy
for the particular script, and ends with the script name, the HTTP method and the .groovy extension. So, a script file at
Scripts > rest > myfolder > myscript.get.groovy will respond to GET method calls at http://mysite/api/1/services/myfolder/myscript.json.

The following is a very simple sample script that returns a date attribute saved in the session. If no attribute is set yet, the current
date is set as the attribute. Assume that the REST script exists under Scripts > rest > session_date.get.groovy
::

    import java.util.Date

    if (!session) {
    	session = request.getSession(true)
    }

    def date = session.getAttribute("date")
    if (!date) {
    	date = new Date()

    	session.setAttribute("date", date)
    }

    return ["date": date]

""""""""""""""""
Script Not Found
""""""""""""""""
Rest scripts will return the ``404`` page when a script is not found.
Developers will still be able to return custom ``404`` responses from rest scripts. e.g.:

.. code-block:: groovy

    response.setStatus(404)
    return 'This is the custom message'

If desired, they could even conditionally send the default response page as well by using ``sendError`` instead:

.. code-block:: groovy

    response.sendError(404)

^^^^^^^^^^^^^^^^^^
Controller Scripts
^^^^^^^^^^^^^^^^^^
Controller scripts are very similar to REST scripts. They have the same variables available, but instead of returning an object,
they return a string with the view to render. Most of the time, this is just the template path, like in the following snippet:

::

    return "/templates/web/registration.ftl"

Controller scripts basically work like a controller in the MVC pattern. They should be put under Scripts > controllers,
and similarly to REST scripts, the URL is made up of the directory hierarchy, the script name and the HTTP method. For example,
a script at Scripts > controllers > myfolder > mycontroller.get.groovy will respond to GET calls at http://mysite/myfolder/mycontroller.

The following is a very simple example script that will do the sum of 2 parameters, put the result in the ``templateModel`` and return
the path of the FTL template that will render the result:

::

    templateModel.result = Integer.parseInt(params.num1) + Integer.parseInt(params.num2)

    return "/templates/web/sum.ftl"

One very common controller script is the sitemap.groovy. A sitemap is used by search engines to have a better idea of how to "crawl"
a website. A sitemap is an XML with references to most of the site's pages, and basically looks like this:

.. code-block:: xml
    :linenos:

    <?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
        <url>
            <loc>http://www.domain.com /</loc>
            <lastmod>2008-01-01</lastmod>
            <changefreq>weekly</changefreq>
            <priority>0.8</priority>
        </url>
        <url>
            <loc>http://www.domain.com/catalog?item=vacation_hawaii</loc>
            <changefreq>weekly</changefreq>
        </url>
        <url>
            <loc>http://www.domain.com/catalog?item=vacation_new_zealand</loc>
            <lastmod>2008-12-23</lastmod>
            <changefreq>weekly</changefreq>
        </url>
        <url>
            <loc>http://www.domain.com/catalog?item=vacation_newfoundland</loc>
            <lastmod>2008-12-23T18:00:15+00:00</lastmod>
            <priority>0.3</priority>
        </url>
        <url>
            <loc>http://www.domain.com/catalog?item=vacation_usa</loc>
            <lastmod>2008-11-23</lastmod>
        </url>
    </urlset>

Search engines look for the sitemap just after the domain, so a sitemap URL would look like www.domain.com/sitemap. The sitemap
controller then must be placed in Scripts > controllers > sitemap.groovy. The code would be similar to this:

::

    import groovy.xml.MarkupBuilder
    import groovy.xml.MarkupBuilderHelper

    def sitemap = []
    def excludeContentTypes = ['/component/level-descriptor']

    parseSiteItem = { siteItem ->
        if (siteItem.isFolder()) {
            def children = siteItem.childItems;
            children.each { child ->
                parseSiteItem(child);
            }
        } else {
            def contentType = siteItem.queryValue('content-type')
            if (!excludeContentTypes.contains(contentType)) {
                def storeUrl = siteItem.getStoreUrl();
                def location = urlTransformationService.transform('storeUrlToFullRenderUrl', storeUrl);
                sitemap.add(location);
            }
        }
    }

    def siteTree = siteItemService.getSiteTree("/site/website", -1)
    if (siteTree) {
        def items = siteTree.childItems;
        items.each { siteItem ->
            parseSiteItem(siteItem);
        }
    }

    response.setContentType("application/xml;charset=UTF-8")

    def writer = response.getWriter()
    def xml = new MarkupBuilder(writer)
    def xmlHelper = new MarkupBuilderHelper(xml)

    xmlHelper.xmlDeclaration(version:"1.0", encoding:"UTF-8")

    xml.urlset(xmlns:"http://www.sitemaps.org/schemas/sitemap/0.9") {
        sitemap.each { location ->
            url {
                loc(location)
                changefreq("daily")
            }
        }
    }

    response.flushBuffer()

    return null


.. _unit-testing-groovy-code:

-----------------------------------
Unit Testing CrafterCMS Groovy Code
-----------------------------------
For larger sites with complex services implemented in Groovy, it is very helpful to have a way to include unit tests in a way that can be easily integrated with CI/CD systems.

This section details how to create unit tests for CrafterCMS Groovy code with Gradle.

For more information on the classes of the variables that can be mocked for unit testing, see :ref:`above <groovy-api>`

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Steps for Creating Groovy Unit Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To create a unit test:

#. Designate a folder for all test related files
#. Write your unit test code
#. Setup your unit test to run with Gradle
#. Execute your unit test

"""""""""""""""""""""""""""""""""""""""""""
Designate Folder for All Test Related Files
"""""""""""""""""""""""""""""""""""""""""""
Designate a folder in the site repository to contain all test related files. For example:

   .. code-block:: text
      :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox*

      /scripts
         /test
           /classes   (all packages & groovy classes for testing)
           /resources (all additional files required for testing)

   |


This structure is the equivalent of the standard folders used for Java/Groovy projects:

   .. code-block:: text

      /src/test/groovy
      /src/test/resources

   |

"""""""""""""""""""""""""
Write Your Unit Test Code
"""""""""""""""""""""""""
There are no restrictions or requirements for the unit test code, developers can choose any testing framework supported by the build tool. Examples: `spring-test <http://docs.spring.io/spring-batch/reference/html/testing.html>`__, `junit <http://junit.org/>`__, `testng <https://testng.org/doc/index.html>`__, `spock <https://spockframework.org/>`__

Remember when writing unit test code, developers will be responsible for:

- Choosing & configuring the testing framework
- Making sure all required dependencies are included (for example external jars)
- Mocking all Crafter Engine classes used by the classes under testing


"""""""""""""""""""""""""""""""""""""""
Setup Your Unit Test to Run With Gradle
"""""""""""""""""""""""""""""""""""""""
To use Gradle the only requirement is to add a ``build.gradle`` in the root folder of the site repository and execute the ``test`` task. Example:

.. code-block:: groovy
   :force:
   :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/build.gradle*
   :linenos:

   // Enable Gradle’s Groovy plugin
   plugins {
     id 'groovy'
   }

   sourceSets {
     // Add the site Groovy classes that will be tested
     main {
       groovy {
         srcDir 'scripts/classes'
       }
     }

     // Add the Groovy classes & resources to perform the tests
     test {
       groovy {
         srcDir 'scripts/test/classes'
       }
       resources {
         srcDir 'scripts/test/resources'
       }
     }
   }

   // Enable the testing framework of choice
   test {
     useJUnit()
   }

   repositories {
     mavenCentral()
   }

   // Include the required dependencies
   dependencies {
     // This dependency is required for two reasons:
     // 1. Make Engine’s classes available for compilation & testing
     // 2. Include the Groovy dependencies required by Gradle
     implementation 'org.craftercms:crafter-engine:4.1.1:classes'

     // Include the chosen testing dependencies
     testImplementation 'junit:junit:4.13.2'
     testImplementation 'org.mockito:mockito-core:4.11.0'
   }

|

""""""""""""""""""""""
Execute Your Unit Test
""""""""""""""""""""""
Given the previous example the tests can be executed using a single command:

   .. code-block:: bash

      gradle test

   |

^^^^^^^
Example
^^^^^^^
Let's take a look at an example of a groovy unit test in a site created using the empty blueprint with a custom groovy script, ``MyService``

.. image:: /_static/images/developer/unit-test/unit-test-groovy-sample-service.webp
    :alt: Unit Testing Groovy - Sample Service
    :width: 35 %
    :align: center

|

.. literalinclude:: /_static/code/developer/groovy-unit-test/MyService.groovy
   :language: groovy
   :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/scripts/classes/org/company/site/api/MyService.groovy*
   :linenos:

|

.. literalinclude:: /_static/code/developer/groovy-unit-test/MySearchService.groovy
   :language: groovy
   :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/scripts/classes/org/company/site/api/MySearchService.groovy*
   :linenos:

|

.. literalinclude:: /_static/code/developer/groovy-unit-test/ExternalApi.groovy
   :language: groovy
   :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/scripts/classes/org/company/site/api/ExternalApi.groovy*
   :linenos:

|

.. literalinclude:: /_static/code/developer/groovy-unit-test/MyServiceImpl.groovy
   :language: groovy
   :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/scripts/classes/org/company/site/impl/MyServiceImpl.groovy*
   :linenos:

|

Let's begin creating our unit test for ``MyService``

"""""""""""""""""""""""""""""""""""""""""""
Designate Folder for All Test Related Files
"""""""""""""""""""""""""""""""""""""""""""
The first thing we need to do is to designate a folder for all test related files. We'll designate the ``/scripts/test`` folder to be used for all test related files.

"""""""""""""""""""""""""
Write Your Unit Test Code
"""""""""""""""""""""""""
Next, we'll write the unit test code.

.. image:: /_static/images/developer/unit-test/unit-test-groovy-sample-unit-test.webp
   :alt: Unit Testing Groovy - Sample Service
   :width: 35 %
   :align: center

|

.. literalinclude:: /_static/code/developer/groovy-unit-test/MyServiceImplTest.groovy
   :language: groovy
   :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/scripts/test/classes/org/company/impl/MyServiceImplTest.groovy*
   :linenos:

|

"""""""""""""""""""""""""""""""""""""""
Setup Your Unit Test to Run With Gradle
"""""""""""""""""""""""""""""""""""""""
We'll now setup our unit test to run with Gradle, by adding a ``build.gradle`` file in the root folder of the site repository and execute the ``test`` task.

.. literalinclude:: /_static/code/developer/groovy-unit-test/build.gradle
   :language: groovy
   :force:
   :caption: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/build.gradle*
   :linenos:

""""""""""""""""""""""
Execute Your Unit Test
""""""""""""""""""""""
Finally, we can run our unit test by running ``gradle test``

   .. code-block:: bash
      :caption: *Output when running unit test*

      $ gradle test

      BUILD SUCCESSFUL in 4s
      3 actionable tasks: 3 up-to-date

   |

Let's take a look at the result of our unit test which can be found here: *CRAFTER_HOME/data/repos/sites/SITENAME/sandbox/build/reports/tests/test/index.html*

.. image:: /_static/images/developer/unit-test/unit-test-build-result.webp
   :alt: Unit Testing Groovy - Unit test  build report
   :width: 75 %
   :align: center

|

--------
See Also
--------
- :ref:`configure-custom-services`