:orphan:

.. meta::
   :description: A guide to getting started with a nodejs-express app on Hasura
   :keywords: hasura, guide, nodejs, express, getting started
   :content-tags: getting started, nodejs, express

.. title:: Deploy a nodejs (Express) application on Hasura

.. rst-class:: guide-title
.. rubric:: Deploy a nodejs (Express) app on Hasura

Introduction
------------

This tutorial will help you build and deploy a nodejs app using Express and Hasura in minutes.

Benefits of using Hasura to deploy your nodejs app:

* Easy deployment with ``git push hasura master``
* Pre-configured Postgres that can be easily used through Data/Auth APIs.

To get started, you only need:

* git installed on your computer

That's it! You can now go ahead and get started with the next section!


Setup
-----

The first step involves installing and configuring the command line tool ``hasuractl``.

Install hasuractl on your favourite operating system:

Installation
^^^^^^^^^^^^

Linux
~~~~~

Install the latest ``hasuractl`` using the following command:

.. code:: bash

    $ curl -Lo hasuractl https://storage.googleapis.com/hasuractl/latest/linux-amd64/hasuractl && chmod +x hasuractl && sudo mv hasuractl /usr/local/bin/


If you would like to add hasuractl manually to your path drop the ``sudo mv hasuractl /usr/local/bin`` from the above command


Windows
~~~~~~~

Download `hasuractl.exe <https://storage.googleapis.com/hasuractl/latest/windows-amd64/hasuractl.exe>`_.
and place it in your ``PATH``. Refer to this `video <https://drive.google.com/file/d/0B_G1GgYOqazYUDJFcVhmNHE1UnM/view>`_
if you need help with the installation on Windows.

    In Windows, you should only use ``git-bash`` to execute commands that you see in this documentation.

Mac OS
~~~~~~

Run the following command to install ``hasuractl``:

.. code:: bash

    $ curl -Lo hasuractl https://storage.googleapis.com/hasuractl/latest/darwin-amd64/hasuractl && chmod +x hasuractl && sudo mv hasuractl /usr/local/bin/

If you would like to add hasuractl manually to your path drop the ``sudo mv hasuractl /usr/local/bin`` from the above command

Post Installation
^^^^^^^^^^^^^^^^^

Once ``hasuractl`` is installed, you will need to login to Hasura using:

.. code:: bash

   $ hasuractl login

This command will take you to your browser to perform login and authorization and return you once you are done.

Next, you can create a directory for your Hasura project using:

.. code:: bash

   $ mkdir project-name && cd project-name

Once you're in the directory you want to use for your project, create a free Hasura trial project using

.. code:: bash
   $ hasuractl init --type=trial

Make a note of the cluster name it displays at the end of the command, say cluster-name.

This command will

* Create the follwing directories in your project folder:

.. code::

   .
   ├── clusters
   ├── hasura.yaml
   ├── migrations
   └── services



* Create a free Hasura trial cluster, the name of the cluster is printed out at the end

* Add the trial cluster to the clusters folder in your project-name directory.

Once this is done, you can open the Hasura console with:

.. code:: bash

   $ hasuractl api-console

Explore the console, and try out the various Hasura APIs at the API Explorer!

Before moving on, let's initialize a git repository in our project folder in order to maintain version control, and to easily deploy using git push.

.. code:: bash

   $ git init

When you're ready to deploy your nodejs app, move on to the next section.

Deploy your app
---------------

In this section, we'll deploy a sample hello-world nodejs-express app on Hasura.

     If you already have an existing Express project that you wish to deploy on Hasura, you can read through this section to get an idea of what happens, and then check out the next section for instructions on deploying your app.

Use the following command to quickly add a sample(called app-name) nodejs app built on Express and set it up for deployment:

.. code:: bash

   $ hasuractl service quickstart app-name --template nodejs-express

This command will do the following:

* Create a folder called app-name inside the services directory and initialize it with a sample nodejs-express app

.. code::

  .
  ├── Dockerfile
  ├── README.md
  ├── app
  │   └── src
  │       ├── package.json
  │       └── server.js
  └── docker-config.yaml

* Configure your Hasura cluster to add a service for your app

* Add a route to your Hasura project at which your app will be live

* Add a git remote to you Hasura project so that you can quickly deploy your project

Once you have the quickstart directory ready, you should add and commit your code to get ready for deploying:

.. code:: bash

   $ git add . && git commit -m "Initialized"

Now deploy your sample nodejs-express app in one step using

.. code:: bash

   $ git push hasura master

Now check your app live at `https://app-name.cluster-name.hasura-app.io <`https://app-name.cluster-name.hasura-app.io>`_ !

This command will push your sample app to a git remote on your Hasura cluster, which then builds a Docker image out of it using the Dockerfile in the services/app-name folder, and put it up live at a subdomain on your cluster.

Deploy an existing nodejs app on Hasura
--------------------------------------

If you went through the previous section, you might recall the various things the ``hasuractl quickstart`` command did in order to set up your Hasura project for easy deployment:

.. code::

  * Create a folder called app-name inside the services directory and initialize it with a sample nodejs-express app

      .
      ├── dockerfile
      ├── readme.md
      ├── app
      │   └── src
      │       ├── package.json
      │       └── server.js
      └── docker-config.yaml

  * Configure your Hasura cluster to add a service for your app

  * Add a route to your Hasura project at which your app will be live

  * Add a git remote to you Hasura project so that you can quickly deploy your project

We will now perform the above steps separately to deploy our existing app on Hasura.

All apps on Hasura are deployed as services running on Docker containers. This means that to deploy our app, we'll have to create Dockerfile that builds the Docker container for us.

Once we have a Dockerfile, we'll need to configure our Hasura project to add a new service for your app, and set up our deployment flow.

We will also need to add to the project a route at which our app will go live.

To actually deploy using git, we'll need to add a git remote to push to.

Once all this configuration is done, we can push to our Hasura project using ``git push hasura master``

We will structure our code to look like:

.. code::

  .
  ├── dockerfile
  ├── readme.md
  ├── app
  │   └── src
  │       ├── package.json
  │       └── server.js
  └── docker-config.yaml

We will place our app source code in the services/app-name/app/src folder.

We will go through the parts of this setup step by step and understand how to
modify them.

So, without further ado, let's get to it!

Step 1: Setting up your Dockerfile
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  If you're new, all you need to know is that Dockerfiles are like recipes to
  build something called a Docker image, which is basically used as the base image
  for a container (Which is like a virtualbox, but lighter). These containers are
  an easy way to deploy and scale up services.

Let's first create an app-name directory in our services folder (Replace app-name with the name of your app.)

.. code:: bash

   $ mkdir services/app-name

Now, inside this folder, we'll add our Dockerfile.

You can use the following sample Dockerfile, or check out the `Writing your own Dockerfile` section below.

Copy the following and paste it in the services/app-name/Dockerfile file.

.. code::

  FROM mhart/alpine-node:7.6.0

  WORKDIR /src

  # Add package.json
  ADD app/src/package.json /src/package.json

  #install node modules
  RUN npm install

  #Add the source code
  ADD app/src /src

  CMD ["node", "server.js"]

This Dockerfile builds the src from app-name/app/src directory, and installs npm modules from the app-name/app/src/package.json

Step 2: Setting up your project source code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now we'll initalize a nodejs-express app in the src directory using a package.json file.

Put the following code in the services/app-name/app/src/package.json file:

.. code::

  {
    "name": "quickstart-docker-express-example",
    "version": "1.0.0",
    "description": "Demo to show git push build and deploy",
    "scripts": {
      "start": "node server.js"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
      "express": "^4.14.0"
    }
  }

Now add your server.js file here, which contains your main express code.

After all this, our services directory now looks like:

.. code::

  services
  └── app-name
      ├── Dockerfile
      └── app
          └── src
              ├── package.json
              ├── server.js
              ├── .....
              └── .....

With this, our project is now ready to get deployed.

Before we do anything however, let's test it locally to ensure
that it is properly configured.

Also remember to add a .gitignore file in the src directory

.. code::

   node_modules

From inside the src directory, run

.. code:: bash

   $ npm install
   $ npm start

This should run your nodejs app locally at `127.0.0.1:8080 <http://127.0.0.1:8080/>`_
Check that it works fine, and now let's get on to the next step!

Step 3: Configuring the Hasura project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We'll generate basic configuration for our service using:

.. code:: bash

   $ hasuractl service add app-name -c hasura

This command will create the service configuration in the clusters/cluster-name directory.

Step 4: Adding a route
^^^^^^^^^^^^^^^^^^^^^^

Now we'll add a route at which our app will live:

.. code:: bash

   $ hasuractl routes add app-name -c hasura

This command adds a route that sets your app up at a subdomain (app-name) of
your Hasura project.

Step 5: Configure a remote
^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that our service is setup, we'll need to setup our remote for easy deployment.
The idea is to deploy the app on git push, so we'll need to first configure a
remote on our cluster

.. code:: bash

   $ hasuractl remotes add -s app-name -c hasura

This command configures our project to set up git push deployment for
the app-name service.

It will also tell you the next few steps to deploy your app, one of which is adding a git remote.

.. code:: bash

   $ git remote add hasura-app-name ssh://hasura@cluster-name.hasura-app.io/~/git/hasura-app-name/

(Make sure you replace app-name and cluster-name in the above command, or use the
proper command from the output of remotes add above)

Step 6: Apply the configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now, we need to apply all these configuration changes to our cluster, so do

.. code:: bash

   $ hasuractl cluster apply -c hasura

Once this is done, we've finished configuring our app, so let's deploy it!

Step 7: Deploying the app
^^^^^^^^^^^^^^^^^^^^^^^^^

First we add and commit our code from the main Hasura project directory (where we did a ``git init`` earlier) :

.. code:: bash

   $ git add .
   $ git commit -m "Initialized"

Now we deploy our code with a

.. code:: bash

   $ git push hasura master

Now check your app live at `https://app-name.cluster-name.hasura-app.io <`https://app-name.cluster-name.hasura-app.io>`_ !

This command will push your sample app to a git remote on your Hasura cluster, which then builds a Docker image out of it using the Dockerfile in the services/app-name folder, and put it up live at a subdomain on your cluster.

Run your app locally
--------------------

If you're using the sample app we created earlier, you can
just run it locally from the services/app-name/app/src folder using

.. code:: bash

   $ npm install
   $ npm start

This should run your nodejs app locally at `127.0.0.1:8080 <http://127.0.0.1:8080/>`_

If you want to run your own app in the hasura service directory locally,
follow the instructions in Step 2 of the `Deploy your existing nodejs app on Hasura` section above.

Writing your own custom Dockerfile
----------------------------------

Let's start by looking at a sample Dockerfile.

.. code::

  FROM mhart/alpine-node:7.6.0

  WORKDIR /src

  # Add package.json
  ADD app/src/package.json /src/package.json

  #install node modules
  RUN npm install

  #Add the source code
  ADD app/src /src

  CMD ["node", "server.js"]

Let's go through this step by step to understand what it does, and how it can be modified.

.. code::

  FROM mhart/alpine-node:7.6.0

This line tells Docker to pull the alpine-node:7.6.0 base image from
the mhart repository on Dockerhub.

.. code::

  WORKDIR /src

  # Add package.json
  ADD app/src/package.json /src/package.json

  #install node modules
  RUN npm install

The WORKDIR command sets the current working directory for the RUN commands
in the Dockerfile. In this case, we are going to deploy our app from the
/src folder of our Dockerfile. This is unrelated to the corresponding
directory on your computer.

The ADD command above picks up the requirements.txt from the src
and puts it in /src, and the RUN command uses npm to install all the
dependencies for your project.

If your package.json is at a different location in the services/app-name/app folder,
you can edit the ADD command to point it to the right location.

.. code::

  #Add the source code
  ADD app/src /src

  CMD ["node", "server.js"]

Now we ADD our source code to the /src directory of the Docker image,
and use our npm to run our nodejs server on the container.

Working with a frontend
-----------------------

Hasura makes working with a frontend very easy with its various Data, Auth and Filestore APIs.
These APIs are designed to be directly called from the frontend via JSON requests, so that
you can avoid writing boilerplate authentication or data access code in your backend.

As for adding a frontend to your nodejs app, you can just add it to your source code as
you would for a normal nodejs app, and deploy using the instructions above.
You can check out the `Hasura manual <https://docs.hasura.io/0.14/manual/index.html>`_ for more info on integrating with a frontend.

Performing Data Operations
--------------------------

To access data from the frontend, you can directly contact the Hasura APIs with JSON requests, instead of
making a request to your nodejs backend and then communicating with the Data APIs.

To query the Data APIs from your nodejs backend however, it is as easy as making a query to the internal data endpoint using fetch.

To learn more about the Data APIs, check out the `Hasura manual <https://docs.hasura.io/0.14/manual/index.html>`_

First install the node-fetch module:

.. code:: bash

   $ npm install --save node-fetch

This will also add it your package.json so that it works as a dependency.

You can make requests to either the external or the internal endpoints.

.. code:: bash

  var fetch =  require('fetch');

  var url = "https://data.cluster-name.hasura-app.io/v1/query"; // Replace cluster-name with your cluster

  var requestOptions = {
      "method": "POST",
      "headers": {
          "Authorization": "Bearer <Admin-Token>",
          "Content-Type": "application/json"
      },
      "body": {
          "type": "select",
          "args": {
              "table": "authors",
              "columns": [
                  "*"
              ]
          }
      }
  };

  fetch(url, requestOptions)
  .then(function(response) {
    return response.json();
  })
  .then(function(result) {
    console.log(result);
  })
  .catch(function(error) {
    console.log('Request Failed:' + error);
  });


You can use this to make any requests to the Data APIs from your nodejs app.

Implementing Auth
-----------------

Hasura's auth system is extremely simple to use.  You simply have to call the auth endpoints from
your frontend to signup, login and logout. These API calls will automatically set a cookie and maintain
your session for you.

The Hasura auth system has three default roles:

* **Admin** : This role is for admin users

* **User** : This role is for all logged in users, all admin users are Users.

* **Anonymous** : This role represents clients that are not logged in.

To check if a user is logged in, and to get his hasura_id, you can do the following:

.. code::

   req.get('X-Hasura-User-Id') // If not anonymous, User is logged in.

   if( req.get('X-Hasura-Role') != 'anonymous'){
       console.log('User is logged in!');
       console.log(req.get('X-Hasura-User-Id'));
   } else {
       console.log('User is not logged in.');
   }

This makes authentication extremely simple.
For more information on using the Hasura Auth APIs, check the  `Hasura manual <https://docs.hasura.io/0.14/manual/index.html>`_ .
