Prepare and Submit a Job
========================

Continuing with the previous example of the Image Classifier app (see
`Find an Application <find_an_application.html>`__), we know there are two
parameters we need to specify: a URL pointing to an image for the classifier,
and the number of predictions the app should return.

Build a Job Template File
-------------------------

To run an instance of this application (called a "job"), we first must assemble
a json description of the job we would like to run. The simplest way to do this
is to use the ``tapis jobs init`` command:

.. code-block:: bash

   $ tapis jobs init tapis.app.imageclassify-1.0u3

.. code-block:: json

    {
      "name": "tapis.app.imageclassify-job-1585230186004",
      "appId": "tapis.app.imageclassify-1.0u3",
      "batchQueue": "normal",
      "maxRunTime": "01:00:00",
      "memoryPerNode": "1GB",
      "nodeCount": 1,
      "processorsPerNode": 1,
      "archive": true,
      "inputs": {},
      "parameters": {
        "imagefile": "https://texassports.com/images/2015/10/16/bevo_1000.jpg",
        "predictions": 5
      },
      "notifications": [
        {
          "event": "*",
          "persistent": true,
          "url": "taccuser@gmail.com"
        }
      ]

The only option given to the command is the name of the app. The output is a
json template for submitting a job against the app. Metadata include a ``name``
for the job (which can be changed), the ``appId``, and other runtime options.
There are also the two parameters we expect filled with default options.

The only change we want to set now is ``"archive": false`` (see below for
details on archiving jobs), and we also need to redirect this json into a file
in order to submit a job. Execute:

.. code-block:: bash

   $ tapis jobs init --no-archive --output job.json tapis.app.imageclassify-1.0u3


.. tip::

   Use ``tapis jobs init --help`` to see further options


Submit a Job
------------

Once you are satisfied that job.json contains the desired content, use the
``tapis jobs submit`` command to run an instance of this job:


.. code-block:: bash

   $ tapis jobs submit -F job.json
   +--------+-------------------------------------------+
   | Field  | Value                                     |
   +--------+-------------------------------------------+
   | id     | f0cb69a1-63a4-4970-9921-843968e66723-007  |
   | name   | tapis.app.imageclassify-job-1589290511905 |
   | status | ACCEPTED                                  |
   +--------+-------------------------------------------+

An ACCEPTED status indicates the job.json was valid, and e-mail alerts (if they
were specified in job.json) will track the progress of the job. Also take note
of the long hexadecimal ``id`` when you submit the job. This identifier can be
used to track progress and download results.

.. note::

   If you are receiving too many e-mail alerts, try changing the job.json to
   only send an alert on ``"event": "FINISHED"``


Track a Job
-----------

The Tapis CLI offers several commands to find and track the progress of jobs. If
you know the specific job ID you are looking for, that can usually be passed to
one of the jobs commands. Otherwise, you can list all jobs and find what you are
looking for based on the name of the job and how recently it was run:

.. code-block:: bash

   # List all jobs you have run
   $ tapis jobs list
   +------------------------------------------+-------------------------------------------+----------+
   | id                                       | name                                      | status   |
   +------------------------------------------+-------------------------------------------+----------+
   | f0cb69a1-63a4-4970-9921-843968e66723-007 | tapis.app.imageclassify-job-1589290511905 | FINISHED |
   +------------------------------------------+-------------------------------------------+----------+

   # List the status of a specific job
   $ tapis jobs status f0cb69a1-63a4-4970-9921-843968e66723-007
   +--------+-------------------------------------------+
   | Field  | Value                                     |
   +--------+-------------------------------------------+
   | id     | f0cb69a1-63a4-4970-9921-843968e66723-007  |
   | name   | tapis.app.imageclassify-job-1589290511905 |
   | status | FINISHED                                  |
   +--------+-------------------------------------------+

   # List the history of a specific job
   $ tapis jobs history f0cb69a1-63a4-4970-9921-843968e66723-007
   +-------------------+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | status            | created       | description                                                                                                                                                                 |
   +-------------------+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | PENDING           | 3 minutes ago | Job processing beginning                                                                                                                                                    |
   | PROCESSING_INPUTS | 3 minutes ago | Identifying input files for staging                                                                                                                                         |
   | STAGED            | 3 minutes ago | Job inputs staged to execution system                                                                                                                                       |
   | STAGING_JOB       | 3 minutes ago | Staging runtime assets to execution system                                                                                                                                  |
   | STAGING_JOB       | 3 minutes ago | Fetching application assets from agave://docking.storage//home/docking/api/v2/prod/apps/tapis.app.imageclassify-1.0u3.zip                                                   |
   | STAGING_JOB       | 3 minutes ago | Staging runtime assets to agave://tapis.execution.system//home/demo/scratch/taccuser/job-f0cb69a1-63a4-4970-9921-843968e66723-007-tapis-app-imageclassify-job-1589290511905 |
   | SUBMITTING        | 2 minutes ago | Submitting job to execution system                                                                                                                                          |
   | QUEUED            | 2 minutes ago | Job queued to execution system queue                                                                                                                                        |
   | RUNNING           | 2 minutes ago | Job running on execution system                                                                                                                                             |
   | CLEANING_UP       | a minute ago  | Job completed execution                                                                                                                                                     |
   | FINISHED          | a minute ago  | Job completed successfully                                                                                                                                                  |
   +-------------------+---------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


Download the Results
--------------------

Once the job status is **FINISHED**, you can list what output is available:

.. code-block:: bash

   $ tapis jobs outputs list f0cb69a1-63a4-4970-9921-843968e66723-007
   +--------------------------------------------------+---------------+-----------+
   | name                                             | lastModified  |    length |
   +--------------------------------------------------+---------------+-----------+
   | classifier_img.sif                               | 3 minutes ago | 379068416 |
   | image.jpg                                        | 3 minutes ago |    116625 |
   | predictions.txt                                  | 2 minutes ago |    498600 |
   | tapis-app-imageclassify-job-1589290511905.err    | 2 minutes ago |       866 |
   | tapis-app-imageclassify-job-1589290511905.ipcexe | 3 minutes ago |      2353 |
   | tapis-app-imageclassify-job-1589290511905.out    | 3 minutes ago |         0 |
   | tapis-app-imageclassify-job-1589290511905.pid    | 3 minutes ago |         6 |
   | test                                             | 3 minutes ago |        21 |
   | wrapper.sh                                       | 3 minutes ago |       196 |
   +--------------------------------------------------+---------------+-----------+

For this app, there are several assets available to download. The important
output is the predictions.txt file. You can choose to download all of the assets
as a bundle, or a single file:

.. code-block:: bash

   # Download a single file
   $ tapis jobs outputs download f0cb69a1-63a4-4970-9921-843968e66723-007 predictions.txt
   +-------------+-------+
   | Field       | Value |
   +-------------+-------+
   | downloaded  | 1     |
   | skipped     | 0     |
   | messages    | 0     |
   | elapsed_sec | 3     |
   +-------------+-------+

    # Download all outputs
    $ tapis jobs outputs download --progress f0cb69a1-63a4-4970-9921-843968e66723-007
    Walking remote resource...
    Found 11 file(s) in 2s
    Downloading .agave.archive...
    Downloading .agave.log...
    Downloading classifier_img.sif...
    Downloading image.jpg...
    Downloading predictions.txt...
    Downloading tapis-app-imageclassify-job-1589290511905.err...
    Downloading tapis-app-imageclassify-job-1589290511905.ipcexe...
    Downloading tapis-app-imageclassify-job-1589290511905.out...
    Downloading tapis-app-imageclassify-job-1589290511905.pid...
    Downloading test.sh...
    Downloading wrapper.sh...
    Downloaded 11 files in 176s
    +-------------+-------+
    | Field       | Value |
    +-------------+-------+
    | downloaded  | 11    |
    | skipped     | 0     |
    | messages    | 0     |
    | elapsed_sec | 178   |
    +-------------+-------+


.. note::

   The ``--progress`` flag prints progress of the download to STDOUT


Job Archives
------------

Job archiving can either be set to true or false. If false, then the job outputs
will remain in the execution folder created for your job. This may be on a
scratch file system with a purge policy, so the outputs may be available for
only a limited time through the jobs service. This can be useful when a job
generates intermediate files that are not needed with the final output. To set
archive mode to false, the following line should appear in your job.json file:

.. code-block::

   "archive": false,


If job archiving is set to true, then the job outputs are written to a storage
system and path that you specify, and will always be available through the jobs
service. Use the following lines in your job.json file:

.. code-block::

   "archive": true,
   "archiveSystem": "tacc.work.taccuser",
   "archivePath": "/path/you/specify"

If you omit the name of the archive path, it will choose a default path on your
storage system with the job identifier in the path name (recommended).
