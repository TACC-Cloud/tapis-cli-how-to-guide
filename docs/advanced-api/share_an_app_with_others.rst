Share an App with Others
========================

As a standard user of Tapis tenants, you have permissions to build and deploy
private apps only. Private apps are, by default, only visible to you. To share
an app with your colleagues, follow the steps below.

Update Permissions on an App
----------------------------

Assuming you have a private app called ``taccuser-fastqc_app-0.11.9`` (developed
`earlier <create_a_custom_app.html>`__
in this how-to guide), you can check who has permissions to access the app with
the following command:

.. code-block:: bash

   $ tapis apps pems list taccuser-fastqc_app-0.11.9
   +----------+------+-------+---------+
   | username | read | write | execute |
   +----------+------+-------+---------+
   | taccuser | True | True  | True    |
   +----------+------+-------+---------+


By default, the creator of an app is the only one with read, write, or execute
privileges. Next, identify the tenant username of the user with whom you would
like to share the app.

.. tip::

   See
   `this page <../api-essentials/share_data_with_others.html#find-another-user>`__
   for an example on how to find another user's username

Given the username ``jdoe``, grant that user permissions with:

.. code-block:: bash

   $ tapis app pems grant taccuser-fastqc_app-0.11.9 jdoe ALL
   +----------+------+-------+---------+
   | username | read | write | execute |
   +----------+------+-------+---------+
   | taccuser | True | True  | True    |
   | jdoe     | True | True  | True    |
   +----------+------+-------+---------+


Ask your collaborator (``jdoe``) to perform the ``tapis apps list`` command, and
they should now be able to see your app.


Update Permissions on an Execution System
-----------------------------------------

Before your collaborator (``jdoe``) can run a job with your private app, they
must also have correct permissions on the execution system associated with the
app.

.. code-block:: bash

   # Find the execution system associated with your private app
   $ tapis apps show -c executionSystem taccuser-fastqc_app-0.11.9
   +-----------------+-------------------------+
   | Field           | Value                   |
   +-----------------+-------------------------+
   | executionSystem | tacc.stampede2.taccuser |
   +-----------------+-------------------------+

   # List permissions on the execution system
   $ tapis systems roles list tacc.stampede2.taccuser
   +----------+-------+
   | username | role  |
   +----------+-------+
   | taccuser | OWNER |
   +----------+-------+

   # Grant 'USER' permission to your collaborator
   $ tapis systems roles grant tacc.stampede2.taccuser jdoe USER
   +----------+-------+
   | username | role  |
   +----------+-------+
   | taccuser | OWNER |
   | jdoe     | USER  |
   +----------+-------+


Ask your collaborator to perform the ``tapis systems list`` command, and they
should now be able to see your private system. Now, your collaborator can run
jobs against your private app using the same ``job.json`` file and
``tapis jobs submit`` commands as you.


Publish the App Globally
------------------------

Standard users do not have the appropriate permissions to make an app public,
thereby sharing it with all users on the tenant. If you have deployed and tested
an app, and think it would be of general use to the community, please contact
your
`tenant admin <../getting-started/request_access_to_a_tenant.html>`__
to ask for information on publishing your app.
