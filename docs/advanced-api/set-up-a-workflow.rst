Set up a Workflow
=================

This guide will demonstrate a few common ways users can leverage the Tapis CLI
for complex workflows.

Chain Multiple Apps
-------------------

A useful feature of Tapis is the ability to use the output of one job as the
input of a second job. It saves time and avoids unnecessary file transfers by
keeping the input / output within Tapis "job API space". For example, consider
the following two hypothetical apps:

+---------------------+----------------+----------------+
| **App Name**        | **Input**      | **Output**     |
+---------------------+----------------+----------------+
| ImageClassifier-1.0 | image.jpg      | *report.txt*\* |
+---------------------+----------------+----------------+
| FileZipper-1.0      | *report.txt*\* | report.txt.zip |
+---------------------+----------------+----------------+

In these examples, the first app, ImageClassifier-1.0 takes an image file as
input, and produces an output report.txt file. The second app, FileZipper-1.0,
takes the output report.txt and compresses it to report.txt.zip.

First, submit a job against the first app:

.. code-block:: bash

   # Assemble job1.json file
   $ cat job1.json
   {
     "name": "Image Classifier Demo Job",
     "appId": "ImageClassifier-1.0",
     "archive": false,
     "inputs": {
       "image": "agave://tacc.work.taccuser/test-data/image.jpg"
     },
     "parameter": {}
   }

   # Submit the job
   $ tapis jobs submit -F job1.json
   +--------+-------------------------------------------+
   | Field  | Value                                     |
   +--------+-------------------------------------------+
   | id     | f0cb69a1-63a4-4970-9921-843968e66723-007  |
   | name   | Image Classifier Demo Job                 |
   | status | ACCEPTED                                  |
   +--------+-------------------------------------------+

   # When it is done, list the outputs
   $ tapis jobs outputs list f0cb69a1-63a4-4970-9921-843968e66723-007
   +------------+---------------+---------+
   | name       | lastModified  |  length |
   +------------+---------------+---------+
   | ...        |               |         |
   | report.txt | 2 minutes ago |  116625 |
   | ...        |               |         |
   +------------+---------------+---------+


Now that the first job is done, prepare and submit the second job referencing
the output from the first job. The URI pointing to the report.txt file takes a
very specific form:

.. code-block:: bash

   # Assemble job2.json file
   $ cat job2.json
   {
     "name": "File Zipper Demo Job",
     "appId": "FileZipper-1.0",
     "archive": false,
     "inputs": {
       "file": "https://api.tacc.utexas.edu/jobs/v2/f0cb69a1-63a4-4970-9921-843968e66723-007/outputs/media/report.txt"
     },
     "parameter": {}
   }

   # Submit the job
   $ tapis jobs submit -F job2.json
   +--------+-------------------------------------------+
   | Field  | Value                                     |
   +--------+-------------------------------------------+
   | id     | 3fea4b88-424a-4c25-b0ef-6e6908eed843-007  |
   | name   | File Zipper Demo Job                      |
   | status | ACCEPTED                                  |
   +--------+-------------------------------------------+

   # When it is done, list the outputs
   $ tapis jobs outputs list 3fea4b88-424a-4c25-b0ef-6e6908eed843-007
   +----------------+---------------+--------+
   | name           | lastModified  | length |
   +----------------+---------------+--------+
   | ...            |               |        |
   | report.txt.zip | 2 minutes ago |   2906 |
   | ...            |               |        |
   +----------------+---------------+--------+


If everything worked correctly, you should now see the final zipped file when
listing the job outputs. As long as you remain on the **tacc.prod** tenant, much
of the URI to the temporary file will remain the same:

.. code-block:: bash

   https://api.tacc.utexas.edu/      # base URL - DO NOT CHANGE
   jobs/v2/                          # refers to jobs API - DO NOT CHANGE
   f0cb69a1-63a4-4970-9921-.../      # job ID from step 1
   outputs/media/                    # location of output data - DO NOT CHANGE
   report.txt                        # name of output file from step 1


Finally, the zipped report can be downloaded as:

.. code-block:: bash

   $ tapis jobs outputs download 3fea4b88-424a-4c25-b0ef-6e6908eed843-007 report.txt.zip
   +-------------+-------+
   | Field       | Value |
   +-------------+-------+
   | downloaded  | 1     |
   | skipped     | 0     |
   | messages    | 0     |
   | elapsed_sec | 8     |
   +-------------+-------+






Parameter Sweeps
----------------

Many public apps are designed to take one input file or configuration, run an
analysis, and return a result. With some simple scripting, it is possible to
perform parameter sweeps using multiple Tapis jobs. For example, imagine you
would like to run FastQC on a series of FASTQ files named: ``fastq_01.fq``,
``fastq_02.fq``, ``fastq_03.fq``, etc.:

.. code-block:: bash

   # Organize the data in a local folder:
   $ ls fastq_data/
   fastq_01.fq  fastq_02.fq  fastq_03.fq

   # Upload all fastqc files to a storage system
   $ tapis files upload agave://tacc.work.taccuser/test-data/ fastq_data
   +-------------------+----------+
   | Field             | Value    |
   +-------------------+----------+
   | uploaded          | 3        |
   | skipped           | 0        |
   | messages          | 0        |
   | bytes_transferred | 65.83 kB |
   | elapsed_sec       | 8        |
   +-------------------+----------+

   # Create a template json file:
   $  tapis jobs init --no-archive --no-notify taccuser-fastqc_app-0.11.9 > job_template.json
   $ cat job_template.json
   {
     "name": "taccuser-fastqc_app-job-1589474193147",
     "appId": "taccuser-fastqc_app-0.11.9",
     "archive": false,
     "inputs": {
       "fastq": "agave://tacc.work.taccuser/public/SP1.fq"
     },
     "parameters": {}
   }


The next  step is to write a short script around the ``job_template.json``
template that populates the file name into a new job json file, then submits the
ob. Here is an example script:

.. code-block:: bash

   #!/bin/bash

   for FILE in ` ls fastq_data/ `
   do

   cat <<EOF >fastqc.json
   {
     "name": "FastQC $FILE",
     "appId": "taccuser-fastqc_app-0.11.9",
     "archive": false,
     "inputs": {
       "fastq": "agave://tacc.work.taccuser/test-data/fastq_data/$FILE"
     },
     "parameters": {}
   }
   EOF

   tapis jobs submit -F fastqc.json

   done

Finally, execute the script:

.. code-block:: bash

   $ bash parameter_sweep.sh
   +--------+------------------------------------------+
   | Field  | Value                                    |
   +--------+------------------------------------------+
   | id     | eef0030f-77d6-4a98-8893-a385137c3b44-007 |
   | name   | FastQC fastq_01.fq                       |
   | status | ACCEPTED                                 |
   +--------+------------------------------------------+
   +--------+------------------------------------------+
   | Field  | Value                                    |
   +--------+------------------------------------------+
   | id     | 130a2099-3ae5-4f2c-ab2d-840a642cb2a9-007 |
   | name   | FastQC fastq_02.fq                       |
   | status | ACCEPTED                                 |
   +--------+------------------------------------------+
   +--------+------------------------------------------+
   | Field  | Value                                    |
   +--------+------------------------------------------+
   | id     | 73ace158-7195-463f-b436-c4a519f7ba83-007 |
   | name   | FastQC fastq_03.fq                       |
   | status | ACCEPTED                                 |
   +--------+------------------------------------------+

   $ tapis jobs list --limit 3
   +------------------------------------------+--------------------+--------+
   | id                                       | name               | status |
   +------------------------------------------+--------------------+--------+
   | 73ace158-7195-463f-b436-c4a519f7ba83-007 | FastQC fastq_03.fq | QUEUED |
   | 130a2099-3ae5-4f2c-ab2d-840a642cb2a9-007 | FastQC fastq_02.fq | QUEUED |
   | eef0030f-77d6-4a98-8893-a385137c3b44-007 | FastQC fastq_01.fq | QUEUED |
   +------------------------------------------+--------------------+--------+


This is a simple example of a control script with plenty of room for advanced
features and error checking.
