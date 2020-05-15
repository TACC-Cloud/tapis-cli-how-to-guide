Interact with Systems
=====================

A Tapis **system** is a server or collection of servers associated with a single
hostname. They may be public or private, and they may either be **storage**
systems (used for storing files) or **execution** systems (used for running
jobs).

On some tenants, users are provided private storage and execution systems
attached to TACC resources. If you would like to create your own systems, skip
ahead to
`Create a Private System <../advanced-api/create_a_private_system.html>`__.

.. warning::

   The **tacc.prod** tenant likely does not contain default systems for your
   username. Skip ahead to
   `Create a Private System <create_a_private_system.html>`_
   before working through this guide.


Find Systems
------------

Systems can be discovered using the ``tapis systems list`` command:

.. code-block::

   $ tapis systems list
   +-------------------------------+-------------------------------------------------+-----------+---------+
   | id                            | name                                            | type      | default |
   +-------------------------------+-------------------------------------------------+-----------+---------+
   | tacc.stampede2.taccuser       | Execution system for TACC Stampede2             | EXECUTION | False   |
   | tacc.work.taccuser            | Storage system for TACC work directory          | STORAGE   | False   |
   | docking.storage               | Storage VM for the drug discovery portal        | STORAGE   | True    |
   | tapis.storage.system          | Tapis storage system                            | STORAGE   | False   |
   | tapis.execution.system        | Tapis execution system                          | EXECUTION | False   |
   | csc.2018.storage              | Storage system for CSC 2018 Institute           | STORAGE   | False   |
   | docking.exec.ls5              | Lonestar 5 Supercomputer                        | EXECUTION | False   |
   | docking.slurm.stampede        | Docking's Stampede SLURM Execution Host         | EXECUTION | False   |
   | docking.work.storage.stampede | Stampede $WORK storage                          | STORAGE   | False   |
   | docking.fork.fleming          | Fork Execution VM for the drug discovery portal | EXECUTION | False   |
   | docking.storage2              | Storage VM for the drug discovery portal        | STORAGE   | False   |
   | docking.exec.lonestar         | Lonestar 4 Supercomputer                        | EXECUTION | False   |
   +-------------------------------+-------------------------------------------------+-----------+---------+

On this tenant, the user :code:`taccuser` sees numerous storage and execution
systems. All systems are either public or private (with varying degrees of
privacy), each of which are shown here.

On tenants with many more systems, it may be useful to narrow the list with
the ``tapis systems search`` command. For example, to discover systems only
owned by you:

.. code-block:: bash

   $ tapis systems search --owner eq taccuser
   +-------------------------+----------------------------------------+-----------+---------+
   | id                      | name                                   | type      | default |
   +-------------------------+----------------------------------------+-----------+---------+
   | tacc.stampede2.taccuser | Execution system for TACC Stampede2    | EXECUTION | False   |
   | tacc.work.taccuser      | Storage system for TACC work directory | STORAGE   | False   |
   +-------------------------+----------------------------------------+-----------+---------+


Other useful searches might include:

.. code-block:: bash

   $ tapis systems search --owner neq taccuser    # systems you don't own
   $ tapis systems search --public eq true          # only public systems
   $ tapis systems search --public eq false         # only private systems
   $ tapis systems search --type eq EXECUTION       # only execution systems
   $ tapis systems search --type eq STORAGE         # only storage systems
   $ tapis systems search --default eq true         # your default system


You can also chain multiple search parameters. For example, to display only
private storage systems:

.. code-block:: bash

   $ tapis systems search --type eq STORAGE --public eq false
   +--------------------+----------------------------------------+---------+---------+
   | id                 | name                                   | type    | default |
   +--------------------+----------------------------------------+---------+---------+
   | tacc.work.taccuser | Storage system for TACC work directory | STORAGE | False   |
   +--------------------+----------------------------------------+---------+---------+


.. note::

   Don't forget to replace instances of **taccuser** with your actual username
   on the tenant



Discover System Hostname and Default Paths
------------------------------------------

To see additional system information, including hostname, root folder locations,
and availability, use the ``tapis systems show`` command with a system ID. For
example, find out more information about your personal storage system as
follows:



.. code-block:: bash

   $ tapis systems show -f json tacc.work.taccuser

.. code-block:: json
   :linenos:
   :emphasize-lines: 32,33

   {
     "id": "tacc.work.taccuser",
     "name": "Storage system for the TACC WORK directory",
     "type": "STORAGE",
     "default": false,
     "available": true,
     "description": "Storage system for the TACC WORK directory via Stampede2",
     "environment": null,
     "executionType": null,
     "globalDefault": false,
     "lastModified": "7 hours ago",
     "login": null,
     "maxSystemJobs": null,
     "maxSystemJobsPerUser": null,
     "owner": "taccuser",
     "public": false,
     "queues": null,
     "revision": 1,
     "scheduler": null,
     "scratchDir": null,
     "site": null,
     "status": "UP",
     "storage": {
       "proxy": null,
       "protocol": "SFTP",
       "mirror": false,
       "port": 22,
       "auth": {
         "type": "SSHKEYS"
       },
       "publicAppsDir": null,
       "host": "stampede2.tacc.utexas.edu",
       "rootDir": "/work/01234/taccuser",
       "homeDir": "/"
     },
     "uuid": "383424038079107562-242ac112-0001-006",
     "workDir": null
   }


The :code:`-f json` flag was provided to render all information about the
system. As described above, this storage system is a gateway to your private
storage space on the TACC `$WORK` filesystem. The `rootDir` is the virtual root
path for operations performed on this system. The highlighted lines emphasize
the host and root directory when performing operations against this system.

In addition to standard host and path information, execution systems also contain
information about queue types and availability.


Check System Status
-------------------

It may be useful to check the status (availability) of a system in a scriptable
way prior to, e.g., uploading files as part of a pipeline. The following command
can be used with extra flags to strip out the useful part of the response:

.. code-block:: bash

   $ tapis systems show -c status -f value tacc.work.taccuser
   UP


.. note::

   Use :code:`tapis command subcommand --help` to find usage information for
   each command
