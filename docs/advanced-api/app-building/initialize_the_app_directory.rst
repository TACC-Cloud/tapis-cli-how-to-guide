Initialize the App Directory
============================

At the core of a Tapis app is an executable. Going in to the app building
process, it is generally assumed that the developer has an executable in mind
and the knowledge to run an instance of the executable with given inputs and /
or parameters. In this example, we will create an app for the
`FastQC <https://www.bioinformatics.babraham.ac.uk/projects/fastqc/>`_
tool. FastQC is a publicly-available quality control tool for raw next-gen
sequencing data.


Structure of an App
-------------------

To begin, run the command ``tapis apps init`` and give an arbitrary name for a
test app:

.. code-block:: bash

   $ tapis apps init test_app
   +-------+--------------------------------+
   | stage | message                        |
   +-------+--------------------------------+
   | setup | Project name: test_app         |
   | setup | Safened project name: test_app |
   | setup | Project path: ./test_app       |
   | clone | Project path: ./test_app       |
   +-------+--------------------------------+


This will create a new template app folder (in this case, called
``test_app/``) with the following form:

.. code-block:: bash

   $ tree test_app/
   test_app/
   ├── Dockerfile
   ├── app.json
   ├── assets
   │   ├── lib
   │   │   ├── VERSION
   │   │   └── container_exec.sh
   │   ├── runner.sh
   │   └── tester.sh
   ├── job.json
   └── project.ini


Several files and folders are created automatically. It is a good idea to take
some time now to look through the directory tree and examine the contents of
each file. A brief summary of the files are as follows:

* ``Dockerfile``: a Dockerfile for the app runtime
* ``app.json``: json file describing the app metadata, inputs, parameters, and outputs
* ``VERSION``: version file containing the image tag
* ``container_exec.sh``: utility script for executing a container on a TACC HPC system
* ``runner.sh``: main run script for the app; takes input and parameters from app.json
* ``tester.sh``: legacy script that may be used to run a local test
* ``job.json``: template for a job json file specific to this app
* ``project.ini``: initialization parameters for the app which are injected in to app.json



Initialize the FastQC App
-------------------------

Use the ``tapis apps init`` command again, but this time provide additional
flags to indicate the name and version of the app:

.. code-block:: bash

   $ tapis apps init --app-label fastqc --app-description "FastQC app" --app-version 0.11.9 fastqc_app
   +-------+----------------------------------+
   | stage | message                          |
   +-------+----------------------------------+
   | setup | Project name: fastqc_app         |
   | setup | Project description: FastQC app  |
   | setup | Project version: 0.11.9          |
   | setup | Safened project name: fastqc_app |
   | setup | Project path: ./fastqc_app       |
   | clone | Project path: ./fastqc_app       |
   +-------+----------------------------------+

   $ tree fastqc_app/
   fastqc_app/
   ├── Dockerfile
   ├── app.json
   ├── assets
   │   ├── lib
   │   │   ├── VERSION
   │   │   └── container_exec.sh
   │   ├── runner.sh
   │   └── tester.sh
   ├── job.json
   └── project.ini


From here on, we will refer to the location of this app bundle as
``~/fastqc_app/``. In the next sections, we will go through the template files
one by one to customize them for this particular app.



``~/fastqc_app/project.ini``
----------------------------

The first file to examine is called ``project.ini``, which contains
initialization parameters for the app. By default, the fields are populated by
some of the flags specified on the command line or picked up from the
environment:

.. code-block:: text

   [app]
   name = fastqc_app
   label = fastqc_app
   description = FastQC app
   version = 0.11.9
   ; bundle = assets
   ; deployment_path =
   deployment_system = tacc.work.taccuser
   execution_system = tacc.stampede2.taccuser

   [docker]
   dockerfile = Dockerfile
   namespace = taccuser
   repo = fastqc_app
   tag = 0.11.9

   [env]

   [git]
   branch = master
   ; remote =

   [grants]
   ; read =
   ; execute =
   ; update =

   [job]


The parameters listed above will be interpreted and injected into the app when
you deploy it. We need to make some changes to the data above. Set the
following:

.. code-block:: bash

   deployment_system = tacc.work.taccuser
   execution_system = tacc.stampede2.taccuser

These should be the names of your private storage and execution systems,
respectively.


``~/fastqc_app/app.json``
-------------------------

This is a templated app json file. By default, it will grab the app ``name``,
``version``, `executionSystem`, `deploymentSystem`, and other parameters from
your ``project.ini``. Now is a good time to modify this file if a typical job
run against this app would require, e.g., more than one node or a non standard
queue. The jinja2-formatted fields surrounded by double curly braces ``{{ }}``
are take from ``app.ini``.

Specific to FastQC, one input is required - a fastq file. Modify ``app.json``
to expect one input fastq file as shown below:

.. code-block:: json

   {
     "checkpointable": false,
     "name": "{{ username }}-{{ app.name }}",
     "executionSystem": "{{ app.execution_system }}",
     "executionType": "HPC",
     "deploymentPath": "{{ username }}/apps/{{ app.name }}-{{ app.version }}",
     "deploymentSystem": "{{ app.deployment_system }}",
     "helpURI": "",
     "label": "{{ app.label }}",
     "shortDescription": "{{ app.description }}",
     "longDescription": "",
     "modules": [
       "load tacc-singularity"
     ],
     "ontology": [],
     "parallelism": "SERIAL",
     "tags": [],
     "templatePath": "runner.sh",
     "testPath": "tester.sh",
     "version": "{{ app.version }}",
     "defaultMaxRunTime": "00:30:00",
     "inputs":[
       {
         "id": "fastq",
         "value": {
           "default": "",
           "visible": true,
           "required": true
         },
         "semantics": {
           "ontology": [
             "http://edamontology.org/format_1930"
           ]
         },
         "details": {
           "label": "FASTQ sequence file"
         }
       }
     ],
     "parameters": [
       {
         "id": "CONTAINER_IMAGE",
         "value": {
           "default": "{{ docker.namespace }}/{{ docker.repo }}:{{ docker.tag }}",
           "type": "string",
           "visible": false,
           "required": true,
           "order": 1000
         }
       }
     ],
     "outputs": []
   }


Please refer back to the previous
`App Documentation <../../api-essentials/find_an_application.html>`__
for a detailed breakdown of a typical app json file.


``~/fastqc_app/job.json``
-------------------------

The ``job.json`` file contains minimal information. The only change needed at
this time is to add the expect input:

.. code-block:: json

   {
     "name": "{{ app.name }}-test-{{ iso8601_basic_short }}",
     "appId": "{{ app.name }}-{{ app.version}}",
     "archive": false,
     "inputs": {
       "fastq": ""
     },
     "parameters": {}
   }


Next Steps
----------

If you have been following along, these files are ready to deploy for your app:

.. code-block:: text

   fastqc_app/
   ├── Dockerfile
   ├── app.json                      # Done
   ├── assets
   │   ├── lib
   │   │   ├── VERSION
   │   │   └── container_exec.sh     # Do not modify
   │   ├── runner.sh
   │   └── tester.sh                 # Do not modify
   ├── job.json                      # Done
   └── project.ini                   # Done


Next, we will build the ``Dockerfile`` and ``runner.sh``.
