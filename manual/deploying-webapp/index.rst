.. _deploy-webapp:

Deploying your webapp on Hasura
===============================

Hasura provides a fast and simple way to deploy your app built on your favourite
frameworks as a service on a secure https subdomain. To deploy your code on
Hasura, all you need to do is a ``git push hasura master``!

To set up this simple git push deployment system, you need the following:

* Your app code in a git repository
* A Dockerfile that contains instructions on building a Docker image for your app
* A git-push deployment enabled service on Hasura

There are two ways of doing this:

* Use the Hasura Quickstart Templates that have "hello world" for a various framesworks
* Use your own Dockerfile


Option 1: Using the Quickstart Templates
----------------------------------------

Apps or Services deployed on Hasura run on Docker images, built according to a
Dockerfile. We've prepared `starter kits <https://github.com/hasura/quickstart-docker-git>`_ for all your favourite
frameworks, that already contain pre-configured Dockerfiles for you to quickly
setup your app!

Step 1: Install `hasuractl` and setup your Hasura project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The easiest way to use these templates is to install and set your project
context on Hasuractl as shown in :doc:`../../ref/cli/hasuractl`.

Before you continue, remember to login and set-context so that `hasuractl` knows the name
of your hasura project.

.. code-block:: console

   $ hasuractl login
   $ mkdir project-name && cd project-name
   $ hasuractl init --type=trial

This will initialize a Hasura project in the project-name folder, create a free Hasura cluster and add it to the project.

Step 3: Initialise your app folder and git repo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Choose a template, and use the following command to set up a service for our app
called <app-name>

.. code-block:: console

    $ hasuractl quickstart <app-name> -t <template-name>

This command will do the following:

* Create a service hosted at `my-app.<project-name>.hasura-app.io`, to which you can deploy your app

* Create routes for the service on your cluster

* Create a remote on the cluster to push your app to

* Add a git remote called hasura, which integrates with the remote on the cluster so that ``git push hasura master`` will deploy your app.

* Copy a hello world app written in the chosen framework (`python-flask`) into the services/<app-name> (`my-app`) directory

Now, `cd` into the folder, commit your code, and get ready to deploy!

.. code-block:: console

    $ cd <app-name>
    $ git commit -am "Initialized"

Step 4: Add your SSH key
^^^^^^^^^^^^^^^^^^^^^^^^
Make sure to add your ssh-key to your Hasura project before you deploy:

.. code-block:: console

    $ cd ~/.ssh/id_rsa.pub > clusters/hasura/authorized_keys
    $ hasuractl cluster apply -c hasura

This will add your key to the authorized_keys folder on the `hasura` cluster.

Step 5: Deploy your app
^^^^^^^^^^^^^^^^^^^^^^^
Now, we deploy our app using:

.. code-block:: console

    $ git push hasura master

Voila, your service is deployed and live! Check out your service live at <app-name>.<project-name>.hasura-app.io!

In case there are any errors in building or deploying your code, the git push command will show you errors and the push will fail. Fix the error, and push again!

.. admonition:: Behind The Scenes

   The Hasura platform basically builds a docker image from the latest git changes
   pushed by you, and deploys the right kubernetes service, deployment underneath.

   If you want finer control over your deployment, you are encouraged to use ``kubectl``
   and peek under the hood of the service that is automatically deployed.


Option 2: Using your own Dockerfile (advanced users)
------------------------------------

Create a git-push enabled service on the Hasura console
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To add a git push service, follow the steps below:

First, we add a service

.. code:: bash

   $ hasuractl service add my-app -c hasura

Now we add routes to expose our service

.. code:: bash

   $ hasuractl routes add my-app -c hasura

Next we add a remote

.. code:: bash

   $ hasuractl remotes add -s my-app -c hasura

Finally, apply the configuration

.. code:: bash

   $ hasuractl cluster apply -c hasura

Now we add code to the services/my-app folder, with the Dockerfile at services/my-app/Dockerfile, and our service is ready to push.

Add your SSH key
^^^^^^^^^^^^^^^^^^^

Make sure to add your ssh-key to your Hasura project before you deploy:

.. code-block:: console

    $ cd ~/.ssh/id_rsa.pub > clusters/hasura/authorized_keys
    $ hasuractl cluster apply -c hasura

This will add your key to the authorized_keys folder on the `hasura` cluster.


Deploy to your git-push enabled service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once the above setup is done, you can commit and deploy with:

.. code:: bash

   $ git commit -am "Init"
   $ git remote add hasura-my-app ssh://hasura@my-app.<cluster-name>.hasura-app.io/~/git/my-app/
   $ git push hasura-my-app master

Voila, your app will now be live at https://my-app.<cluster-name>.hasura-app.io !

In case there are any errors in building or deploying your code, the git push command will show you errors and the push will fail. Fix the error, and push again!
