Create a Custom Actor
=====================

This guide will demonstrate how to create a custom actor from scratch. It is
assumed you are already familiar with how to
`Work with Actors <work_with_actors.html>`__.
In this example, we will build a simple word count actor that counts and prints
the number of words in a provided message.


Components of an Actor
----------------------

Make a new directory and add the following files:

.. code-block:: bash

   $ mkdir word-count-actor/ && cd word-count-actor/

   $ touch Dockerfile word_count.py environment.json

   $ tree ../word-count-actor/
   word-count-actor/
   ├── Dockerfile
   ├── environment.json
   └── word_count.py

   0 directories, 3 files


Write the Actor Function
------------------------

The ``word_count.py`` python script is where the code for your main function can
be found. An example of a functional actor that performs a word count is:

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


Define Environment Variables
----------------------------

The ``environment.json`` file may contain useful environment variables or
configurations to pass to the actor at creation time. These variables will be
part of the "context" taken from the environment, as in the example python
script above. For the purposes of this example, add the following definition to
``environment.json``:

.. code-block:: json

   {
     "foo": "bar"
   }


Create a Dockerfile
-------------------

The only requirements are python and the agavepy python library, which is
available through
`PyPi <https://pypi.org/>`_.
A bare-bones Dockerfile needs to satisfy those dependencies, add the actor
python script, and set a default command to run the actor python script. Add
the following lines to your ``Dockerfile``:

.. code-block:: bash

   FROM python:3.7-slim

   RUN pip install --no-cache-dir agavepy==0.9.3

   ADD word_count.py /word_count.py

   RUN chmod +x /word_count.py

   CMD ["python3", "/word_count.py"]

.. tip::

   Creating small Docker images is important for maintaining actor speed and
   efficiency

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
