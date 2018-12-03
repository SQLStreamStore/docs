#Postgres Storage

The database schema for the Postgres SQL Stream Store driver
consists of two tables, a custom data type, and multiple
functions.

# Database Schema Initialization

Database initialization can be accomplished in one of two
ways: from the library itself or manually.

Initializing the database via the library is relatively
straight forward:

```
var settings = new PostgresStreamStoreSettings("Host=localhost;Port=5432;User Id=postgres;Database=yourdatabase");

using (var streamStore = new PostgresStreamStore(settings))
{
  await streamStore.CreateSchema();
}
```

This will create the Postgres SQL Stream Store database
objects under the schema `public`.

The other way is to manually run each of the scripts
found here - starting with `Tables.sql` and excluding
`DropAll.sql`. Be careful to replace the string
`__schema__` in each of the sql scripts with your schema
(defaults to `public`).

# Settings

## Constructor

Takes a connection string.

## CreateStreamNotifier

Allows overriding of the stream store notifier.

## Schema

The schema SQL Stream Store should place database objects
into. Defaults to `public`.

## ExplainAnalyze

Loads the `auto_explain` module and turns it on for all
queries. Useful for index tuning. Defaults to `false`;

## GetUtcNow

A delegate to return the current UTC now. Used in testing
to control timestamps and time related operations. If not
set, the database server will control the timestamp.

## LogName

The log name used for any of the log messages.

## ScavengeAsynchronously

Allows setting whether or not deleting expired (i.e., older
than maxCount) messages happens in the same database
transaction as append to stream or not.

This does not effect scavenging when setting a stream's metadata
- it is always run in the same transaction.

## ConnectionFactory

Allows overriding the way a `NpgsqlConnection` is created
given a connection string. 

The default implementation simply passes the connection string
into the `NpgsqlConnection` constructor.

