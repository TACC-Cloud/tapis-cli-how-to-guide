Work with Actors
================

In Tapis, **actors** are container-based functions-as-a-service that follow the
actor model of concurrent computation. An actor responds to messages it receives
by changing its state, performing an action, sending out response messages, or
all of the above.

The function an actor performs is exposed as the default command in a container.
It is typically quick and requires little processing power - i.e. an app may be
configured to
`run FastQC <../advanced-api/create_a_custom_app.html>`__,
and an actor may trigger a job using that app.

The guide below is a brief introduction to interacting with actors on the Tapis
platform. For a full reference guide to actors, see the
`Abaco Documentation <https://tacc-cloud.readthedocs.io/projects/abaco/en/latest/index.html>`_.

Create a New Actor
------------------

The function of an actor is exposed as the default command in a Docker
container. Here, we will create an actor from an existing Docker container image
called **tacc/hello-world:latest** available on
`Docker Hub <https://hub.docker.com/repository/docker/tacc/hello-world>`__.
The default command for this container simply prints the message "Hello, World" or
the message sent to it, which will be captured in the actor logs.

Create the actor as:

.. code-block:: bash

   $ tapis actors create --repo tacc/hello-world:latest \
                         -n example-actor \
                         -d "Test actor that says Hello, World"
   +----------------+-----------------------------+
   | Field          | Value                       |
   +----------------+-----------------------------+
   | id             | NN5N0kGDvZQpA               |
   | name           | example-actor               |
   | owner          | taccuser                    |
   | image          | tacc/hello-world:latest     |
   | lastUpdateTime | 2021-07-14T22:25:06.171534  |
   | status         | SUBMITTED                   |
   | cronOn         | False                       |
   +----------------+-----------------------------+

The ``--repo`` flag points to the Docker Hub repo on which this actor is based,
the ``-n`` flag and ``-d`` flag attach a human-readable name and description to
the actor, the ``-e`` flags demonstrate how to set (optional) environment
variables for the actor.

The resulting actor is assigned an id: ``NN5N0kGDvZQpA``. The actor id can be
queried by:

.. code-block:: bash

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


Above, you can see the plain text name, description, and any default environment
variables that were passed on the command line. In addition, you can see the
"status" of the actor is "READY", meaning it is ready to receive and act on
messages. Finally, you can list all actors visible to you with:

.. code-block:: bash

   $ tapis actors list
   +---------------+---------------+----------+-----------------------------+----------------------------+--------+-------+
   | id            | name          | owner    | image                       | lastUpdateTime             | status | cronOn|
   +---------------+---------------+----------+-----------------------------+----------------------------+--------+-------+
   | NN5N0kGDvZQpA | example-actor | taccuser | tacc/hello-world:latest     | 2021-07-14T22:25:06.171Z   | READY  | False |
   +---------------+---------------+----------+-----------------------------+----------------------------+--------+-------+


Probe the Underlying Container
------------------------------

An actor now exists and is waiting for a message to respond to. But, how will
the actor respond when sent a message? We can probe the underlying container to
figure out what this specific actor will do. First pull the container locally:

.. code-block:: bash

   $ docker pull tacc/hello-world:latest
   latest: Pulling from tacc/hello-world
   Digest: sha256:baf7241b9d6fb1b123825021b831337307b9fa0aa4d45b14c9405ebf2a36a929
   Status: Image is up to date for tacc/hello-world:latest
   docker.io/tacc/hello-world:latest

Then find the default command for the container:

.. code-block:: bash

   $ docker inspect tacc/hello-world:latest | jq ".[].ContainerConfig.Cmd"
   [
    "/bin/sh",
    "-c",
    "#(nop) ",
    "CMD [\"python\" \"/hello_world.py\"]"
   ]

It runs ``hello_world.py`` at the root level. Print out the contents of ``hello_world.py``
to inspect:

.. code-block:: bash

   $ docker run --rm tacc/hello-world:latest cat /hello_world.py

.. code-block:: python
   :emphasize-lines: 13

   1   """Say Hello, World or the message received from user input"""
   2   from agavepy.actors import get_context
   3
   4   def say_hello_world(m):
   5   """Print message from user if present, else echo "Hello, World"""
   6       if m == " ":
   7           print("Actor says: Hello, World")
   8       else:
   9           print("Actor received message: {}".format(m))
   10
   11  def main():
   12  """Main entry to grab message context from user input"""
   13      context = get_context()
   14      message = context['raw_message']
   15      say_hello_world(message)
   16
   17  if __name__ == '__main__':
   18      main()


This container, when run, will first get the message that was passed to it (from
the ``get_context()`` function, line 10). Then it will print various parts of
the message and the environment.

Submit a Message to the Actor
-----------------------------

Next, let's craft a simple message to send to the reactor. Messages can be plain
text or in JSON format. When using the python actor libraries as in the example
above, JSON-formatted messages are made available as python dictionaries.

.. code-block:: bash

   # Write a message
   $ export MESSAGE='Hello, World'
   $ echo $MESSAGE
   Hello, World

   $ Submit the message to the actor
   $ tapis actors submit -m "$MESSAGE" NN5N0kGDvZQpA
   +-------------+---------------+
   |  Field      | Value         |
   +-------------+---------------+
   | executionId | N4xQ5WM5Np1X0 |
   | msg         | Hello, World  |
   +-------------+---------------+

The id of the actor (``N4xQ5WM5Np1X0``) was used on the command line to specify
which actor should receive the message. In response, an "execution id"
(``N4xQ5WM5Np1X0``) is returned. An execution is a specific instance of an actor.
List all the executions for a given actor as:

.. code-block::bash

   $ tapis actors execs list NN5N0kGDvZQpA
   +---------------+----------+
   | executionId   | status   |
   +---------------+----------+
   | N4xQ5WM5Np1X0 | COMPLETE |
   +---------------+----------+

The above execution has already completed. Show detailed information for the
execution with:

.. code-block:: bash

   $ tapis actors execs show -v boEg3mEvrKO5w ayB45Oe8GJvAA
   {
      "actorId": "NN5N0kGDvZQpA",
      "apiServer": "https://api.tacc.utexas.edu",
      "cpu": 121748743,
      "exitCode": 0,
      "finalState": {
        "Dead": false,
        "Error": "",
        "ExitCode": 0,
        "FinishedAt": "2021-07-14T22:32:45.602Z",
        "OOMKilled": false,
        "Paused": false,
        "Pid": 0,
        "Restarting": false,
        "Running": false,
        "StartedAt": "2021-07-14T22:32:45.223Z",
        "Status": "exited"
      },
      "id": "N4xQ5WM5Np1X0",
      "io": 176,
      "messageReceivedTime": "2021-07-14T22:32:37.051Z",
      "runtime": 1,
      "startTime": "2021-07-14T22:32:44.752Z",
      "status": "COMPLETE",
      "workerId": "JABKl4BeDwXJD",
      "_links": {
        "logs": "https://api.tacc.utexas.edu/actors/v2/NN5N0kGDvZQpA/executions/N4xQ5WM5Np1X0/logs",
        "owner": "https://api.tacc.utexas.edu/profiles/v2/sgopal",
        "self": "https://api.tacc.utexas.edu/actors/v2/NN5N0kGDvZQpA/executions/N4xQ5WM5Np1X0"
      }
   }


Check the Logs for an Execution
-------------------------------

An execution's logs will contain whatever was printed to STDOUT / STDERR by the
actor. In our demo actor, we just expect the actor to print the message passed to it.

.. code-block:: bash

   $ tapis actors execs logs NN5N0kGDvZQpA N4xQ5WM5Np1X0
   Logs for execution N4xQ5WM5Np1X0
    Actor received message: Hello, World



Sure enough, the information in the execution logs match what we expected
``hello_world.py`` to print. The message was pulled in by the
``get_context()`` function. It was not done in this script, but in a normal
scenario, the actor would then act on the contents of that message to, e.g.,
kick off a job, perform some data management, send messages to other actors, or
more.


Run Synchronously
-----------------

The previous message submission (with ``tapis actors submit``) was an
*asynchronous* run, meaning the command prompt detached from the process after
it was submitted to the actor. In that case, it was up to us to check the execution
to see if it had completed and manually print the logs.

There is also a mode to run actors *synchronously* using ``tapis actors run``,
meaning the command line stays attached to the process awaiting a response after
sending a message to the actor. For example:

.. code-block:: bash

   $ tapis actors run -m "$MESSAGE" NN5N0kGDvZQpA
   FULL CONTEXT:
   {
     "username": "taccuser",
     "HOSTNAME": "33d4dd334ef9",
     "_abaco_worker_id": "X5xGkZ0lol0D3",
     "raw_message": "Hello, World",
     "actor_dbid": "TACC-PROD_boEg3mEvrKO5w",
     "new_foo": "new_bar",
     "_abaco_container_repo": "tacc/hello-world:latest",
     "content_type": null,
     "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
     "MSG": "{\"key1\":\"value1\", \"key2\":\"value2\"}",
     "HOME": "/",
     "_abaco_actor_state": "{}",
     "_abaco_actor_name": "example-actor",
     "_abaco_Content_Type": "str",
     "execution_id": "jP3RExQW108wM",
     "_abaco_synchronous": "True",
     "_abaco_access_token": "de6d11bdbb5a16bdd85beec692b1b283",
     "message_dict": {
       "key2": "value2",
       "key1": "value1"
     },
     "_abaco_api_server": "https://api.tacc.utexas.edu",
     "_abaco_actor_dbid": "TACC-PROD_boEg3mEvrKO5w",
     "_abaco_jwt_header_name": "X-Jwt-Assertion-Tacc-Prod",
     "_abaco_actor_id": "boEg3mEvrKO5w",
     "_abaco_execution_id": "jP3RExQW108wM",
     "state": "{}",
     "_abaco_username": "taccuser",
     "actor_id": "boEg3mEvrKO5w"
   }
   ...

The output above is truncated because it is mostly the same response as our
first execution of the actor. This time, however, we did not need to query the
logs for this execution for them to print to screen - that was done
automatically. In addition, the new environment variable settings can be seen
in the context (see highlighted line).


Delete an Actor
---------------

Similar to other resources in Tapis, actors can be deleted with the following:

.. code-block:: bash

   $ tapis actors delete NN5N0kGDvZQpA
   +----------+-------------------+
   | Field    | Value             |
   +----------+-------------------+
   | deleted  | ['NN5N0kGDvZQpA'] |
   | messages | []                |
   +----------+-------------------+

This will delete the actor and any associated executions.
