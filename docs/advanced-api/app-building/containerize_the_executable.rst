Containerize the Executable
===========================

If an image of your executable already exists, and was created by a trusted
source, consider using that rather than building your own. You may find existing
images on hubs such as
`Docker Hub <https://hub.docker.com/>`_,
`BioContainers <https://biocontainers.pro/>`_,
or
`Quay.io <https://quay.io/>`_.


This tutorial is a quick and dirty summary of how to build your own Docker image
as if there is not one available for your executable. This is not meant to
replace the full
`Docker documentation <https://docs.docker.com/develop/>`_.

We will continue with the example of FastQC from the
`previous page <initialize_the_app_directory.html>`__.

Choose a Source Image
---------------------

*Prerequisite:* You should have a
`Docker ID <https://hub.docker.com>`_
and docker should be installed on your local machine.

The only dependency for
`FastQC <https://www.bioinformatics.babraham.ac.uk/projects/fastqc/>`_
is a reasonably recent Java Runtime Environment. Thus, most modern Linux OS-es
should suffice. The TACC Docker hub organization provides a few base images that
would work for this. Pull one manually now so it is in your environment:

.. code-block:: bash

   $ docker pull tacc/tacc-ubuntu18-mvapich2.3-psm2


A list of available base images can be found
`here <https://github.com/TACC/tacc-containers>`_.

Also, now is a good time to prepare a ``src/`` directory in the application
bundle. If your code is a python script, for example, this is where you want to
put it. This directory is not necessary, but nice to have if you want to keep
your executable together with the app assets:

.. code-block:: bash

   $ cd ~/fastqc-app/
   $ mkdir src/


Install and Test Interactively
------------------------------

The installation process for ``fastqc`` is extremely simple. It is good practice
to test the installation interactively, and record the steps for a Dockerfile:

.. code-block:: bash

   # Start an interactive docker session
   $ docker run --rm -it tacc/tacc-ubuntu18-mvapich2.3-psm2

   # Update and install necessary packages
   [docker] $ apt-get update
   [docker] $ apt-get upgrade -y
   [docker] $ apt-get install wget -y
   [docker] $ apt-get install zip -y
   [docker] $ apt-get install default-jre -y

   # Install FastQC
   [docker] $ wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip
   [docker] $ unzip fastqc_v0.11.9.zip
   [docker] $ rm fastqc_v0.11.9.zip
   [docker] $ chmod +x /FastQC/fastqc


After a bit of trial and error, the commands above are a reasonably short path
to installing the `fastqc` executable. You can test it from within the docker
image to make sure it is working by, for example:

.. code-block:: bash

   [docker] $ /FastQC/fastqc -h

               FastQC - A high throughput sequence QC analysis tool

   SYNOPSIS

           fastqc seqfile1 seqfile2 .. seqfileN

       fastqc [-o output dir] [--(no)extract] [-f fastq|bam|sam]
              [-c contaminant file] seqfile1 .. seqfileN
   ... etc.


Note on Source Code and Mounting Directories
--------------------------------------------

In this instance, we could have downloaded the source zip file for FastQC
directly to the ``src/`` directory of our app bundle, then mounted that directory
within the image, e.g.:

.. code-block:: bash

   $ cd ~/fastqc-app/src/
   $ wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip
   $ docker run --rm -it -v $PWD:/opt/src tacc/tacc-ubuntu18-mvapich2.3-psm2
   ... etc.


That route is perfectly reasonable and can be followed here. In fact, if your
app is a standalone python script, for example, this is the best method for
including it in your Docker image.

However, some packages have very large zip or tar.gz files (100s of MB), and
would be cumbersome to keep in this fastqc app bundle folder. It is up to the
app developer to find the balance between completeness of source files and
responsible disk usage.

Here, we decide to not download the source permanently. Instead, we make a
record of where the source came from. For example:

.. code-block:: bash

   $ cd ~/fastqc-app/src/
   $ echo "Source: https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip" \
       >> README.md


Write the Dockerfile
--------------------

Next, translate the steps required to install your software package into a
resonable ``Dockerfile``. The ``Dockerfile`` should be located at the root
directory, ``~/fastqc-app/Dockerfile``:

.. code-block:: text

   FROM tacc/tacc-ubuntu18-mvapich2.3-psm2

   RUN apt-get update \
       && apt-get upgrade -y \
       && apt-get install wget -y \
       && apt-get install zip -y \
       && apt-get install default-jre -y

   RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip \
       && unzip fastqc_v0.11.9.zip \
       && rm fastqc_v0.11.9.zip \
       && chmod +x FastQC/fastqc

   ENV PATH "/FastQC/:$PATH"


Build and Test the Image
------------------------

Navigate to the top of the app directory, ``~/fastqc-app/``, and the command to
build a new Docker image is:

.. code-block:: bash

   $ docker build -f Dockerfile --force-rm -t taccuser/fastqc:0.11.9 ./


Once built, test the new image with an example command:

.. code-block:: bash

   $ docker run --rm fastqc:0.11.9 fastqc -h
   - or -
   $ docker run --rm fastqc:0.11.9 perl /FastQC/fastqc -h


.. note::

   Calling the complete path to executables is sometimes safer than relying on
   PATH environment variables

If you see the FastQC help text, the installation likely was successful. At this
time, it might be prudent to test with real data as well. Download some test
data into a `~/fastqc-app/tests/` directory:

.. code-block:: bash

   $ cd ~/fastqc-app/
   $ mkdir tests/

   # Download random sample data or provide your own
   # wget https://molb7621.github.io/workshop/_downloads/SP1.fq


Next, run the FastQC pipeline on the example data:

.. code-block:: bash

   $ docker run -v $PWD:/data fastqc:0.11.9 perl /FastQC/fastqc /data/SP1.fq


If successful, you should find the output files ``SP1_fastqc.html`` and
``SP1_fastqc.zip`` in the ``~/fastqc-app/tests/`` directory.



Push Your Image to the Cloud
----------------------------

If you are happy with the tests, push your Docker image to a publicly available
repository. It can be your own personal repository as long as it is set to
public, and not private. To push to your own repository, make sure it was
namespaced with your Docker ID. Then:


.. code-block:: bash

   $ docker push taccuser/fastqc:0.11.9


Assemble Run Commands
---------------------

The final step is to put instructions in ``runner.sh`` on how the app should be
run. In general, these are the same commands we used for testing above.

Most of the lines in the default file should be left alone. See the last three
lines of this file for what should be added:

.. code-block:: text

   # Allow over-ride
   if [ -z "${CONTAINER_IMAGE}" ]
   then
       version=$(cat ./_util/VERSION)
       CONTAINER_IMAGE="index.docker.io/taccuser/fastqc:${version}"
   fi
   . lib/container_exec.sh

   # Write an excution command below that will run a script or binary inside the
   # requested container, assuming that the current working directory is
   # mounted in the container as its WORKDIR. In place of 'docker run'
   # use 'container_exec' which will handle setup of the container on
   # a variety of host environments.
   #
   # Here is a template:
   #
   # container_exec ${CONTAINER_IMAGE} COMMAND OPTS INPUTS
   #
   # Here is an example of counting words in local file 'poems.txt',
   # outputting to a file 'wc_out.txt'
   #
   # container_exec ${CONTAINER_IMAGE} wc poems.txt > wc_out.txt
   #

   # set -x

   # set +x

   COMMAND="perl /FastQC/fastqc"
   PARAMS="${fastq}"
   container_exec ${CONTAINER_IMAGE} ${COMMAND} ${PARAMS}


Update the ``VERSION`` File
---------------------------

Finally, put the version / docker tag into the file located at
``~/fastqc_app/assets/lib/VERSION``:

.. code-block:: bash

   $ echo "0.11.9" >> ~/fastqc_app/assets/lib/VERSION
   $ cat ~/fastqc_app/assets/lib/VERSION
   0.11.9
