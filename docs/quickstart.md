# Quick Start

The core interface for all operations is `IStreamStore`. In this quick start, we
will:

 1. Add SQLStreamStore to a console project.
 2. Create an in-memory stream store.
 3. Append to a stream.
 4. Read the stream.
 5. Subscribe to a all stream.

**1. Add SQLStreamStore to a console project**

From command line in the directory of your project:

```bash
dotnet install SQLStreamStore
```

...or use your IDE's GUI.

**2. Create an in-memory stream store**

You will typically use the in-memory implementation for testing. It has been
carefully written to exhibit same behaviours as their real implementation (the
same suite of acceptance tests are applied to it) but obviously faster.

```csharp
var streamStore = new InMemoryStreamStore();
```

**3. Append to a stream**

First we create a `NewStreamMessage`:

```csharp
var jsonMessage = @"{ ""foo"" : ""bar"" }";
var messageId = Guid.NewGuid();
var messageType = "type";
var newStreamMessage = new NewStreamMessage(messageId, messageType, jsonMessage);
```

1. All messages stored are in JSON format.
2. Message IDs are `Guid`s. While here we are creating a `NewGuid()`, you should
   use deterministic GUID generation based on the message content (or some other
   mechanism) to better leverage idempotent append capabilities.
3. The message type is just a string. This is deliberately to discourage using
   CLR types whose names (and namespaces) have a tendency to change in a
   different life cycle to the data.
4. `NewStreamMessage` represents the data to append. It can be a single message,
   or a collection.

...then we append to a stream:

```csharp
var result = await streamStore.AppendToStream("stream-1", ExpectedVersion.NoStream, newStreamMessage);
```

1. The stream Id is a string. The max length supported is 1000 chars. Store
   implementations may use a hashed version of this for performance /
   optimisation reasons.
2. The `ExpectedVersion` defines the type of concurrency control applied when
   appending.
3. The returned result contains the stores `CurrentPosition` and the stream's
   `CurrentVersion`.

**4. Read the stream**

There are two ways to read from the store, read the individual stream, or read
across all stream.

Reading the individual stream and getting the first message:

```csharp
var readStreamPage = await streamStore.ReadStreamForwards("stream-1", StreamVersion.Start, maxCount: 100);
var streamMessage = readStreamPage.Messages[0];
```

1. The `StreamVersion` specifies where to start reading from.
2. The `maxcount` specifies the _maximum_ number of results to return. The value
   of this is dependent on how you tune your interaction with the store. Larger
   numbers may result in more memory usage, smaller numbers may result more
   round trips.

Reading the all stream and getting the first message:

```csharp
var readAllPage = await streamStore.ReadAllForwards(Position.Start, maxCount: 100);
streamMessage = readAllPage.Messages[0];
```

1. The `Position` specifies where to start reading from.

**5. Subscribe to all stream**

The subscription API takes a position _to continue from_ and two delegates to
handle when a message is receive and when the subscription is dropped:

```csharp
streamStore.SubscribeToAll(Position.None, StreamMessageReceived, SubscriptionDropped);
```

... and the corresponding handlers:

```csharp
private static Task StreamMessageReceived(
    IAllStreamSubscription subscription,
    StreamMessage streammessage,
    CancellationToken cancellationtoken)
{
    // Do something with the stream message such as projection
}

private static void SubscriptionDropped(
    IAllStreamSubscription subscription,
    SubscriptionDroppedReason reason,
    Exception exception)
{
    // React to the SubscriptionDroppedReason such as re-establish a subscription
}
```