Create a Custom App
===================


You can find Tapis applications ("apps") in your tenant's catalog by using the
``tapis apps list`` command described
`previously <../api-essentials/find_an_application.html>`__.
If the app you are looking for is not available, you can create your own and add
it to the catalog.

Components of an App
--------------------

The essential components you need to create your own app are:

1. An app bundle directory containing definitions and assets for the app
2. A Docker image containing the executable and all runtime dependencies
3. A wrapper script (written in bash) that runs the executable

Create an App by Example
------------------------

The best way to demonstrate the creation of a custom app is by example. The
following sub-pages will go through the process:

.. toctree::
   :maxdepth: 1

   app-building/initialize_the_app_directory
   app-building/containerize_the_executable
   app-building/deploy_and_test_the_app
   app-building/best_practices_and_next_steps
