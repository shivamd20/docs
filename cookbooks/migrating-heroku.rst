Migrating from Heroku

1. Database Migration
2. App Migration
3. Addon Migration
4. Adding Collaborators
5. Custom Domain


1. DATABASE MIGRATION (HEROKU POSTGRESQL TO HASURA POSTGRESQL)

If you have a PostgreSQL database running on Heroku, follow the steps below. If you are using some other data store, you need to manage it as a custom service in Hasura. (More on this later)

-Export SQL Dump from Heroku

Get your heroku-postgresql credentials.  via this link - https://data.heroku.com/?filter=heroku-postgresql. Select your app's corresponding postgresql datastore. Under Administration->Database Credentials->Click on View Credentials.

Now that you have the database credentials ready, lets take a pg_dump.
The format of the pg_dump command will be:

pg_dump --verbose --host=<host_name> --port=<port> --username=<username> --password --dbname=<dbname> > heroku-db.sql

(Should the above one be data-only?)

Replace <host_name>, <port>, <username>, <dbname> with values of your heroku-postgresql credentials. You will be prompted to enter the database password. 

-Import SQL Dump into Hasura

To access Postgres in your Hasura project, first you have to create SSH tunnel. For example:

$ ssh -p 22 -L 5432:postgres.default:5432 hasura@ssh.test42.hasura-app.io

This will create a SSH tunnel from your local machine to ssh.test42.hasura-app.io; and will forward your machine’s port 5432 to postgres.default:5432. The postgres.default:5432 is the internal hostname + port of the Postgres instance (as seen by the ssh.test42.hasura-app.io service).

Once the tunnel is created, then you can run psql as if you are connecting to your local machine’s Postgres.

psql -h localhost -p 5432 -U admin -d hasuradb -f heroku-db.sql

Note: Database name needs to be hasuradb while importing to hasura

2. APP MIGRATION (HEROKU APP TO HASURA)

Consider each app in Heroku as a custom service on Hasura. 

- PRE-REQUISITIES OF HASURA CUSTOM SERVICE:
    - Fetch the environment variables of your Heroku App.

        - Using cURL to fetch environment variables
            curl -XGET https://api.heroku.com/apps/<app-name>/config-vars -H "Accept: application/vnd.heroku+json; version=3" -H "Authorization: Bearer <auth-token>"

        Note: You need the auth-token to make the query:
        Fetch your auth token using heroku-cli "heroku auth:token" or by opening the netrc file "vi ~/.netrc" and copy the password corresponding to api.heroku.com

        - Using heroku-cli to fetch environment variables
            heroku config --app=<app-name>

        - Using the browser to fetch environment variables
            https://dashboard.heroku.com/apps/<app-name>/settings. Click on reveal Config Vars button

    - Create Dockerfile of your heroku-app.

        Now on your machine, navigate to your Heroku app's source code. (the folder containing your Procfile or app.json). In the top level directory add a Dockerfile, if you don't have one. (Refer this repo for Dockerfile formats of popular languages and frameworks - https://github.com/hasura/quickstart-docker-git)

- Deploy your heroku app on Hasura using Git
 
Create a new custom service on your Hasura Project with git push "Enable"d.
<PUT IMAGE HERE>

Add the hasura remote of your custom service that you created with git push:
git remote add hasura ssh://hasura@<endpoint-name>.<project-name>.hasura-app.io/~/git/<endpoint-name>/
git add .
git commit -m "Heroku app migrated to Hasura"
git push hasura master

Now you have deployed your Heroku app on Hasura using git push flow.

- Deploy using Docker

Now that you have Dockerfile in your parent directory of your app, you can use:
docker build -t <your-hasura-app>:<tag> .
docker push <your-hasura-app>:<tag>

Once the image is pushed to DockerHub, use the image name in Add custom service page of your Hasura Project. Input the appropriate environment variables that you fetched from Heroku for your app while creating the custom service.

3. ADDON MIGRATION (HEROKU ADDONS)

To be added

4. ADDING COLLABORATORS

You can add collaborators to your Hasura Project by creating them on Auth database.

5. Custom Domain

To be added

THINGS YOU DON'T NEED TO WORRY ABOUT:
- SSL Certificates (Hasura provides them by default for your project)
- Multi Dyno Setup (If you have multiple dynos in your Heroku, you don't need to worry about. Just select the infrastructure of your choice)

NOTES:
- By default, Hasura expects "git push" deployed services to run on 8080. Change the PORT on the Hasura UI, according to the PORT your services needs to use.
<PUT IMAGE HERE>
- If you are using any of the heroku-cli commands given above, you should be authenticated. Refer this article on https://devcenter.heroku.com/articles/authentication
