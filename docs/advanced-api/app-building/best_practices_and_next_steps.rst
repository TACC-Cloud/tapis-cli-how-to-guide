Best Practices and Next Steps
=============================


Congratulations! You built and deployed your own Tapis. Consider the following
recommended best practices when building app bundles, and share your work with
others in the following ways:


Best Practices for Developing Containerized App Bundles
-------------------------------------------------------

The app development approach demonstrated here is meant to be *flexible*, in
that it can adapt to many different scientific applications and data sets, and
*scalable*, in that it can run efficiently on TACC peta-scale systems. Some
tips for the app developer in building new apps:

1. Write a clean Dockerfile so there is no question of source code / version provenance. Minimize image size where possible by removing, e.g. source code tarballs and installation directories.
2. Design a robust, but small and portable test case to package with the app bundle. Make liberal use of error checking in ``tester.sh`` and ``runner.sh``.
3. Use only command line arguments when calling the containerized executable (with the ``container_exec`` function). If the executable requires a configuration file, use a wrapper script inside the container to parse inputs from the command line and generate the appropriate configuration file.
4. Explicitly declare all inputs, and explicitly write all outputs. This includes file name and full path.
5. Package and curate outputs into a user-friendly format. Some use cases may benefit from a tarball of all output files; some use cases may benefit from individual files.
6. Make output file names deterministic and predictable to facilitate scripting and job chaining.
7. Document all expected outputs in the ``tester.sh`` and ``runner.sh`` wrapper scripts. Where appropriate, validate output and provide helpful error messaging.
8. Share your Docker images and app bundles with the SD2E community to benefit others and elicit feedback.

Best practices were adapted from the
`Computational Genomics Lab <https://toil.readthedocs.io/en/3.12.0/developingWorkflows/developing.html#best-practices-for-dockerizing-toil-workflows>`_.


Put Your App under Version Control
----------------------------------

As you iterate with new changes in your codebase, or as new versions of your
software are released, you will want to deploy updated copies of your app.
Keeping the app bundle directory under version control (e.g. with Git) enables
the developer to make incremental updates to the app version without losing any
past information.

Many Tapis tenants maintain organizations in Github or Gitlab. Please consult
with other members of your tenant and consider contributing your app bundle
repo to the organization. Doing so promotes transparency into the function of an
app, and enables others to build their own copies.


Considerations for Docker Images
--------------------------------

Some Tapis tenants maintain Docker Hub organizations. Consult with your tenant
admin to find out if it would be appropriate to push your image into the
organization name space. Doing so may give added benefits in terms of privacy of
sensitive code and provenance. In addition, always make sure to name your images
with a descriptive name, an tag appropriate to the version, and **LABEL** your
image with a maintainer.
