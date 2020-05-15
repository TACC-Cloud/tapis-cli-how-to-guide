Initialize a Session
====================

You must set up a Tapis session on each host where you will use the Tapis CLI.
This is a scripted process implemented by the command :code:`tapis auth init`.
This is a one-time operation where you will be asked to agree to terms, select
a tenant, and finally enter a username and password for that tenant.


.. code-block:: text

   $ tapis auth init

   Use of Tapis requires acceptance of the TACC Acceptable Use Policy
   which can be found at https://portal.tacc.utexas.edu/tacc-usage-policy

   Do you agree to abide by this AUP? (type 'y' or 'n' then Return) y

   Use of Tapis requires acceptance of the Tapis Project Code of Conduct
   which can be found at https://tapis-project.org/code-conduct

   Do you agree to abide by this CoC? (type 'y' or 'n' then Return) y

   To improve our ability to support Tapis and the Tapis CLI, we would like to
   collect your IP address, operating system and Python version. No personally-
   identifiable information will be collected. This data will only be shared in
   aggregate form with funders and Tapis platform stakeholders.

   Do you consent to this reporting? [Y/n]: Y
   +---------------+--------------------------------------+----------------------------------------+
   |      Name     |             Description              |                  URL                   |
   +---------------+--------------------------------------+----------------------------------------+
   |      3dem     |             3dem Tenant              |         https://api.3dem.org/          |
   |   agave.prod  |         Agave Public Tenant          |      https://public.agaveapi.co/       |
   |  araport.org  |               Araport                |        https://api.araport.org/        |
   |     bridge    |                Bridge                |     https://api.bridge.tacc.cloud/     |
   |   designsafe  |              DesignSafe              |    https://agave.designsafe-ci.org/    |
   |  iplantc.org  |         CyVerse Science APIs         |       https://agave.iplantc.org/       |
   |      irec     |              iReceptor               | https://irec.tenants.prod.tacc.cloud/  |
   |    portals    |            Portals Tenant            |  https://portals-api.tacc.utexas.edu/  |
   |      sd2e     |             SD2E Tenant              |         https://api.sd2e.org/          |
   |      sgci     | Science Gateways Community Institute |        https://sgci.tacc.cloud/        |
   |   tacc.prod   |                 TACC                 |      https://api.tacc.utexas.edu/      |
   | vdjserver.org |              VDJ Server              | https://vdj-agave-api.tacc.utexas.edu/ |
   +---------------+--------------------------------------+----------------------------------------+
   Enter a tenant name [tacc.prod]:
   tacc.prod username: taccuser
   tacc.prod password for taccuser:


Session tokens and other metadata are stored in ``~/.agave/config.json`` as well
as in ``~/.env``.
