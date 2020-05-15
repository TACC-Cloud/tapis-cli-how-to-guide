Find an Application
===================

Each Tapis tenant has its own set of public applications ("apps" for short) that
are  available for tenant-authenticated users to run. Public apps are also tied
to public execution systems.

List Available Applications
---------------------------

The ``tapis apps list`` command can be used to list all apps available to you.
The **tacc.prod** tenant has the following apps available:

.. code-block:: bash

   $ tapis apps list
   +-----------------------------------+------------------+
   | id                                | label            |
   +-----------------------------------+------------------+
   | tapis.app.imageclassify-1.0u3     | Image Classifier |
   | tapis.app.imageclassify-1.0u2     | Image Classifier |
   | tapis.app.imageclassify-1.0u1     | Image Classifier |
   | vina-ls5-1.1.2u3                  | Autodock Vina    |
   | vina-ls5-1.1.2u2                  | Autodock Vina    |
   | vina-ls5-1.1.2u1                  | Autodock Vina    |
   | opensees-2.4.4-slurm-2.4.4.5804u1 | OpenSees         |
   | opensees-fork-2.4.4.5804u2        | OpenSees         |
   | opensees-2.4.4.5804u1             | OpenSees         |
   | opensees-fork-2.4.4.5804u1        | OpenSees         |
   | vina-lonestar-1.1.2u4             | Autodock Vina    |
   | vina-lonestar-1.1.2u3             | Autodock Vina    |
   | vina-lonestar-1.1.2u2             | Autodock Vina    |
   | pdb2pdbqt-1.00u1                  | pdb2pdbqt        |
   +-----------------------------------+------------------+

Public apps will have a revision tag at the end (``u1``, ``u2``, ``u3`` etc.).
The higher the number, the newer the revision.

Search for Applications by Name
-------------------------------

Use ``tapis apps search`` to search for apps on a number of different criteria.
Some common searches might include:

.. code-block:: bash

   # Search for an app by part of its name
   $ tapis apps search --name like imageclassify

   # Search for apps that you own
   $ tapis apps search --owner eq taccuser

   # Search for apps that you don't own (public or shared with you)
   $ tapis apps search --owner neq taccuser



Display Application Information
-------------------------------

Many applications you will find in the catalog can be used in multiple ways. It
is up to the developer of an app to decide which function(s) of the particular
tool the app will perform, as well as what are the expected inputs, parameters,
and outputs. To see the description of an app use:

.. code-block:: bash

   $ tapis apps show tapis.app.imageclassify-1.0u3
   +--------------------------+------------------------------------------------------------------+
   | Field                    | Value                                                            |
   +--------------------------+------------------------------------------------------------------+
   | id                       | tapis.app.imageclassify-1.0u3                                    |
   | name                     | tapis.app.imageclassify                                          |
   | version                  | 1.0                                                              |
   | revision                 | 3                                                                |
   | label                    | Image Classifier                                                 |
   | lastModified             | 6 months ago                                                     |
   | shortDescription         | Classify an image using a small ImageNet model                   |
   | longDescription          |                                                                  |
   | owner                    | cicsvc                                                           |
   | isPublic                 | True                                                             |
   | executionType            | CLI                                                              |
   | executionSystem          | tapis.execution.system                                           |
   | deploymentSystem         | docking.storage                                                  |
   | available                | True                                                             |
   | parallelism              | SERIAL                                                           |
   | defaultProcessorsPerNode | 1                                                                |
   | defaultMemoryPerNode     | 1                                                                |
   | defaultNodeCount         | 1                                                                |
   | defaultMaxRunTime        | None                                                             |
   | defaultQueue             | None                                                             |
   | helpURI                  |                                                                  |
   | deploymentPath           | /home/docking/api/v2/prod/apps/tapis.app.imageclassify-1.0u3.zip |
   | templatePath             | wrapper.sh                                                       |
   | testPath                 | test/test.sh                                                     |
   | checkpointable           | False                                                            |
   | uuid                     | 3162334876895875561-242ac119-0001-005                            |
   | icon                     | None                                                             |
   +--------------------------+------------------------------------------------------------------+


The output of this command is a table-formatted description of the app including
select metadata. To see all of the app details including inputs, parameters, and
outputs, use the ``-f json`` flag to show json format:

.. code-block:: bash

   $ tapis apps show -f json tapis.app.imageclassify-1.0u3

.. code-block:: json

    {
      "id": "tapis.app.imageclassify-1.0u3",
      "name": "tapis.app.imageclassify",
      "version": "1.0",
      "revision": 3,
      "label": "Image Classifier",
      "lastModified": "6 months ago",
      "shortDescription": "Classify an image using a small ImageNet model",
      "longDescription": "",
      "owner": "cicsvc",
      "isPublic": true,
      "executionType": "CLI",
      "executionSystem": "tapis.execution.system",
      "deploymentSystem": "docking.storage",
      "available": true,
      "parallelism": "SERIAL",
      "defaultProcessorsPerNode": 1,
      "defaultMemoryPerNode": 1,
      "defaultNodeCount": 1,
      "defaultMaxRunTime": null,
      "defaultQueue": null,
      "tags": [
        "tensorflow",
        "ImageNet"
      ],
      "ontology": [],
      "helpURI": "",
      "deploymentPath": "/home/docking/api/v2/prod/apps/tapis.app.imageclassify-1.0u3.zip",
      "templatePath": "wrapper.sh",
      "testPath": "test/test.sh",
      "checkpointable": false,
      "modules": [
        "load tacc-singularity/2.6.0"
      ],
      "inputs": [],
      "parameters": [
        {
          "id": "imagefile",
          "value": {
            "visible": true,
            "required": true,
            "type": "string",
            "order": 0,
            "enquote": false,
            "default": "https://texassports.com/images/2015/10/16/bevo_1000.jpg",
            "validator": null
          },
          "details": {
            "label": "Image to classify",
            "description": "",
            "argument": "--image_file ",
            "showArgument": true,
            "repeatArgument": false
          },
          "semantics": {
            "minCardinality": 1,
            "maxCardinality": 1,
            "ontology": [
              "http://edamontology.org/format_3547"
            ]
          }
        },
        {
          "id": "predictions",
          "value": {
            "visible": true,
            "required": true,
            "type": "number",
            "order": 0,
            "enquote": false,
            "default": 5,
            "validator": null
          },
          "details": {
            "label": "Number of predictions to return",
            "description": null,
            "argument": "--num_top_predictions ",
            "showArgument": true,
            "repeatArgument": false
          },
          "semantics": {
            "minCardinality": 1,
            "maxCardinality": 1,
            "ontology": []
          }
        }
      ],
      "outputs": [],
      "uuid": "3162334876895875561-242ac119-0001-005",
      "icon": null,
      "_links": {
        "self": {
          "href": "https://api.tacc.utexas.edu/apps/v2/tapis.app.imageclassify-1.0u3"
        },
        "executionSystem": {
          "href": "https://api.tacc.utexas.edu/systems/v2/tapis.execution.system"
        },
        "storageSystem": {
          "href": "https://api.tacc.utexas.edu/systems/v2/docking.storage"
        },
        "history": {
          "href": "https://api.tacc.utexas.edu/apps/v2/tapis.app.imageclassify-1.0u3/history"
        },
        "metadata": {
          "href": "https://api.tacc.utexas.edu/meta/v2/data/?q=%7B%22associationIds%22%3A%223162334876895875561-242ac119-0001-005%22%7D"
        },
        "owner": {
          "href": "https://api.tacc.utexas.edu/profiles/v2/cicsvc"
        },
        "permissions": {
          "href": "https://api.tacc.utexas.edu/apps/v2/tapis.app.imageclassify-1.0u3/pems"
        }
      }
    }


Important Application Sections
------------------------------

**Metadata:** The metadata of the app json includes information about the app
availability, runtime resources required, description, and much more. Some key
information in the metadata section includes the identity of the HPC system
(``executionSystem``) on which the app runs. In the above case, it is
``tapis.execution.system``. Also, the ``shortDescription`` of the above app
suggests that the function is to classify an image using a small ImageNet model.

**Inputs:** The above app does not contain any inputs. This section is used to
describe required data and/or folders for running the app. Any files or folders
specified in the inputs section will be staged to the execution system prior to
running.

**Parameters:** This section describes important information, typically command
line options, for running the app. The above app requires two parameters - a URL
pointing to an image for the classifier, and the number of predictions that it
should return.

**Outputs:** The above app does not define any outputs. This section may be used
to specify expected output file or folder names, counts, and ontologies. While
this feature is still under development, it can be used to aid in chaining apps
together by providing the output of an app as input into a different app.


More information on each of these sections and understanding Tapis apps can be
found in the
`Tapis Documentation <https://tacc-cloud.readthedocs.io/projects/agave/en/latest/agave/guides/apps/introduction.html>`_.
