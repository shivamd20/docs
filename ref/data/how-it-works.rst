How it works
============

The data service on Hasura exposes an HTTP/JSON API over a PostgreSQL database.
This API is designed to be used by any web client (speaking JSON), especially
frontend interfaces like Android and iOS apps, browser based JavaScript apps
etc.

The data API provides the following features:

1. CRUD APIs on PostgreSQL tables with a MongoDB-esque JSON query syntax.
2. Rich query syntax for making complicated select queries.
3. Role based access control at row and column level.

Internally, it works by converting a JSON query into an SQL statement,
executing the SQL statement on Postgres, and then converting the results back
into JSON format and sending it back to the client.

