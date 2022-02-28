# SQL Server Storage

For operational reasons, you should familiarize yourself with the underlying
database schema. The schema is implemented as two tables, `Streams` and
`Messages`, and a number of indexes.

The `Streams` table contains the collection of unique streams in the store. As
stream Ids are strings there are a number of optimisations applied:

1. The original stream Id is stored in column `IdOriginal` and limited to 1000
   characters.

2. When a stream is appended for first time is it checked to see if it is
   parsable as a `Guid`. If so, then that is stored in the `Id` column. If not
   then a `Sha1` hash of the Id is generated and used. This helps with stream
   lookups and the Id unique index constraint.

3. An `IdInternal` identity column is used for joins with the messages table.

Please refer to `CreateSchema.sql` for full schema details.

## Initializing the Database

The SqlStreamStore database can be initialized in a few ways:

* **Via SQL Script**: In the project's [`ScriptsV3` folder](https://github.com/SQLStreamStore/SQLStreamStore/tree/master/src/SqlStreamStore.MsSql/ScriptsV3), you'll find [a `CreateSchema.sql` script](https://github.com/SQLStreamStore/SQLStreamStore/blob/master/src/SqlStreamStore.MsSql/ScriptsV3/CreateSchema.sql). Executing the SQL script within a database will install the schema.
* **Via a method**: on the object for a given store, you can call `CreateSchemaIfNotExists()`. This will ensure the schema is created

## Checking Whether the Schema Matches

On the object for a store, you can call `CheckSchema()` which will return a result on whether the schema matches expectations.

This is ideal to run on startup / health checks

## Migrating from V2 Schema to V3

* **Via SQL Script**: In the project's [`ScriptsV3` folder](https://github.com/SQLStreamStore/SQLStreamStore/tree/master/src/SqlStreamStore.MsSql/ScriptsV3), you'll find [a `Migration_v3.sql` script](https://github.com/SQLStreamStore/SQLStreamStore/blob/master/src/SqlStreamStore.MsSql/ScriptsV3/Migration_v3.sql). Executing the SQL script within a database will migrate the schema from v2 to v3.
* **Via method**: On the object for a store, you can call `Migrate()` which will migrate the schema.

## Using

TODO
