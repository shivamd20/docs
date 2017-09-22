:orphan:

.. meta::
   :description: A guide to getting started with a Python(Flask) app on Hasura
   :keywords: hasura, guide, python, flask, getting started
   :content-tags: getting started, python, flask

.. title:: Deploy a Python (Flask) application on Hasura

.. rst-class:: guide-title
.. rubric:: Deploy a Python (Flask) app on Hasura

.. role:: python(code)
   :language: python

Introduction
------------

This tutorial will help you build and deploy a Python app using Flask and Hasura in minutes.

Benefits of using Hasura to deploy your Flask app:

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

   $ hasuractl console -c cluster-name

Explore the console, and try out the various Hasura APIs at the API Explorer!

Before moving on, let's initialize a git repository in our project folder in order to maintain version control, and to easily deploy using git push.

.. code:: bash

   $ git init

When you're ready to deploy your python app, move on to the next section.

Deploy your app
---------------

In this section, we'll deploy a sample hello-world Flask app on Hasura.

     If you already have an existing Flask project that you wish to deploy on Hasura, you can read through this section to get an idea of what happens, and then check out the next section for instructions on deploying your app.

Use the following command to quickly add a sample(called app-name) Python app built on Flask and set it up for deployment:

.. code:: bash

   $ hasuractl service quickstart app-name --template python-flask -c cluster-name

This command will do the following:

* Create a folder called app-name inside the services directory and initialize it with a sample python-flask app

.. code::

    .
    ├── Dockerfile
    ├── README.md
    ├── app
    │   ├── conf
    │   │   └── gunicorn_config.py
    │   └── src
    │       ├── __init__.py
    │       ├── requirements.txt
    │       └── server.py
    └── docker-config.yaml

* Configure your Hasura cluster to add a service for your app

* Add a route to your Hasura project at which your app will be live

* Add a git remote to you Hasura project so that you can quickly deploy your project

Once you have the quickstart directory ready, you should add and commit your code to get ready for deploying:

.. code:: bash

   $ git add . && git commit -m "Initialized"

Now deploy your sample python-flask app in one step using

.. code:: bash

   $ git push hasura master

Now check your app live at `https://app-name.cluster-name.hasura-app.io <`https://app-name.cluster-name.hasura-app.io>`_ !

This command will push your sample app to a git remote on your Hasura cluster, which then builds a Docker image out of it using the Dockerfile in the services/app-name folder, and put it up live at a subdomain on your cluster.

Deploy an existing Flask app on Hasura
--------------------------------------

If you went through the previous section, you might recall the various things the ``hasuractl quickstart`` command did in order to set up your Hasura project for easy deployment:

.. code::

  * Create a folder called app-name inside the services directory and initialize it with a sample python-flask app

      .
      ├── Dockerfile
      ├── README.md
      ├── app
      │   ├── conf
      │   │   └── gunicorn_config.py
      │   └── src
      │       ├── __init__.py
      │       ├── requirements.txt
      │       └── server.py
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

  services
  └── app-name
      ├── Dockerfile
      └── app
          ├── conf
          │   └── gunicorn_config.py
          └── src
              ├── __init__.py
              ├── requirements.txt
              └── server.py

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

  FROM python:3.5.2-alpine

  WORKDIR /usr/src/app

  # install requirements
  # this way when you build you won't need to install again

  COPY app/src/requirements.txt /tmp/requirements.txt
  RUN pip3 install -r /tmp/requirements.txt

  COPY app /usr/src/app

  # App port number is configured through the gunicorn config file
  CMD ["gunicorn", "--config", "./conf/gunicorn_config.py", "src:app"]


Let's also create the conf/gunicorn_config.py file and paste the follwing into it:

.. code:: python

  import os
  import multiprocessing

  bind = "0.0.0.0:8080"
  workers = (multiprocessing.cpu_count() * 2) + 1
  accesslog = "-"
  access_log_format = '%(h)s %(l)s %(u)s %(t)s "%(r)s" %(s)s %(b)s "%(f)s" "%(a)s"'
  loglevel = "debug"
  capture_output = True
  enable_stdio_inheritance = True

This is a basic configuration file for our gunicorn server that
runs on 0.0.0.0 (the server) on port 8080 (Default Hasura port for
custom services)

``gunicorn``, which is now running from the WORKDIR (/usr/src/app, if you remember
from earlier in the Dockerfile), is expecting your __init__.py directory at src/__init__.py.
You can change this by changing the app module (the "src:app" part in the CMD command in the Dockerfile).

( Don't forget to add gunicorn to your requirements.txt file!)

Step 2: Setting up your project source code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now we'll initalize a Flask app in the src directory using an __init__.py file.

Put the following code in the services/app-name/app/src/__init__.py file:

.. code::

  from flask import flask

  app = flask(__name__)

  from .server import *

With this, the src folder can now be imported as a module, so place all your source code inside this folder.

In your main Flask app initialization file (say server.py) , instead of importing flask and setting :python:`app = flask(__name__)`, just import the
src package you created using:

.. code::

   from src import app

After all this, our services directory now looks like:

.. code::

  services
  └── app-name
      ├── Dockerfile
      └── app
          ├── conf
          │   └── gunicorn_config.py
          └── src
              ├── __init__.py
              ├── requirements.txt
              ├── server.py
              ├── .....
              └── .....

With this, our project is now ready to get deployed.

Before we do anything however, let's test it locally to ensure
that it is properly configured.

From inside the src directory, run

.. code:: bash

   $ FLASK_APP=__init__.py flask run

This should run your flask app locally at `127.0.0.1:5000 <http://127.0.0.1:5000/>`_
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

   $ FLASK_APP=__init__.py flask run

This should run your flask app locally at `127.0.0.1:5000 <http://127.0.0.1:5000/>`_

If you want to run your own app in the hasura service directory locally,
follow the instructions in Step 2 of the `Deploy your existing Flask app on Hasura` section above.

Writing your own custom Dockerfile
----------------------------------

Let's start by looking at a sample Dockerfile.

.. code::

  FROM python:3.5.2-alpine

  WORKDIR /usr/src/app

  # install requirements
  # this way when you build you won't need to install again

  COPY app/src/requirements.txt /tmp/requirements.txt
  RUN pip3 install -r /tmp/requirements.txt

  COPY app /usr/src/app

  # App port number is configured through the gunicorn config file
  CMD ["gunicorn", "--config", "./conf/gunicorn_config.py", "src:app"]

Let's go through this step by step to understand what it does, and how it can be modified.

.. code::

   FROM python:3.5.2-alpine

This line tells Docker to pull the 3.5.2-alpine base image from
the python repository on Dockerhub.

.. code::

  WORKDIR /usr/src/app

  # install requirements
  # this way when you build you won't need to install again
  # and since COPY is cached we don't need to wait
  COPY app/src/requirements.txt /tmp/requirements.txt
  RUN pip3 install -r /tmp/requirements.txt

The WORKDIR command sets the current working directory for the RUN commands
in the Dockerfile. In this case, we are going to deploy our app from the
/usr/src/app folder of our Dockerfile. This is unrelated to the corresponding
directory on your computer.

The COPY command above picks up the requirements.txt from the src
and puts it in /tmp, and the RUN command uses pip to install all the
dependencies for your project.

If your requirements.txt is at a different location in the services/app-name/app folder,
you can edit the COPY command to point it to the right location.

.. code::

  COPY app /usr/src/app

  # App port number is configured through the gunicorn config file
  CMD ["gunicorn", "--config", "./conf/gunicorn_config.py", "src:app"]

Now we COPY our source code to the /usr/src/app directory of the Docker image,
and use our gunicorn to run our Flask server on the container, which will
start listening as per the gunicorn_config.py file.
( Check out Step 1 of the `Deploying an existing Flask app on Hasura` section for a sample gunicorn_config.py file)

Working with a frontend
-----------------------

Hasura makes working with a frontend very easy with its various Data, Auth and Filestore APIs.
These APIs are designed to be directly called from the frontend via JSON requests, so that
you can avoid writing boilerplate authentication or data access code in your backend.

As for adding a frontend to your flask app, you can just add it to your source code as
you would for a normal Flask app, and deploy using the instructions above.
You can check out the `Hasura manual <https://docs.hasura.io/0.14/manual/index.html>`_ for more info on integrating with a frontend.

Performing Data Operations
--------------------------

To access data from the frontend, you can directly contact the Hasura APIs with JSON requests, instead of
making a request to your flask app and then communicating with the Data APIs.

To query the Data APIs from your Flask app however, it is as easy as making a query to the internal data endpoint using requests.

To learn more about the Data APIs, check out the `Hasura manual <https://docs.hasura.io/0.14/manual/index.html>`_

You can make requests to either the external or the internal endpoints.
To make a request to the external requests using the requests library:

.. code:: bash

  import requests
  import json

  dataUrl = "https://data.cluster-name.hasura-app.io/v1/query" # Replace cluster name with your cluster
  # To make requests to the internal data endpoint
  # dataUrl = "http://data.hasura/v1/query"
  query = {
      "type": "select",
      "args": {
          "table": "authors",
          "columns": [
              "*"
          ]
      }
  }

  headers = { "Content-Type":"application/json","Authorization":"Bearer <Your admin token here>"}
  # For the internal endpoint, headers should be
  # headers = { "Content-Type":"application/json", "X-Hasura-User-Id":"1", "X-Hasura-Role":"admin"}
  # The internal endpoint does not need an auth token, so it is usually preferred. But it can only
  # be accessed from inside the project.

  response = requests.post( dataUrl, data=json.dumps(query), headers = headers)

  print(response.text)
  # This will contain a table does not exist error if the authors table does not exist
  # A json response with a list of author objects if the table exists

  # Sample response with name and id columns
  #  u'[{"name":"Alice Cooper","id":1}, \n {"name":"John Smith","id":2}]'

  authorList = json.loads(response.text)
  author1 = authorList[0]['name']
  print(author1)    # Prints "Alice Cooper"


You can use this to make any requests to the Data APIs from your Flask app.

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

   from flask import request

   if request.headers['X-Hasura-Role'] != 'anonymous':
       print("User is logged in!")
       print("User id is %s" % request.headers['X-Hasura-User-Id']) # Prints the hasura_id of the user
   else:
       print("User is not logged in")


This makes authentication extremely simple.
For more information on using the Hasura Auth APIs, check the  `Hasura manual <https://docs.hasura.io/0.14/manual/index.html>`_ .
