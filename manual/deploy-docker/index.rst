.. meta::
   :description: How to deploy docker images using hasura
   :keywords: hasura, manual, docker, image, custom service

===========================================
Deploying custom services using Docker images
===========================================

Not all requirements for your app may be met by the Hasura data, auth APIs.
Custom APIs and services will always need to be added specifically to the project.

If the service needed to be run has a ``docker`` image, Hasura provides an easy way to deploy the service by simply specifying the ``docker`` image.

**Example: Adding a custom database browser (adminer) using its Docker image**

.. code:: bash

   $ hasuractl service add adminer -c hasura -i clue/adminer -p 80

This will create a service called adminer that will pull the clue/adminer docker image and serve it on port 80 of https://adminer.cluster-name.hasura-app.io

.. admonition:: Automatic SSL certificates

   The Hasura platform automatically creates Grade A SSL certificates using LetsEncrypt.

   SSL certificate creation can take a few minutes. During this time ``https://adminer.<project-name>.hasura-app.io``
   will not served, and you'll have to access your service on ``http`` instead. As soon as
   the certificate is ready, ``http://adminer.<project-name>.hasura-app.io`` will automatically
   start redirecting to the ``https`` version.
