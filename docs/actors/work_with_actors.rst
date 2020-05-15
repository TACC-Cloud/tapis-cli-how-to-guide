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
called **jturcino/abaco-trial:latest** available on
'Docker Hub <https://hub.docker.com/r/jturcino/abaco-trial/>'_
The default command for this container simply prints some information about the
current environment to STDOUT, which will be captured in the actor logs.

Create the actor as:

.. code-block:: bash

   $ tapis actors create --repo jturcino/abaco-trial:latest \
                         -n example-actor \
                         -d "Test actor that prints environment info" \
                         -e foo=bar \
                         -e bar=baz
   +----------------+-----------------------------+
   | Field          | Value                       |
   +----------------+-----------------------------+
   | id             | boEg3mEvrKO5w               |
   | name           | example-actor               |
   | owner          | taccuser                    |
   | image          | jturcino/abaco-trial:latest |
   | lastUpdateTime | 2020-05-15 02:39:37.230860  |
   | status         | SUBMITTED                   |
   +----------------+-----------------------------+

The ``--repo`` flag points to the Docker Hub repo on which this actor is based,
the ``-n`` flag and ``-d`` flag attach a human-readable name and description to
the actor, and the ``-e`` flags demonstrate how to set (optional) environment
variables for the actor.

The resulting actor is assigned an id: ``boEg3mEvrKO5w``. The actor id can be
queried by:

.. code-block:: bash

   $ tapis actors show -v boEg3mEvrKO5w
   {
     "id": "boEg3mEvrKO5w",
     "name": "example-actor",
     "description": "Test actor that prints environment info",
     "owner": "taccuser",
     "image": "jturcino/abaco-trial:latest",
     "createTime": "2020-05-15 02:39:37.230860",
     "lastUpdateTime": "2020-05-15 02:39:37.230860",
     "defaultEnvironment": {
       "bar": "baz",
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
       "executions": "https://api.tacc.utexas.edu/actors/v2/boEg3mEvrKO5w/executions",
       "owner": "https://api.tacc.utexas.edu/profiles/v2/taccuser",
       "self": "https://api.tacc.utexas.edu/actors/v2/boEg3mEvrKO5w"
     }
   }

Above, you can see the plain text name, description, and default environment
variables that were passed on the command line. In addition, you can see the
"status" of the actor is "READY", meaning it is ready to receive and act on
messages. Finally, you can list all actors visible to you with:

.. code-block:: bash

   $ tapis actors list
   +---------------+---------------+----------+-----------------------------+----------------------------+--------+
   | id            | name          | owner    | image                       | lastUpdateTime             | status |
   +---------------+---------------+----------+-----------------------------+----------------------------+--------+
   | boEg3mEvrKO5w | example-actor | taccuser | jturcino/abaco-trial:latest | 2020-05-15 02:39:37.230860 | READY  |
   +---------------+---------------+----------+-----------------------------+----------------------------+--------+


Probe the Underlying Container
------------------------------

An actor now exists and is waiting for a message to respond to. But, how will
the actor respond when sent a message? We can probe the underlying container to
figure out what this specific actor will do. First pull the container locally:

.. code-block:: bash

   $ docker pull jturcino/abaco-trial:latest
   latest: Pulling from jturcino/abaco-trial
   ...
   Digest: sha256:976a83992e1f36b6a1afa0ba71c59ab3d5ff17e66a2f6b1ff1c8a370003087b4
   Status: Downloaded newer image for jturcino/abaco-trial:latest
   docker.io/jturcino/abaco-trial:latest

Then find the default command for the container:

.. code-block:: bash

   $ docker inspect jturcino/abaco-trial:latest | jq ".[].ContainerConfig.Cmd"
   [
     "/bin/sh",
     "-c",
     "#(nop) ",
     "CMD [\"python\" \"/script.py\"]"
   ]

It runs ``script.py`` at the root level. Print out the contents of ``script.py``
to inspect:

.. code-block:: bash

   $ docker run --rm jturcino/abaco-trial:latest cat /script.py

.. code-block:: python
   :emphasize-lines: 10

    1 #!/usr/bin/env python
    2
    3 import os
    4 import sys
    5 import json
    6 from agavepy.actors import get_context
    7
    8 if __name__ == '__main__':
    9
   10     context = get_context()
   11     print 'FULL CONTEXT:'
   12     print json.dumps(context, indent=2)
   13
   14     print '\nMESSAGE:'
   15     message = context.message_dict
   16     print json.dumps(message, indent=2)
   17
   18     print '\nFULL ENVIRONMENT:'
   19     print json.dumps(dict(os.environ), indent=2)
   20
   21     print '\nROOT FILES:'
   22     print ' '.join(os.listdir('/'))


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
   $ export MESSAGE='{"key1":"value1", "key2":"value2"}'
   $ echo $MESSAGE
   {"key1":"value1", "key2":"value2"}

   $ Submit the message to the actor
   $ tapis actors submit -m "$MESSAGE" boEg3mEvrKO5w
   +-------------+------------------------------------+
   | Field       | Value                              |
   +-------------+------------------------------------+
   | executionId | ayB45Oe8GJvAA                      |
   | msg         | {"key1":"value1", "key2":"value2"} |
   +-------------+------------------------------------+

The id of the actor (``boEg3mEvrKO5w``) was used on the command line to specify
which actor should receive the message. In response, an "execution id"
(``ayB45Oe8GJvAA``) is returned. An execution is a specific instance of an actor.
List all the executions for a given actor as:

.. code-block::bash

   $ tapis actors execs list boEg3mEvrKO5w
   +---------------+----------+
   | executionId   | status   |
   +---------------+----------+
   | ayB45Oe8GJvAA | COMPLETE |
   +---------------+----------+

The above execution has already completed. Show detailed information for the
execution with:

.. code-block:: bash

   $ tapis actors execs show -v boEg3mEvrKO5w ayB45Oe8GJvAA
   {
     "actorId": "boEg3mEvrKO5w",
     "apiServer": "https://api.tacc.utexas.edu",
     "cpu": 0,
     "exitCode": 0,
     "finalState": {
       "Dead": false,
       "Error": "",
       "ExitCode": 0,
       "FinishedAt": "2020-05-15T02:52:27.885125138Z",
       "OOMKilled": false,
       "Paused": false,
       "Pid": 0,
       "Restarting": false,
       "Running": false,
       "StartedAt": "2020-05-15T02:52:27.778284083Z",
       "Status": "exited"
     },
     "id": "ayB45Oe8GJvAA",
     "io": 0,
     "messageReceivedTime": "2020-05-15 02:52:20.737730",
     "runtime": 1,
     "startTime": "2020-05-15 02:52:27.137874",
     "status": "COMPLETE",
     "workerId": "AMgVYDQ8lvlP6",
     "_links": {
       "logs": "https://api.tacc.utexas.edu/actors/v2/boEg3mEvrKO5w/executions/ayB45Oe8GJvAA/logs",
       "owner": "https://api.tacc.utexas.edu/profiles/v2/taccuser",
       "self": "https://api.tacc.utexas.edu/actors/v2/boEg3mEvrKO5w/executions/ayB45Oe8GJvAA"
     }
   }


Check the Logs for an Execution
-------------------------------

An execution's logs will contain whatever was printed to STDOUT / STDERR by the
actor. In our demo actor, we just expect the actor to print various parts of the
environment.

.. code-block:: bash

   $ tapis actors execs logs boEg3mEvrKO5w ayB45Oe8GJvAA
   Logs for execution ayB45Oe8GJvAA
    FULL CONTEXT:
   {
     "username": "taccuser",
     "_abaco_jwt_header_name": "X-Jwt-Assertion-Tacc-Prod",
     "_abaco_worker_id": "AMgVYDQ8lvlP6",
     "raw_message": "{\"key1\":\"value1\", \"key2\":\"value2\"}",
     "actor_dbid": "TACC-PROD_boEg3mEvrKO5w",
     "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
     "_abaco_access_token": "7a2f635733c430b29819c267590f042",
     "_abaco_container_repo": "jturcino/abaco-trial:latest",
     "content_type": null,
     "HOME": "/",
     "MSG": "{\"key1\":\"value1\", \"key2\":\"value2\"}",
     "bar": "baz",
     "_abaco_api_server": "https://api.tacc.utexas.edu",
     "_abaco_actor_name": "example-actor",
     "_abaco_Content_Type": "str",
     "execution_id": "ayB45Oe8GJvAA",
     "_abaco_synchronous": "False",
     "_abaco_actor_state": "{}",
     "message_dict": {
       "key2": "value2",
       "key1": "value1"
     },
     "_abaco_actor_dbid": "TACC-PROD_boEg3mEvrKO5w",
     "HOSTNAME": "9e70a9f51927",
     "_abaco_actor_id": "boEg3mEvrKO5w",
     "_abaco_execution_id": "ayB45Oe8GJvAA",
     "environment": "_abaco_synchronous",
     "state": "{}",
     "_abaco_username": "taccuser",
     "actor_id": "boEg3mEvrKO5w",
     "foo": "bar"
   }

   MESSAGE:
   {
     "key2": "value2",
     "key1": "value1"
   }

   FULL ENVIRONMENT:
   {
     "_abaco_synchronous": "False",
     "_abaco_actor_state": "{}",
     "bar": "baz",
     "_abaco_actor_id": "boEg3mEvrKO5w",
     "_abaco_actor_dbid": "TACC-PROD_boEg3mEvrKO5w",
     "HOSTNAME": "9e70a9f51927",
     "_abaco_execution_id": "ayB45Oe8GJvAA",
     "_abaco_Content_Type": "str",
     "_abaco_container_repo": "jturcino/abaco-trial:latest",
     "environment": "_abaco_synchronous",
     "_abaco_jwt_header_name": "X-Jwt-Assertion-Tacc-Prod",
     "_abaco_username": "taccuser",
     "_abaco_worker_id": "AMgVYDQ8lvlP6",
     "_abaco_access_token": "7a2f635733c430b29819c267590f042",
     "MSG": "{\"key1\":\"value1\", \"key2\":\"value2\"}",
     "HOME": "/",
     "_abaco_api_server": "https://api.tacc.utexas.edu",
     "_abaco_actor_name": "example-actor",
     "foo": "bar",
     "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
   }

   ROOT FILES:
   bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var _abaco_results.sock .dockerenv requirements.txt agavepy script.py

Sure enough, the information in the execution logs match what we expected
``script.py`` to print. The message dictionary was pulled in by the
```get_context()`` function. It was not done in this script, but in a normal
scenario, the actor would then act on the contents of that message to, e.g.,
kick off a job, perform some data management, send messages to other actors, or
more.


Update an Actor
---------------

Updating an actor would be useful to modify its environment, or to deploy a new
tagged version of the Docker container containing, perhaps, an updated actor
python script. Update an existing actor as:


.. code-block:: bash

   $ tapis actors update --repo jturcino/abaco-trial:latest \
                         -e new_foo=new_bar \
                         boEg3mEvrKO5w
   +----------------+-----------------------------+
   | Field          | Value                       |
   +----------------+-----------------------------+
   | id             | boEg3mEvrKO5w               |
   | name           | example-actor               |
   | owner          | taccuser                    |
   | image          | jturcino/abaco-trial:latest |
   | lastUpdateTime | 2020-05-15 03:03:03.724195  |
   | status         | READY                       |
   +----------------+-----------------------------+

In this example, a new environment variable was provided and the previously-passed
environment variables were omitted. The Docker repo stayed the same, but must
still be passed on the command line


Run Synchronously
-----------------

The previous message submission (with ``tapis actors submit``) was an
*asynchronous* run, meaning the command prompt detached from the process after
it was submit to the actor. In that case, it was up to us to check the execution
to see if it had completed and manually print the logs.

There is also a mode to run actors *synchronously* using ``tapis actors run``,
meaning the command line stays attached to the process awaiting a response after
sending a message to the actor. For example:

.. code-block:: bash
   :emphasize-lines: 9

   $ tapis actors run -m "$MESSAGE" boEg3mEvrKO5w
   FULL CONTEXT:
   {
     "username": "taccuser",
     "HOSTNAME": "33d4dd334ef9",
     "_abaco_worker_id": "X5xGkZ0lol0D3",
     "raw_message": "{\"key1\":\"value1\", \"key2\":\"value2\"}",
     "actor_dbid": "TACC-PROD_boEg3mEvrKO5w",
     "new_foo": "new_bar",
     "_abaco_container_repo": "jturcino/abaco-trial:latest",
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

   $ tapis actors delete boEg3mEvrKO5w
   +----------+-------------------+
   | Field    | Value             |
   +----------+-------------------+
   | deleted  | ['boEg3mEvrKO5w'] |
   | messages | []                |
   +----------+-------------------+

This will delete the actor and any associated executions.
