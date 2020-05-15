Import Data from Alternative Sources
====================================

.. warning::

   This functionality does not yet exist


The Tapis platform enables management of multiple storage systems representing
different hosts under the same tenant namespace. Data can be moved efficiently
directly between hosts (storage systems).


Import Files from other Systems
-------------------------------

To import data from other storage systems, e.g. from the community data space to
your private data space, use ``tapis files import``:

.. code-block:: bash

   $ tapis files import agave://community-data/public_file.txt  /
                        agave://private-system/destination_folder/


With the above syntax, the file located at the root directory on the
``community-data`` storage system will be imported to your private storage
system, and placed in your directory ``destination_folder``.

Please also note that even though you are *able* to import files from other
Tapis storage systems, you may not always *need* to import those files. Most
applications of Tapis will allow you to provide the complete URI path to the
file, e.g. ``agave://community-data/public_file.txt``. This is useful, for
example, in the case of large reference libraries. Pointing to the remote
libraries rather than copying them saves time and disk space.


Import Files from the Web
-------------------------

You can also import files from the web using the URL. This is useful
to import files that are not part of an existing Tapis storage system:

.. code-block:: bash

   $ tapis files import https://website.com/raw_file \
                        agave://private-system/destination_folder
