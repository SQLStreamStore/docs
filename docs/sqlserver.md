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

# Initializing the Database

TODO

# Using

TODO

# Migrating from V2 Schema to V3

             // simplified code example
             
             var settings = new MsSqlStreamStoreV3Settings(); // you can set some not default values
             var v3Store = new MsSqlStreamStoreV3(settings);
             await v3Store.CreateSchemaIfNotExists();
             var checkSchemaResult = await v3Store.CheckSchema();
            _logger.LogInformation($"checkSchemaResult before Migrate  CurrentVersion {checkSchemaResult.CurrentVersion }  ");
            if (checkSchemaResult.CurrentVersion == 2)
            {
              //  you can consider to run migration as fire and forget to not block the main thread
              //Task.Factory.StartNew(() => MigrateToV3());
                
              _logger.LogInformation($"Before Migrate IsMatch {checkSchemaResult.IsMatch()} CurrentVersion {checkSchemaResult.CurrentVersion} ExpectedVersion {checkSchemaResult.ExpectedVersion} ");
              var progress = new Progress<MigrateProgress>();
              progress.ProgressChanged += (_, migrateProgress) =>
                   _logger.LogInformation($"Migration stage complete: {migrateProgress.Stage}");
              await v3Store.Migrate(progress, CancellationToken.None)
            
              checkSchemaResult = await v3Store.CheckSchema();
              _logger.LogInformation($"After Migrate IsMatch {checkSchemaResult.IsMatch()} CurrentVersion {checkSchemaResult.CurrentVersion} ExpectedVersion {checkSchemaResult.ExpectedVersion} ");
            }
            return v3Store;


See more details in [Wiki Example-of-Migration-to-MsSqlStreamStoreV3](https://github.com/SQLStreamStore/SQLStreamStore/wiki/Example-of-Migration-to-MsSqlStreamStoreV3)
            
            
