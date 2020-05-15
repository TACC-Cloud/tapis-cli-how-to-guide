Modify an Existing App
======================

Modifying an existing app would come in handy if there is a need to change the,
e.g., execution system, description, max run time, default inputs, parameter
descriptions, or a number of other things. As an example, the guide below goes
through the process of adding a reference to default input data.

.. note:

   If there is a major change to an app code base, as in a new version is
   released, it would be more appropriate to
   `deploy <create_a_custom_app.html>`>>
   a new version of the app instead of modifying the existing app.


Modify the App Json
-------------------

To modify a Tapis app after it has been deployed, edit the original app ``json``
file and use the command line interface to push the changes to the tenant. Below
is an example app ``json`` file from a
`previous section <create_a_custom_app.html>`__
of this how-to guide:

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



If you followed the
`Create a Custom App <create_a_custom_app.html>`__
how-to guide, then you may have a sample fastq file located here:

.. code-block:: bash

   agave://data-tacc-work-username/test-data/SP1.fq

Modify the original ``app.json`` file to include the complete URI to this sample
data as follows:

.. code-block:: json
   :emphasize-lines: 6

   {
   "inputs":[
     {
       "id": "fastq",
       "value": {
         "default": "agave://tacc.work.taccuser/test-data/SP1.fq",
         "visible": true,
          "required": true
       }
    }

Update the App
--------------

Then, push the app update by performing the following:

.. code-block::

   $ tapis apps update -F app.json taccuser-fastqc_app-0.11.9



If successful, Tapis will automatically increment the revision number associated
with the app. To confirm, use the ``tapis apps show`` command:

.. code-block:: bash

   $ tapis apps show -c id -c revision taccuser-fastqc_app-0.11.9
   +----------+----------------------------+
   | Field    | Value                      |
   +----------+----------------------------+
   | id       | taccuser-fastqc_app-0.11.9 |
   | revision | 2                          |
   +----------+----------------------------+


Further Help
------------

Additional fields that can be used in app descriptions can be found in the
`Tapis Documentation <https://tacc-cloud.readthedocs.io/en/latest/>`_.
