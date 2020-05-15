Deploy and Test the App
=======================

At this point, your app bundle directory structure should look similar to:

.. code-block:: text

   fastqc_app/
   ├── Dockerfile                   # Done
   ├── app.json                     # Done
   ├── assets
   │   ├── lib
   │   │   ├── VERSION              # Done
   │   │   └── container_exec.sh    # Do not modify
   │   ├── runner.sh                # Done
   │   └── tester.sh                # Do not modify
   ├── job.json                     # Done
   ├── project.ini                  # Done
   ├── src
   │   └── mycode.py                # Optional
   └── test
       └── SP1.fq                   # Optional



The final steps are to deploy the app in to the catalog on the tenant, then run
a test job to confirm it is working.

Upload Test Data
----------------

The Tapis app takes FASTQ data as input. Now is a good time to upload some
sample data to a storage system.

.. code-block:: bash

   # Make a directory for test data
   $ tapis files mkdir agave://tacc.work.taccuser/ test-data
   +--------------+---------------------------------------+
   | Field        | Value                                 |
   +--------------+---------------------------------------+
   | name         | test-data                             |
   | uuid         | 1105399583769563626-242ac112-0001-002 |
   | owner        | taccuser                              |
   | path         | /test-data                            |
   | lastModified | 2020-05-13T18:40:00.027-05:00         |
   | source       | None                                  |
   | status       | STAGING_COMPLETED                     |
   | nativeFormat | dir                                   |
   | systemId     | tacc.work.taccuser                    |
   +--------------+---------------------------------------+

   # Upload the SP1.fq file
   $ tapis files upload agave://tacc.work.taccuser/test-data/ ./test/SP1.fq
   +-------------------+----------+
   | Field             | Value    |
   +-------------------+----------+
   | uploaded          | 1        |
   | skipped           | 0        |
   | messages          | 0        |
   | bytes_transferred | 21.94 kB |
   | elapsed_sec       | 3        |
   +-------------------+----------+

   # Confirm the file exists on the Tapis storage system
   $ tapis files list agave://tacc.work.taccuser/test-data/
   +--------+--------------+--------+
   | name   | lastModified | length |
   +--------+--------------+--------+
   | SP1.fq | 1 minute ago |  22471 |
   +--------+--------------+--------+

The test data can be referenced in future calls to the app as:

.. code-block:: bash

   agave://tacc.work.taccuser/test-data/SP1.fq


Deploy the App
--------------

The ``tapis apps deploy`` command does multiple things. First, it builds and
pushes your docker image to the namespace specified in ``project.ini``. Second,
it uploads the app assets to a Tapis storage space specified by
``deploymentSystem`` and ``deploymentPath`` in the app json file. Finally, it
registers the app with the tenant.

Navigate in to the app bundle folder and issue:

.. code-block:: bash

   $ pwd
   ~/fastqc_app/

   $ ls
   Dockerfile      app.json        assets/         job.json
   project.ini     src/            test/

   $ tapis apps deploy -W ./
   +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | stage  | message                                                                                                                                                                                     |
   +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | build  | Step 1/4 : FROM tacc/tacc-ubuntu18-mvapich2.3-psm2                                                                                                                                          |
   | build  |  ---> d554e642ddc5                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Step 2/4 : RUN apt-get update     && apt-get upgrade -y     && apt-get install wget -y     && apt-get install zip -y     && apt-get install default-jre -y                                  |
   | build  |  ---> Using cache                                                                                                                                                                           |
   |        |                                                                                                                                                                                             |
   | build  |  ---> aa1f50856b62                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Step 3/4 : RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip     && unzip fastqc_v0.11.9.zip     && rm fastqc_v0.11.9.zip     && chmod +x FastQC/fastqc |
   | build  |  ---> Using cache                                                                                                                                                                           |
   |        |                                                                                                                                                                                             |
   | build  |  ---> 3bb6917b68d6                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Step 4/4 : ENV PATH "/FastQC/:$PATH"                                                                                                                                                        |
   | build  |  ---> Using cache                                                                                                                                                                           |
   |        |                                                                                                                                                                                             |
   | build  |  ---> 356927b0a8f6                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Successfully built 356927b0a8f6                                                                                                                                                             |
   |        |                                                                                                                                                                                             |
   | build  | Successfully tagged taccuser/fastqc_app:0.11.9                                                                                                                                              |
   |        |                                                                                                                                                                                             |
   | push   | The push refers to repository [docker.io/taccuser/fastqc_app]                                                                                                                               |
   | push   | 0.11.9: digest: sha256:29eb2fdb1503fdd38ae311dabfc13958f0910253580614dba0d3ac2dd0753e41 size: 4085                                                                                          |
   | upload | assets/runner.sh                                                                                                                                                                            |
   | upload | assets/tester.sh                                                                                                                                                                            |
   | upload | assets/lib/VERSION                                                                                                                                                                          |
   | upload | assets/lib/container_exec.sh                                                                                                                                                                |
   | create | Created Tapis app fastqc_app-0.11.9 revision 1                                                                                                                                              |
   +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


If all goes well, you should see a successful message at the end of the log
above, and you should see the new app listed in the apps catalog:

.. code-block:: bash

   $ tapis apps search --name like fastqc
   +----------------------------+----------+------------+------------------+----------+-------------------------+
   | id                         | revision | label      | shortDescription | isPublic | executionSystem         |
   +----------------------------+----------+------------+------------------+----------+-------------------------+
   | taccuser-fastqc_app-0.11.9 |        2 | fastqc_app | FastQC app       | False    | tacc.stampede2.taccuser |
   +----------------------------+----------+------------+------------------+----------+-------------------------+


Submit a Test Job
-----------------

Submitting a test job has been
`described previously <../../api-essentials/prepare_and_submit_a_job.html>`__
in this how-to guide. Here, testing will be performed in the same way. First,
create an appropriate ``job.json`` file.

.. code-block:: bash

   $ tapis jobs init --no-archive --output fastqc_job.json taccuser-fastqc_app-0.11.9


Which will output the following json, which can be streamed into a file for
submission (add the highlighted test data line):

.. code-block:: json
   :emphasize-lines: 11

   {
     "name": "taccuser-fastqc_app-job-1589377205989",
     "appId": "taccuser-fastqc_app-0.11.9",
     "batchQueue": "skx-normal",
     "maxRunTime": "01:00:00",
     "memoryPerNode": "1GB",
     "nodeCount": 1,
     "processorsPerNode": 1,
     "archive": false,
     "inputs": {
       "fastq": "agave://tacc.work.taccuser/test-data/SP1.fq"
     },
     "parameters": {},
     "notifications": [
       {
         "event": "FINISHED",
         "persistent": true,
         "url": "taccuser@gmail.com"
       },
       {
         "event": "FAILED",
         "persistent": true,
         "url": "taccuser@gmail.com"
       }
     ]
   }

Then, submit the test job:

.. code-block:: bash

   $ tapis jobs submit -F fastqc_job.json
   +--------+------------------------------------------+
   | Field  | Value                                    |
   +--------+------------------------------------------+
   | id     | 4e972f77-5bf9-446e-87a2-3541c4ea5745-007 |
   | name   | taccuser-fastqc_app-job-1589377205989    |
   | status | ACCEPTED                                 |
   +--------+------------------------------------------+


Finally, when the job status is **FINISHED**, inspect and retrieve the output:

.. code-block:: bash

   $ tapis jobs history 4e972f77-5bf9-446e-87a2-3541c4ea5745-007
   +-------------------+----------------+------------------------------------------------------------------------------+
   | status            | created        | description                                                                  |
   +-------------------+----------------+------------------------------------------------------------------------------+
   | PENDING           | 8 minutes ago  | Job processing beginning                                                     |
   | PROCESSING_INPUTS | 8 minutes ago  | Identifying input files for staging                                          |
   | STAGING_INPUTS    | 8 minutes ago  | Transferring job input data to execution system                              |
   | STAGING_INPUTS    | 8 minutes ago  | Job input copy in progress: agave://tacc.work.taccuser/public/SP1.fq to agav |
   |                   |                | e://tacc.stampede2.taccuser//scratch/05896/taccuser/taccuser/job-4e972f77-5b |
   |                   |                | f9-446e-87a2-3541c4ea5745-007-taccuser-fastqc_app-job-1589377205989/SP1.fq   |
   | STAGED            | 8 minutes ago  | Job inputs staged to execution system                                        |
   | STAGING_JOB       | 8 minutes ago  | Staging runtime assets to execution system                                   |
   | STAGING_JOB       | 8 minutes ago  | Fetching application assets from                                             |
   |                   |                | agave://tacc.work.taccuser/taccuser/apps/fastqc_app-0.11.9                   |
   | STAGING_JOB       | 8 minutes ago  | Staging runtime assets to agave://tacc.stampede2.taccuser//scratch/05896/sd2 |
   |                   |                | e0004/taccuser/job-4e972f77-5bf9-446e-87a2-3541c4ea5745-007-taccuser-fastqc_ |
   |                   |                | app-job-1589377205989                                                        |
   | SUBMITTING        | 8 minutes ago  | Submitting job to execution system                                           |
   | QUEUED            | 7 minutes ago  | Job queued to execution system queue                                         |
   | RUNNING           | 3 minutes ago  | Job running on execution system                                              |
   | CLEANING_UP       | 29 seconds ago | Job completed execution                                                      |
   | FINISHED          | 29 seconds ago | Job completed successfully                                                   |
   +-------------------+----------------+------------------------------------------------------------------------------+

   $ tapis jobs outputs list 4e972f77-5bf9-446e-87a2-3541c4ea5745-007
   +------------------------------------------------------------------------------------+-----------------+--------+
   | name                                                                               | lastModified    | length |
   +------------------------------------------------------------------------------------+-----------------+--------+
   | SP1.fq                                                                             | 21 minutes ago  |  22471 |
   | SP1_fastqc.html                                                                    | 17 minutes ago  | 561767 |
   | SP1_fastqc.zip                                                                     | 17 minutes ago  | 420233 |
   | container_exec.log                                                                 | 17 minutes ago  |  19232 |
   | lib                                                                                | 17 minutes ago  |   4096 |
   | runner.sh                                                                          | 17 minutes ago  |    875 |
   | taccuser-fastqc_app-job-1589377205989-4e972f77-5bf9-446e-87a2-3541c4ea5745-007.err | 21 minutes ago  |    372 |
   | taccuser-fastqc_app-job-1589377205989-4e972f77-5bf9-446e-87a2-3541c4ea5745-007.out | 21 minutes ago  |     29 |
   | taccuser-fastqc_app-job-1589377205989.ipcexe                                       | 21 minutes ago  |   2772 |
   | tester.sh                                                                          | 21 minutes ago  |     44 |
   +------------------------------------------------------------------------------------+-----------------+--------+

   $ tapis jobs outputs download 4e972f77-5bf9-446e-87a2-3541c4ea5745-007
   Walking remote resource...
   Found 13 file(s) in 5s
   Downloading .agave.archive...
   Downloading .agave.log...
   Downloading container_exec.log...
   Downloading container_exec.sh...
   Downloading VERSION...
   Downloading runner.sh...
   Downloading taccuser-fastqc_app-job-1589377205989-4e972f77-5bf9-446e-87a2-3541c4ea5745-007.err...
   Downloading taccuser-fastqc_app-job-1589377205989-4e972f77-5bf9-446e-87a2-3541c4ea5745-007.out...
   Downloading taccuser-fastqc_app-job-1589377205989.ipcexe...
   Downloading SP1.fq...
   Downloading SP1_fastqc.html...
   Downloading SP1_fastqc.zip...
   Downloading tester.sh...
   Downloaded 13 files in 61s
   +-------------+-------+
   | Field       | Value |
   +-------------+-------+
   | downloaded  | 13    |
   | skipped     | 0     |
   | messages    | 0     |
   | elapsed_sec | 66    |
   +-------------+-------+



If the file ``SP1_fastq.html`` exists, then the run was successful.
