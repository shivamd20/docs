SQL
=====

.. _run_sql:

run_sql
--------

Syntax
^^^^^^

.. list-table::
   :header-rows: 1

   * - Key
     - Required
     - Schema
     - Description
   * - sql
     - true
     - String
     - The sql to be executed
   * - cascade
     - false
     - Boolean
     - When set to ``true``, the effect (if possible) is cascaded to any hasuradb dependent objects (relationships, permissions, templates).

Description
^^^^^^^^^^^

``run_sql`` is used to execute arbitrary SQL statements. Multiple SQL statements can be separated by a ``;``. Currently, the result of these statements is not returned. So, this can only be used for DDL statements.

When the SQL that is executed affects the hasuradb objects (like a drop column or drop table), an error is returned instead of leaving the state in an inconsistent state. However, if you would like to drop the dependent objects, you can set ``cascade`` to ``true``. The SQL operations that will affect the hasuradb objects are

1. Dropping columns
2. Dropping tables
3. Altering types of columns

In case of 1 and 2, the dependent objects (if any) can be dropped using ``cascade``. However, when altering type, if any objects are affected, the change cannot be cascaded. So, those dependent objects have to be manually dropped before the sql statement.

.. note::
   Currently, renames of tables and columns are not allowed in the SQL statement.

Example
&&&&&&&

1. Create the table ``author`` and add a select permission for ``anonymous`` role.

.. code-block:: http

   POST /v1/query HTTP/1.1
   Content-Type: application/json
   Authorization: Bearer <admin-token>

   {
       "type": "bulk",
       "args": [
           {
               "type": "run_sql",
               "args": {
                   "sql": "CREATE TABLE author (id integer CHECK (id > 0), name text)"
               }
           },
           {
               "type": "create_select_permisison",
               "args": {
                   "table" : "author",
                   "role" : "anonymous",
                   "permission" : {
                       "columns" : ["id", "name"],
                       "filter" : {}
                   }
               }
           }
       ]
   }

2. Drop the column ``name``.

.. code-block:: http

   POST /v1/query HTTP/1.1
   Content-Type: application/json
   Authorization: Bearer <admin-token>

   {
       "type": "run_sql",
       "args": {
           "sql": "ALTER TABLE author DROP COLUMN name"
       }
   }

Because the ``select`` permission for ``anonymous`` role is dependent on the column ``name``, the above query throws an error. You can now drop the dependent objects (permission author.anonymous.select) by setting ``cascade`` to `true`.

.. code-block:: http

   POST /v1/query HTTP/1.1
   Content-Type: application/json
   Authorization: Bearer <admin-token>

   {
       "type": "run_sql",
       "args": {
           "sql": "ALTER TABLE author DROP COLUMN name",
           "cascade" : true
       }
   }
