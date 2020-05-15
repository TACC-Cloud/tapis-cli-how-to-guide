Install the CLI
===============

The Tapis CLI is available as a Python package. We highly recommend using Python
3.7+ as the Python runtime behind the Tapis CLI. We support Python 2.7 for
legacy applications, but on a best-effort basis as Python 2.7 is a deprecated
language.


Install with Pip
----------------

.. code-block:: bash

   $ pip install tapis-cli


Install from Source
-------------------

.. code-block:: bash

   $ git clone https://github.com/TACC-Cloud/tapis-cli-ng.git
   $ cd tapis-cli-ng/
   $ pip install --upgrade --user .


Run in a Container
------------------

.. code-block:: bash

   $ docker run --rm -it -v ${PWD}:/work -v ${HOME}/.agave:/root/.agave \
         tacc/tapis-cli-ng:latest /bin/bash


