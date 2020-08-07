There are several ways to read messages from SQL Stream Store.

# Read All

The 'all' stream is a _virtual_ stream over all messages in SQL Stream Store,
ordered by their written `Position`. There are two ways of reading the 'all'
stream, forwards and backwards.

**Backwards**

The method to read the 'all' stream backwards is:

```csharp
Task<ReadAllPage> ReadAllBackwards(
    long fromPositionInclusive,
    int maxCount,
    bool prefetchJsonData,
    CancellationToken cancellationToken = default)
```

1. `fromPositionInclusive` is an `Int64`  that specifies where to start reading
from. Use `Position.End` to start reading backwards from the end.
2. `maxCount` is an `Int32` to specify the page size of the read.
3. `prefetchJsonData` is a `Boolean` that specifies whether or not to preload
the message data.
4. The `ReadAllPage` contains several properties to give the caller information
about what was read. `Messages` are the messages in this read.
`FromPosition` represents where this page was read from.
`NextPosition` indicates where the next page should be read from. `IsEnd` indicates
if this is the last page to read, i.e., are there no more messages to read.
`Direction` indicates which direction the 'all' stream was read in, in this case
`ReadDirection.Backwards`. Finally, there is a method `ReadNext` which simply 
returns the next page.  

**Forwards**

The method to read the 'all' stream forwards is:

```csharp
Task<ReadAllPage> ReadAllForwards(
    long fromPositionInclusive,
    int maxCount,
    bool prefetchJsonData,
    CancellationToken cancellationToken = default)
```

1. `fromPositionInclusive` is an `Int64`  that specifies where to start reading
from. Use `Position.Start` to start reading forwards from the first message.
2. `maxCount` is an `Int32` to specify the page size of the read.
3. `prefetchJsonData` is a `Boolean` that specifies whether or not to preload
the message data.
4. The `ReadAllPage` contains several properties to give the caller information
about what was read. `Messages` are the messages in this read.
`FromPosition` represents where this page was read from.
`NextPosition` indicates where the next page should be read from. `IsEnd` indicates
if this is the last page to read, i.e., are there no more messages to read.
`Direction` indicates which direction the 'all' stream was read in, in this case
`ReadDirection.Forwards`. Finally, there is a method `ReadNext` which simply 
returns the next page.  
