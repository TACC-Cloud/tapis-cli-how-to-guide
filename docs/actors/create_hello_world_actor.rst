Initialize a new Tapis Actor project
====================================

This guide will demonstrate how to create a custom actor from scratch. It is
assumed you are already familiar with how to
`Work with Actors <work_with_actors.html>`__.
In this example, we will build a simple word count actor that counts and prints
the number of words in a provided message.

We will demonstrate how to initialize an actor project from scratch.


Create a project "hello_world_actor"
-----------------------------------
To get started with creating an actor, running the ``tapis actors init`` command will fetch a very simple
code skeleton you can fill in and deploy.

For example:

.. code-block:: bash

   $ tapis actors init

   +-------+-----------------------------------------------------+
   | stage | message                                             |
   +-------+-----------------------------------------------------+
   | setup | Project path: ./new_actor                           |
   | setup | CookieCutter variable name=new_actor                |
   | setup | CookieCutter variable project_slug=new_actor        |
   | setup | CookieCutter variable docker_namespace=reshg        |
   | setup | CookieCutter variable docker_registry=e             |
   | clone | Project path: ./new_actor                           |
   +-------+-----------------------------------------------------+



.. note::

   There are many project templates you can start working with.  See tapis actors init -L
   for an up to date listing.



.. code-block:: bash

   $ tapis actors init --list-templates
   +--------------------+--------------------+--------------------------------------------------------+----------+
   | id                 | name               | description                                            | level    |
   +--------------------+--------------------+--------------------------------------------------------+----------+
   | default            | Default            | Basic code and configuration skeleton                  | beginner |
   | echo               | Echo               | Echo message                                           | beginner |
   | hello_world        | Hello World        | Say Hello, World!                                      | beginner |
   | sd2e_base          | sd2e_base          | Default reactor context for                            | beginner |
   |                    |                    | docker://sd2e/reactors:python3                         |          |
   | tacc_reactors_base | tacc_reactors_base | Default actor context for                              | beginner |
   |                    |                    | docker://sd2e/reactors:python3                         |          |
   +--------------------+--------------------+--------------------------------------------------------+----------+

To use one of these templates:

.. code-block:: bash

   $ tapis actors init --template hello_world


Components of an Actor
----------------------

The new_actor/ project would contain the following files:

.. code-block:: bash

   $ tree ../new_actor/
   new-actor/
   ├── Dockerfile
   ├── project.ini
   ├── config.yml
   ├── default.py
   ├── requirements.txt
   ├── secrets.jsonsample
   └── message.jsonschema


Write the Actor Function
------------------------

The ``default.py`` script can be renamed to ``hello_world.py``. The python script is where the code for your
main function can be found. An example of a functional actor that says "Hello, World" is:

.. code-block:: python

    """Say Hello, World or the message received from user input"""
    from agavepy.actors import get_context

    # function to print the message
    def say_hello_world(m):
    """Print message from user if present, else echo "Hello, World"""
        if m == " ":
            print("Actor says: Hello, World")
        else:
            print("Actor received message: {}".format(m))

    def main():
    """Main entry to grab message context from user input"""
        context = get_context()
        message = context['raw_message']
        say_hello_world(message)

    if __name__ == '__main__':
        main()


This code makes use of the **agavepy** python library which we will install in
the Docker container. The library includes an "actors" object which is useful to
grab the message and other context from the environment. And, it can be used to
interact with other parts of the Tapis platform. Add the above code to your
``hello_world.py`` file.


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
   ADD hello_world.py /hello_world.py

   # command to run the python script
   CMD ["python", "/hello_world.py"]

.. tip::

   Creating small Docker images is important for maintaining actor speed and
   efficiency

Runtime Preparation
-------------------

1. Define secrets.json

   Rename secrets.json.sample to secrets.json, and obtain the required values from the Infrastructure team for secrets.json.

2. Define message.jsonschema

   Define the Schema for Actor launch message.

Build and Push the Dockerfile
-----------------------------

The Docker image must be pushed to a public repository in order for the actor
to use it. Use the following Docker commands in your local actor folder to build
and push to a repository that you have access to:

.. code-block:: bash

   # Build and tag the image
   $ docker build -t taccuser/hello-world:1.0 .
   Sending build context to Docker daemon  4.096kB
   Step 1/5 : FROM python:3.7-slim
   ...
   Successfully built b0a76425e8b3
   Successfully tagged taccuser/hello-world:1.0

   # Push the tagged image to Docker Hub
   $ docker push taccuser/hello-world:1.0
   The push refers to repository [docker.io/taccuser/word-count]
   ...
   1.0: digest: sha256:67cc6f6f00589d9ae83b99d779e4893a25e103d07e4f660c14d9a0ee06a9ddaf size: 1995


Create the Actor
----------------

Next, create an actor referring to the Docker repository above.

.. code-block:: bash

   $ tapis actors create --repo taccuser/hello-world:1.0 \
                         -n hello-world \
                         -d "Actor to say Hello, World"
   +----------------+----------------------------+
   | Field          | Value                      |
   +----------------+----------------------------+
   | id             | NN5N0kGDvZQpA              |
   | name           | hello-world                |
   | owner          | taccuser                   |
   | image          | taccuser/hello-world:1.0   |
   | lastUpdateTime | 2021-07-14T22:25:06.171534 |
   | status         | SUBMITTED                  |
   | cronOn         | False                      |
   +----------------+-----------------------------+

After a few seconds, the actor should be in state "READY", meaning it is ready
to accept and process messages. Verbosely show the actor metadata to see that
it's status is "READY", it is pointing to the correct docker image, and that it
received the environment variables from ``environment.json``:

.. code-block:: bash
   :emphasize-lines: 7,11,20

   $ tapis actors show -v NN5N0kGDvZQpA
   {
    "id": "NN5N0kGDvZQpA",
    "name": "example-actor",
    "description": "Test actor that says Hello, World",
    "owner": "sgopal",
    "image": "tacc/hello-world:latest",
    "createTime": "2021-07-14T22:25:06.171Z",
    "lastUpdateTime": "2021-07-14T22:25:06.171Z",
    "defaultEnvironment": {},
    "gid": 862347,
    "hints": [],
    "link": "",
    "mounts": [],
    "privileged": false,
    "queue": "default",
    "stateless": true,
    "status": "READY",
    "statusMessage": " ",
    "token": true,
    "uid": 862347,
    "useContainerUid": false,
    "webhook": "",
    "cronOn": false,
    "cronSchedule": null,
    "cronNextEx": null,
    "_links": {
      "executions": "https://api.tacc.utexas.edu/actors/v2/NN5N0kGDvZQpA/executions",
      "owner": "https://api.tacc.utexas.edu/profiles/v2/sgopal",
      "self": "https://api.tacc.utexas.edu/actors/v2/NN5N0kGDvZQpA"
      }
    }



Run a Test Execution
--------------------

Finally, pass a message to the actor to run a test execution. The number of
words in the message should be returned in the actor execution logs:

.. code-block:: bash

   # Send a message to the word-count actor
   $ tapis actors submit -m "Hello, World" NN5N0kGDvZQpA
   +-------------+-------------------------------------+
   | Field       | Value                               |
   +-------------+-------------------------------------+
   | executionId | NN5N0kGDvZQpA                       |
   | msg         | Hello, World                        |
   +-------------+-------------------------------------+

   # List executions of the word-count actor
   $ tapis actors execs list NN5N0kGDvZQpA
   +---------------+----------+
   | executionId   | status   |
   +---------------+----------+
   | N4xQ5WM5Np1X0 | COMPLETE |
   +---------------+----------+

   # Get the logs from the completed actor execution
   $ tapis actors execs logs NN5N0kGDvZQpA N4xQ5WM5Np1X0
   Logs for execution N4xQ5WM5Np1X0
    Actor received message: Hello, World

The actor can also be run synchronously using ``tapis actors run``:

.. code-block:: bash

   $ tapis actors run -m "Hello, World" NN5N0kGDvZQpA
   Actor received message: Hello, World


Next Steps
----------

Remember to put your actor under version control. Use a ``.gitignore`` file to
avoid accidentally committing anything that contains API keys or passwords.

Please refer to the
`Abaco Documentation <https://tacc-cloud.readthedocs.io/projects/abaco/en/latest/index.html>`_
for more information on creating and working with actors.
