# MySql Storage

The database schema for the MySql Stream Store driver
consists of two tables, and multiple stored procedures.

# Database Schema Initialization

Database initialization can be accomplished in one of two
ways: from the library itself or manually.

Initializing the database via the library is relatively
straight forward:

```
var settings = new MySqlStreamStoreSettings("Host=localhost;Port=3306;User Id=root;Database=yourdatabase");

using (var streamStore = new MySqlStreamStore(settings))
{
  await streamStore.CreateSchemaIfNotExists();
}
```

This will create the MySql Stream Store database
objects in a database `yourdatabase`.

The other way is to manually run each of the scripts
found here - starting with `Tables.sql` and excluding
`DropAll.sql`.

# Settings

**Constructor**

Takes a connection string.

**CreateStreamNotifier**

Allows overriding of the stream store notifier.

**GetUtcNow**

A delegate to return the current UTC now. Used in testing
to control timestamps and time related operations. If not
set, the database server will control the timestamp.

**LogName**

The log name used for any of the log messages.

**ScavengeAsynchronously**

Allows setting whether or not deleting expired (i.e., older
than maxCount) messages happens in the same database
transaction as append to stream or not.

This does not effect scavenging when setting a stream's
metadata - it is always run in the same transaction.

**ConnectionFactory**

Allows overriding the way a `NpgsqlConnection` is created,
given a connection string. 

The default implementation simply passes the connection string
into the `NpgsqlConnection` constructor.

**DisableDeletionTracking**

Disables stream and message deletion tracking. Will increase
performance, however subscribers won't know if a stream or a
message has been deleted. This can be modified at runtime.