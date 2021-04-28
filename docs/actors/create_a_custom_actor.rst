Initialize a new Tapis Actor project
====================================

This guide will demonstrate how to create a custom actor from scratch. It is
assumed you are already familiar with how to
`Work with Actors <work_with_actors.html>`__.
In this example, we will build a simple word count actor that counts and prints
the number of words in a provided message.

We will demonstrate how to initialize an actor project from scratch.


Create a project "word_count_actor"
-----------------------------------
To get started with creating an actor, running the ``tapis actors init`` command will fetch a very simple
code skeleton you can fill in and deploy.

For example:

.. code-block:: bash

   $ tapis actors init "word_count_actor"

   +-------+-----------------------------------------------------+
   | stage | message                                             |
   +-------+-----------------------------------------------------+
   | setup | Project path: ./word_count_actor                    |
   | setup | CookieCutter variable name=word_count_actor         |
   | setup | CookieCutter variable project_slug=word_count_actor |
   | setup | CookieCutter variable docker_namespace=reshg        |
   | setup | CookieCutter variable docker_registry=e             |
   | clone | Project path: ./word_count_actor                    |
   +-------+-----------------------------------------------------+



.. note:: 
   
   There are many project templates you can start working with.  See tapis actors init -L
   for an up to date listing.

  

.. code-block:: bash

   $ tapis actors init --list-templates

   +-----------+-----------+-----------------------------------------------------------+----------+
   | id        | name      | description                                               | level    |
   +-----------+-----------+-----------------------------------------------------------+----------+
   | default   | Default   | Basic code and configuration skeleton                     | beginner |
   | echo      | Echo      | Echo input message                                        | beginner |
   | sd2e_base | sd2e_base | Default reactor context for                               | beginner |
   |           |           | docker://sd2e/reactors:python3                            |          |
   +-----------+-----------+-----------------------------------------------------------+----------+

To use one of these templates:

.. code-block:: bash

   $ tapis actors init --template echo
   

Components of an Actor
----------------------

The word_count_actor/ project would contain the following files:

.. code-block:: bash

   $ tree ../word-count-actor/
   word-count-actor/
   ├── Dockerfile
   ├── project.ini
   ├── config.yml
   ├── default.py
   ├── requirements.txt
   ├── secrets.jsonsample
   └── message.jsonschema
   
 
Write the Actor Function
------------------------

The ``default.py`` script can be renamed to ``word_count.py``. The python script is where the code for your
main function can be found. An example of a functional actor that performs a word count is:

.. code-block:: python

   #!/usr/bin/env python
   from agavepy.actors import get_context

   def main():
       context = get_context()
       message = context['raw_message']

       try:
           word_count = len(message.split(' '))
           print('The number of words is: ' + str(word_count))
       except Exception as e:
           print('An unexpected error has occurred: ' + e)

   if __name__ == '__main__':
       main()


This code makes use of the **agavepy** python library which we will install in
the Docker container. The library includes an "actors" object which is useful to
grab the message and other context from the environment. And, it can be used to
interact with other parts of the Tapis platform. Add the above code to your
``word_count.py`` file.


Define Requirements
-------------------

The ``requirements.txt`` file may contain the dependencies required for a project.
The default ``requirements.txt`` contains agavepy python package.

Create a Dockerfile
-------------------

The only requirements are python and the agavepy python library, which is
available through
`PyPi <https://pypi.org/>`_. These are mentioned in the ``requirements.txt`` file
A bare-bones Dockerfile needs to satisfy those dependencies, add the actor
python script, and set a default command to run the actor python script.
The following lines should be present in your ``Dockerfile``:

.. code-block:: bash

   # pull base image
   FROM python:3.7-alpine

   # add requirements.txt to docker container
   ADD requirements.txt /requirements.txt

   # install requirements.txt
   RUN pip3 install -r /requirements.txt

   # add the python script to docker container
   ADD word_count.py /word_count.py

   # command to run the python script
   CMD ["python", "/word_count.py"]

.. tip::

   Creating small Docker images is important for maintaining actor speed and
   efficiency

Runtime Preparation
-------------------

1. Define secrets.json 

   Copy secrets.json.sample to secrets.json, and obtain the required values from the Infrastructure team for secrets.json. 

2. Define message.jsonschema
   
   Schema for Actor launch message. 

Build and Push the Dockerfile
-----------------------------

The Docker image must be pushed to a public repository in order for the actor
to use it. Use the following Docker commands in your local actor folder to build
and push to a repository that you have access to:

.. code-block:: bash

   # Build and tag the image
   $ docker build -t taccuser/word-count:1.0 .
   Sending build context to Docker daemon  4.096kB
   Step 1/5 : FROM python:3.7-slim
   ...
   Successfully built b0a76425e8b3
   Successfully tagged taccuser/word-count:1.0

   # Push the tagged image to Docker Hub
   $ docker push taccuser/word-count:1.0
   The push refers to repository [docker.io/taccuser/word-count]
   ...
   1.0: digest: sha256:67cc6f6f00589d9ae83b99d779e4893a25e103d07e4f660c14d9a0ee06a9ddaf size: 1995


Create the Actor
----------------

Next, create an actor referring to the Docker repository above. Also, pass the
JSON file containing environment variables:

.. code-block:: bash

   $ tapis actors create --repo taccuser/word-count:1.0 \
                         -n word-count \
                         -d "Count the number of words in the message" \
                         -E environment.json
   +----------------+----------------------------+
   | Field          | Value                      |
   +----------------+----------------------------+
   | id             | KKP0jKRGJ5l5K              |
   | name           | word-count                 |
   | owner          | taccuser                   |
   | image          | taccuser/word-count:1.0    |
   | lastUpdateTime | 2020-05-15 18:00:33.685417 |
   | status         | SUBMITTED                  |
   +----------------+----------------------------+

After a few seconds, the actor should be in state "READY", meaning it is ready
to accept and process messages. Verbosely show the actor metadata to see that
it's status is "READY", it is pointing to the correct docker image, and that it
received the environment variables from ``environment.json``:

.. code-block:: bash
   :emphasize-lines: 7,11,20

   $ tapis actors show -v KKP0jKRGJ5l5K
   {
     "id": "KKP0jKRGJ5l5K",
     "name": "word-count",
     "description": "Count the number of words in the message",
     "owner": "taccuser",
     "image": "taccuser/word-count:1.0",
     "createTime": "2020-05-15 18:00:33.685417",
     "lastUpdateTime": "2020-05-15 18:00:33.685417",
     "defaultEnvironment": {
       "foo": "bar"
     },
     "gid": 851953,
     "hints": [],
     "link": "",
     "mounts": [],
     "privileged": false,
     "queue": "default",
     "stateless": true,
     "status": "READY",
     "statusMessage": " ",
     "token": true,
     "uid": 851953,
     "useContainerUid": false,
     "webhook": "",
     "_links": {
       "executions": "https://api.tacc.utexas.edu/actors/v2/KKP0jKRGJ5l5K/executions",
       "owner": "https://api.tacc.utexas.edu/profiles/v2/taccuser",
       "self": "https://api.tacc.utexas.edu/actors/v2/KKP0jKRGJ5l5K"
     }
   }


Run a Test Execution
--------------------

Finally, pass a message to the actor to run a test execution. The number of
words in the message should be returned in the actor execution logs:

.. code-block:: bash

   # Send a message to the word-count actor
   $ tapis actors submit -m "This is a test message with 8 words" KKP0jKRGJ5l5K
   +-------------+-------------------------------------+
   | Field       | Value                               |
   +-------------+-------------------------------------+
   | executionId | K1p3AZZpXjwZr                       |
   | msg         | This is a test message with 8 words |
   +-------------+-------------------------------------+

   # List executions of the word-count actor
   $ tapis actors execs list KKP0jKRGJ5l5K
   +---------------+----------+
   | executionId   | status   |
   +---------------+----------+
   | K1p3AZZpXjwZr | COMPLETE |
   +---------------+----------+

   # Get the logs from the completed actor execution
   $ tapis actors execs logs KKP0jKRGJ5l5K K1p3AZZpXjwZr
   Logs for execution K1p3AZZpXjwZr
    The number of words is: 8

The actor can also be run synchronously using ``tapis actors run``:

.. code-block:: bash

   $ tapis actors run -m "This is an example of running the actor synchronously" KKP0jKRGJ5l5K
   The number of words is: 9


Next Steps
----------

Remember to put your actor under version control. Use a ``.gitignore`` file to
avoid accidentally committing anything that contains API keys or passwords.

Please refer to the
`Abaco Documentation <https://tacc-cloud.readthedocs.io/projects/abaco/en/latest/index.html>`_
for more information on creating and working with actors.
