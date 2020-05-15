Perform Basic File Operations
=============================

Data in the Tapis ecosystem can be managed using the ``files`` service. With
many parallels to Unix-style commands (``ls, mv, cp, rm,`` etc.), the
``tapis files -`` commands can be used to list files, upload and download data,
remotely manage and organize data, and import external data from the web.


List Files and Navigate the File Tree
-------------------------------------

To list the files available to you on a storage system, use:

.. code-block:: bash

   $ tapis files list agave://tacc.work.taccuser/
   +-----------+--------------+--------+
   | name      | lastModified | length |
   +-----------+--------------+--------+
   | jobs      | 2 years ago  |   4096 |
   | maverick  | 2 years ago  |   4096 |
   | stampede2 | 2 years ago  |   4096 |
   | wrangler  | 2 years ago  |   4096 |
   +-----------+--------------+--------+

The URI provided on the command line (``agave://tacc.work.taccuser/``) takes the
form:

1. ``agave://`` => refer to an Agave URI
2. ``tacc.work.taccuser`` => the name of the storage system
3. ``/`` => the relative path from the root directory on the storage system


To make a new folder, then list the contents of that folder:

.. code:: bash

  $ tapis files mkdir agave://tacc.work.taccuser/ test_folder
  +--------------+---------------------------------------+
  | Field        | Value                                 |
  +--------------+---------------------------------------+
  | name         | test_folder                           |
  | uuid         | 2668156827089366550-242ac112-0001-002 |
  | owner        | taccuser                              |
  | path         | /test_folder                          |
  | lastModified | 2020-05-12T07:48:19.141-05:00         |
  | source       | None                                  |
  | status       | STAGING_COMPLETED                     |
  | nativeFormat | dir                                   |
  | systemId     | tacc.work.taccuser                    |
  +--------------+---------------------------------------+

  $ tapis files list agave://tacc.work.taccuser/test_folder/
        # currently empty

To remove a folder, use the ``tapis files delete``:

.. code:: bash

   $ tapis files delete agave://tacc.work.taccuser/test_folder
   +--------------+-------+
   | Field        | Value |
   +--------------+-------+
   | deleted      | 1     |
   | skipped      | 0     |
   | warnings     | 0     |
   | elapsed_msec | 2318  |
   +--------------+-------+


.. warning::

   The ``tapis files delete`` command will delete folders with or without contents.



Upload and Download Files
-------------------------

Files can be transferred from your local machine to the remote storage system
using ``tapis files upload``, and from the remote storage system to your
local machine using the ``tapis files download``.

First, find or create a local file and upload it to the storage system (recreate
the ``test_folder/`` if you deleted it in the previous example):

.. code-block:: bash

   $ touch local_file.txt
   $ echo 'Hello, world!' > local_file.txt
   $ tapis files upload agave://tacc.work.taccuser/test_folder local_file.txt
   +-------------------+-------------+
   | Field             | Value       |
   +-------------------+-------------+
   | uploaded          | 1           |
   | skipped           | 0           |
   | messages          | 0           |
   | bytes_transferred | 14.00 bytes |
   | elapsed_sec       | 2           |
   +-------------------+-------------+

   $ tapis files list agave://tacc.work.taccuser/test_folder/
   +----------------+----------------+--------+
   | name           | lastModified   | length |
   +----------------+----------------+--------+
   | local_file.txt | 26 seconds ago |     14 |
   +----------------+----------------+--------+

Use ``tapis files copy`` to make a copy of the file on the remote system:

.. code-block:: bash

   $ tapis files copy agave://tacc.work.taccuser/test_folder/local_file.txt /test_folder/remote_copy.txt
   +--------------+--------------------------------------------------------------------------------------------------+
   | Field        | Value                                                                                            |
   +--------------+--------------------------------------------------------------------------------------------------+
   | name         | remote_copy.txt                                                                                  |
   | uuid         | 6484805032038306282-242ac112-0001-002                                                            |
   | owner        | taccuser                                                                                         |
   | path         | /test_folder/remote_copy.txt                                                                     |
   | lastModified | 2020-05-12T07:51:52.187-05:00                                                                    |
   | source       | https://api.tacc.utexas.edu/files/v2/media/system/tacc.work.taccuser//test_folder/local_file.txt |
   | status       | STAGING_COMPLETED                                                                                |
   | nativeFormat | raw                                                                                              |
   | systemId     | tacc.work.taccuser                                                                               |
   +--------------+--------------------------------------------------------------------------------------------------+

   $ tapis files list agave://tacc.work.taccuser/test_folder
   +-----------------+---------------+--------+
   | name            | lastModified  | length |
   +-----------------+---------------+--------+
   | local_file.txt  | 7 minutes ago |     14 |
   | remote_copy.txt | 3 minutes ago |     14 |
   +-----------------+---------------+--------+


Note that the second argument provided on the command line contains both the
name of the copied file, and the full path relative to the root directory for
the storage system.

To download the result:

.. code-block:: bash

   $ tapis files download agave://tacc.work.taccuser/test_folder/remote_copy.txt
   $ ls
   local_file.txt    remote_copy.txt
   $ cat remote_copy.txt
   Hello, world!


.. note::

   Use the ``-W`` flag to recursively download the contents of a whole directory



Other File Operations
---------------------

Using the Tapis CLI, files and folders can also be renamed, moved, and deleted
remotely on the storage system. The syntax for these operations is very similar
to the ``tapis files copy`` command syntax. Here are some common examples:

.. code-block:: bash

   # Rename a file in place
   $ tapis files move agave://tacc.work.taccuser/test_folder/remote_copy.txt /test_folder/renamed.txt

   # Make a subfolder in the test_folder/ folder
   $ tapis files mkdir agave://tacc.work.taccuser/test_folder/ subfolder

   # Rename a folder in place
   $ tapis files move agave://tacc.work.taccuser/test_folder/subfolder /test_folder/renamed_folder

   # Move a file into that subfolder
   $ tapis files move agave://tacc.work.taccuser/test_folder/renamed.txt /test_folder/renamed_folder/renamed.txt

   # Delete a file or a folder
   $ tapis files delete agave://tacc.work.taccuser/test_folder/renamed_folder


Be cautious with ``tapis files move`` and ``tapis files delete`` commands. Just
like a Linux filesystem, files inadvertently deleted or overwritten are most
likely unrecoverable.


File or Folder History
----------------------

You can list the history of events for a specific file or folder. This will give
more descriptive information (when applicable) related to number of retries,
permission grants and revocations, reasons for failure, and hiccups that may
have occurred in the transfer process.

.. code-block:: bash

   $ tapis files history agave://tacc.work.taccuser/test_folder/local_file.txt
   +-------------------+---------------+-------------------------------------------------------------------------------+
   | status            | created       | description                                                                   |
   +-------------------+---------------+-------------------------------------------------------------------------------+
   | STAGING_QUEUED    | 6 minutes ago | File/folder queued for staging                                                |
   | STAGING_COMPLETED | 6 minutes ago | Your scheduled transfer of http://129.114.97.130/local_file.txt completed     |
   |                   |               | staging. You can access the raw file on Storage system for TACC work          |
   |                   |               | directory at /work/01234/taccuser/test_folder/local_file.txt or via the API   |
   |                   |               | at https://api.tacc.utexas.edu/files/v2/media/system/tacc.work.taccuser//test |
   |                   |               | _folder/local_file.txt.                                                       |
   | DOWNLOAD          | 4 minutes ago | File was downloaded                                                           |
   +-------------------+---------------+-------------------------------------------------------------------------------+



Further Help
------------

Reminder: at any time, you can issue a Tapis CLI command with the ``-h`` flag to
find more information on the function and usage of the command. Extensive Tapis
CLI documentation can be found
`HERE <https://tapis-cli.readthedocs.io/en/latest/>`_.
