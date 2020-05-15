Create a Private System
=======================

Many tenants automatically provide their users with private storage and
execution systems connecting to TACC resources, but some do not. This guide
walks through the process of creating your own private systems. This can also
be used to help you connect to non-TACC lab servers, cloud VMs, and clusters.


Gather Relevant Information
---------------------------

To register any system in Tapis, you need the hostname and login credentials.
The preferred login credentials are username and SSH key pairs. Storage systems
(for storing files) require a default path where you have write access.
Execution systems (for running jobs) also require a default path where job
runtime files will be staged and the job will be executed. If it is an 'HPC'
type execution system, then you also need information about the queueing system
(queue names, limits, etc.).

For this demonstration, we will set up a storage system to access the TACC work
directory (via Stampede2):

.. code-block:: bash

    hostname: stampede2.tacc.utexas.edu
    username: taccuser
    credentials: <ssh keys>
    storage path: /work/01234/taccuser


And we will set up an execution system for the Stampede2 HPC cluster:

.. code-block:: bash

   hostname: stampede2.tacc.utexas.edu
   username: taccuser
   credentials: <ssh keys preferred>
   storage path: /work/01234/taccuser
   job runtime path: /scratch/01234/taccuser
   queue_type: SLURM
   queue: normal (limits in Stampede2 user guide)


.. note::

   This guide assumes you have the appropriate permissions and credentials to
   access Stampede2. These can be attained by having an active
   `Stampede2 Allocation <https://portal.tacc.utexas.edu/allocations-overview>`_


Register a Storage System
-------------------------

To register a system, you need to assemble a json description of the above
requirements and some additional metadata. Start by saving the following storage
system template in a file called ``tacc.work.taccuser.json``:

.. code-block:: json

   {
     "id": "tacc.work.taccuser",
     "name": "Storage system for the TACC WORK directory",
     "description": "Storage system for the TACC WORK directory via Stampede2",
     "type": "STORAGE",
     "storage": {
       "host": "stampede2.tacc.utexas.edu",
       "port": 22,
       "protocol": "SFTP",
       "rootDir": "/work/01234/taccuser",
       "homeDir": "/",
       "auth":{
         "username":"taccuser",
         "publicKey": " <enter public key here> ",
         "privateKey": " <enter private key here> ",
         "type": "SSHKEYS"
       }
     }
   }

Most fields are fairly self explanatory, but here is a brief breakdown of the
important options:

  * ``id``: a unique identifier and how the system name appears for ``tapis systems list`` commands
  * ``name``: common display name for the system
  * ``description``: optional long plain text description of the system
  * ``type``: can be ``STORAGE`` or ``EXECUTION``
  * ``host``: IP address or hostname of system
  * ``rootDir``: path of the virtual root directory on the remote system
  * ``homeDir``: path relative to ``rootDir`` for ``tapis files -`` operations
  * ``username``: this is your username for the target system
  * ``publicKey``: cut and paste your public key here
  * ``privateKey``: cut and paste your private key here


Edit the username and paths in the above template to match your username and
work folder. A copy of your public key should be added to the
:code:`~/.ssh/authorized_keys` file on the remote host. The public and private
key should be pasted on one line each similar to the following. Replace the
line breaks in the private key with the newline character :code:`\n`:

.. code-block:: json

   {
     "auth":{
       "username": "taccuser",
       "publicKey": "ssh-rsa AAAAB3NzaC1yc2EBBAADAQABMQRgQqSuJdTi+VwMif8qouSSEWVduKZHpzOnS1zlknAyYXmQQFcaJ+vNAQayVMTqv+A+1lzxppTdgZ0Dn42EOYWRa6B/IEMPzDuKb7F0qNFiH9m+OZJDYdIWS1rlN1oK32jHUi0xV8kM3KOLf2TIjDBUyZRpMGyQ= user@email.com",
       "privateKey": "-----BEGIN RSA PRIVATE KEY-----\nMIIEpAIBAAKCAQEA1Jhi5BNiogg3NtALJepyTz5xS3j/dpYBGf5ERBH0C\n4SCb9VAxOCyb4l+QDrOQnLRX2RV4JjHlw7r8qmc6IvPmk83oTYqYN2NuzMjxI\nsqjVfmJgnF4sPuQy+Pioie9UeekAJRfaJLChZxLfyfppUVNTOOg6rVkERV/n9IDr\nTY2r/B16XtzcjGYvhW35Avy2FlTHvJldwaxmY4UuNey7r9LXAved4nqTj7d\n5PVKgWB8Bu6h5U1EGgnPhFFi8MJCO4/bByqAYdEffC9Y+cWBFq749XNPafid\nDlKFza44RR5Fg86OZxJW7NGoMnIjVYRIcUQIDAQABAoIBAQCovogIBscMW6R/\nfTwM/h3OlUu9EdlVOfygwkq5GfdbPBco291UOmDwN08aryTR8JtVLPO5ZtevX\nTVXVpWtejdLr5aL/R/uYxhxaIoeI7ppQBk3daSNsZia2lRp1j4qil\nyKfy5WxHdzjAhg3gamYtTk981qJIOSR0kQxxz3ax23BN5C/r1uqHK1hFUlCgx\nRrjt2M2/TvFtGZdRmxH4Kdco7IeOtj0xAYS/hGBV4CRa+4zWb3ikNOVxcFL61\nuT/60043DsVI22B5zv3XODtfSjquqlYl5eHZdf+HdL6u91CKrjmvpg80OfQ\ngmPwhOjdAoGBAOtRhXta/Y8X1U+XykaXfVFsfzFsslMtI73XII+nKYdtDFlSl\nLYg6PB5Gk9Q++RdinHzL7DwAXOVWW2nwfoEKxjsYCw4ihYVO/UEG\nqqCeu0X/r9N6u78HfeZEX5XH3+QtR/d9bP2mLjhY8LTAoGBAOdH\nnHnrMpeiEzou+5UC3lKRUN84LX/o9kp6t6WSF5oT7tQEyJKVICgLBOMVbASvXZxncYziYJKIzrqDkC+QXdZpF0x/u04vryDz9ySl9rhBYaD74e+FFXkDImMAQ\nCL1InIelCmXcWsORJd+5yCGOSS3TL2lA+1YXLAoGf47hMm/uT0HvzVhDq7\nD+764ZgRHjN8tpn9N0hz/Gj0zaw+9lOXEXG1DnlGzo016sAOc+2tFZx\n3j8w9cZQJ0zTE2u7Lz8CL9yKXicsOgFhdeyrF4AwtJ2CLtZF383wim2QFi4/Ypkl\nL4lsYnJYnJjQCKgA6bROu0+rA1TUvCzXHbgH7t6eYRcZeKnJNZ+m3PhBs+8W\nov4nLLTz8Q7GN8g6T1//QojS8y   ZR9GAr0Z0BbtW8om+fVehPFAMm8x6tS4sTFl0\nUp+i0r4VF7PnvfSIC+AHJUe+a4XPmmphVsnxEpsS+tQ2yUh7Akmt\np8WOECgYAJuaT+FBqIWhvmaymOjUFfQug67+lv7w3qzzWQAq8DyTweFNJ4E\nIbE1RnT86V2xhPr3YgjmRyyONlb/Xr8fZrz8KpmSehT99a+QY6gkIoWrfQ5xS7g6\nI/GDX2x54eANWX0xXKMQXfTU+WN6s5WPl/BL+/Cj43Hfg==\n-----END RSA PRIVATE KEY-----",
       "type": "SSHKEYS"
     }
   }

.. warning::

   Remember, never share this json file because it contains a plain text copy of
   your private key.

You will need to keep a copy of this file to edit the storage system in the
future. To register this system with Tapis, use the following command:

.. code-block:: bash

   $ tapis systems create -F tacc.work.taccuser.json
   +----------------------+------------------------------------------------------+
   | Field                | Value                                                |
   +----------------------+------------------------------------------------------+
   | id                   | tacc.work.taccuser                                   |
   | name                 | Storage system for TACC work directory               |
   | type                 | STORAGE                                              |
   | default              | False                                                |
   | available            | True                                                 |
   | description          | Storage system for TACC work directory via Stampede2 |
   | executionType        | None                                                 |
   | globalDefault        | False                                                |
   | lastModified         | just now                                             |
   | maxSystemJobs        | None                                                 |
   | maxSystemJobsPerUser | None                                                 |
   | owner                | taccuser                                             |
   | public               | False                                                |
   | revision             | 1                                                    |
   | scheduler            | None                                                 |
   | scratchDir           | None                                                 |
   | site                 | None                                                 |
   | status               | UP                                                   |
   | uuid                 | 383424038079107562-242ac112-0001-006                 |
   | workDir              | None                                                 |
   +----------------------+------------------------------------------------------+


Confirm that it worked by searching for the storage system and listing files
in the root directory:


.. code-block:: bash

   $ tapis systems search --id eq tacc.work.taccuser
   +--------------------+----------------------------------------+---------+---------+
   | id                 | name                                   | type    | default |
   +--------------------+----------------------------------------+---------+---------+
   | tacc.work.taccuser | Storage system for TACC work directory | STORAGE | False   |
   +--------------------+----------------------------------------+---------+---------+

   $ tapis files list agave://tacc.work.taccuser/
   +-----------+--------------+--------+
   | name      | lastModified | length |
   +-----------+--------------+--------+
   | jobs      | 2 years ago  |   4096 |
   | maverick  | 2 years ago  |   4096 |
   | stampede2 | 2 years ago  |   4096 |
   | wrangler  | 2 years ago  |   4096 |
   +-----------+--------------+--------+


Register an Execution System
----------------------------

An execution system contains many of the same fields as a storage system, but it
is a bit more involved. Save the following template for a Stampede2 execution
system into a file called ``tacc.stampede2.taccuser``:


.. code-block:: json

   {
     "id": "tacc.stampede2.taccuser",
     "name": "Execution system for TACC Stampede2",
     "description": "Execution system for TACC Stampede2",
     "type": "EXECUTION",
     "executionType": "HPC",
     "scheduler": "SLURM",
     "maxSystemJobsPerUser": 50,
     "scratchDir": "/scratch/01234/taccuser",
     "login": {
       "host": "stampede2.tacc.utexas.edu",
       "port": 22,
       "protocol": "SSH",
       "auth": {
         "username": "taccuser",
         "publicKey": " <enter public key here> ",
         "privateKey": " <enter private key here> ",
         "type": "SSHKEYS"
       }
     },
     "storage": {
       "host": "stampede2.tacc.utexas.edu",
       "port": 22,
       "protocol": "SFTP",
       "rootDir": "/",
       "homeDir": "/work/01234/taccuser",
       "auth": {
         "username": "taccuser",
         "publicKey": " <enter public key here> ",
         "privateKey": " <enter private key here> ",
         "type": "SSHKEYS"
       }
     },
     "queues": [
       {
         "name": "normal",
         "maxProcessorsPerNode": 68,
         "maxMemoryPerNode": "96GB",
         "maxNodes": 256,
         "maxRequestedTime": "48:00:00",
         "customDirectives": "-A <enter allocation name here>",
         "default": true
       }
     ]
   }

Some of the new or changed fields in this execution system include:

  * ``type``: execution system rather than storage system
  * ``executionType``: ``HPC`` indicates a cluster with a job scheduler
  * ``scheduler``: Stampede2 uses a SLURM scheduler
  * ``maxSystemJobsPerUser``: maximum concurrent jobs on the system per user
  * ``scratchDir``: path for job working directory at runtime, relative to ``rootDir``
  * ``login``: similar to `storage`, host and credential information
  * ``queues``: an array of batch queue definitions for the HPC system

For this execution system, there are two locations to cut and paste your SSH
keys. Again, because keys will be stored in plain text in this file, do not
share this file with anyone and keep it secure. In addition, the ``queues``
parameter has an option called ``customDirectives`` which should contain the
name of an allocation you have access to on Stampede2. And finally, as before,
make sure to change the username and paths to match your account on the HPC
system.

Once the appropriate changes have been made to the json file, register the
system with Tapis using the following command:

.. code-block:: bash

   $ tapis systems create -F tacc.stampede2.taccuser.json
   +----------------------+---------------------------------------+
   | Field                | Value                                 |
   +----------------------+---------------------------------------+
   | id                   | tacc.stampede2.taccuser               |
   | name                 | Execution system for TACC Stampede2   |
   | type                 | EXECUTION                             |
   | default              | False                                 |
   | available            | True                                  |
   | description          | Execution system for TACC Stampede2   |
   | executionType        | HPC                                   |
   | globalDefault        | False                                 |
   | lastModified         | just now                              |
   | maxSystemJobs        | 2147483647                            |
   | maxSystemJobsPerUser | 50                                    |
   | owner                | taccuser                              |
   | public               | False                                 |
   | revision             | 1                                     |
   | scheduler            | SLURM                                 |
   | scratchDir           | /scratch/01234/taccuser/              |
   | site                 | None                                  |
   | status               | UP                                    |
   | uuid                 | 4903282542648684054-242ac112-0001-006 |
   | workDir              |                                       |
   +----------------------+---------------------------------------+


Finally, confirm that the system exists by searching for it then listing the
available queues:

.. code-block:: bash

   # Search for your private systems
   $ tapis systems search --public eq false
   +-------------------------+----------------------------------------+-----------+---------+
   | id                      | name                                   | type      | default |
   +-------------------------+----------------------------------------+-----------+---------+
   | tacc.stampede2.taccuser | Execution system for TACC Stampede2    | EXECUTION | False   |
   | tacc.work.taccuser      | Storage system for TACC work directory | STORAGE   | False   |
   +-------------------------+----------------------------------------+-----------+---------+

   # List queues on the execution system
   $ tapis systems queues list -f json tacc.stampede2.taccuser
   [
     {
       "name": "normal",
       "description": null,
       "default": true,
       "maxUserJobs": -1,
       "maxRequestedTime": "48:00:00"
     }
   ]



Additional Help
---------------

Further information about creating storage and execution systems, including full
descriptions of the parameters above as well as other optional parameters, can
be found in the
`Tapis platform documentation <https://tacc-cloud.readthedocs.io/en/latest/>`_
