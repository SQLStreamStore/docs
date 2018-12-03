# Appending Messages to a Stream

The method to append a message to a stream on `IStreamStore` is:

    Task<AppendResult> AppendToStream(
            StreamId streamId,
            int expectedVersion,
            NewStreamMessage[] messages,
            CancellationToken cancellationToken = default);

1. The `streamId` is a value object that wraps a string ensuring non-null and no
   whitespace. StreamIds lengths should not exceed the limits set by underlying
   store.
2. The `expectedVersion` parameter is used for concurrency checking. You can
   supply a specific that you expect the stream to be at and if the stream is at
   a different version, a `WrongExpectedVersionException` is thrown.
   Alternatively you can supply `ExpectedVersion.Any` if you don't care what the
   current stream version is (including if it doesn't yet exist) or
   `ExpectedVersion.NoStream` if you explicitly expect it to not yet exist.
3. The `message` parameter defines the collection of messages that are appended
   in a transaction. If empty or null then the call is effectively a no-op.
4. `AppendResult` return value contains two properties, `CurrentVersion` and
   `CurrentPosition`. This is useful to return to callers if they need to
   subsequently load a projection to help with reading their own writes.

The constructor of `NewStreamMessage` is:

    public NewStreamMessage(
        Guid messageId,
        string type,
        string jsonData,
        string jsonMetadata = null)

1. `messageId` parameter is the unique id of the message being appended.
   It has an important function with regards to idempotent handling (See
   idempotency section below). MessageIds within stream must be unique.
2. `type` parameter represents the message type. Examples include `car-leased`
   and `customer-registered`. Using a fully qualified CLR type name (e.g.
   `Company.App.Domian.Event.Foo`) is anti-pattern. CLR types are re-named and
   moved so you want to maintain a map of event type -> clr type in your
   application.
3. `jsonData` paramater is string. SQLStreamStore doesn't check
   the structure nor validity of this. It is names json to encourage json only
   usage.
4. `jsonMetadata` paramater is option metadata about the message that is
   typically orthogonal and/or doesn't belong in the main message body. Examples
   of usage include the security context (`sub` / `client_id`) that caused the
   event as well as causation / correlation identifiers.

## Idempotency

Idempotent appends is when an stream append operation occurs multiple times but
the messages are appended once. This is useful for retry / resume type of
operations. When appending messages, the `MessageId` of `NewStreamMessage`,
coupled with the `expectedVersion`, determines the idempotency policy applied.

**With `ExpectedVersion.Any`**: If the collection of messages have been
previously written in the same order they appear in the append request, no new
messages are written. If the message ordering is different, or if there are
additional new messages with the previous written ones, then a
`WrongExpectedVersionException` is thrown. Examples:
  
    // using int instead of guid for message id to aid clarity
    var m1 = new NewStreamMessage(1, "t", "data");
    var m2 = new NewStreamMessage(2, "t", "data");
    var m3 = new NewStreamMessage(3, "t", "data");
    var m4 = new NewStreamMessage(4, "t", "data");

    // Creates stream
    store.AppendToStream(streamId, ExpectedVersion.Any, new [] { m1, m2, m3} );

    // Idempotent appends
    store.AppendToStream(streamId, ExpectedVersion.Any, new [] { m1, m2, m3} );
    store.AppendToStream(streamId, ExpectedVersion.Any, new [] { m1, m2 );
    store.AppendToStream(streamId, ExpectedVersion.Any, new [] { m2, m3} );
    store.AppendToStream(streamId, ExpectedVersion.Any, new [] { m3} );

    // Throws WrongExpectedVersionException
    store.AppendToStream(streamId, ExpectedVersion.Any, new [] { m2, m1, m3} ); // out of order
    store.AppendToStream(streamId, ExpectedVersion.Any, new [] { m3, m4} ); // partial previous write

**With a specific expected`** If the collection of messages have been previously
written in the same order they appear in the append request starting at the
expected version no new messages are written.

    // using int instead of guid for message id to aid clarity
    var m1 = new NewStreamMessage(1, "t", "data");
    var m2 = new NewStreamMessage(2, "t", "data");
    var m3 = new NewStreamMessage(3, "t", "data");
    var m4 = new NewStreamMessage(4, "t", "data");

    // Creates stream
    store.AppendToStream(streamId, ExpectedVersion.NoStream, new [] { m1, m2, m3} );

    // Idempotent appends
    store.AppendToStream(streamId, ExpectedVersion.NoStream, new [] { m1, m2, m3} );
    store.AppendToStream(streamId, ExpectedVersion.NoStream, new [] { m1 );
    store.AppendToStream(streamId, 0, new [] { m2 } );
    store.AppendToStream(streamId, 1, new [] { m3 } );

    // Throws WrongExpectedVersionException
    store.AppendToStream(streamId, ExpectedVersion.NoStream, new [] { m2, m1, m3} ); // out of order
    store.AppendToStream(streamId, 1, new [] { m3, m4} ); // partial previous writes

## Deterministic Message ID Generation

In order to leverage idempotent appends the message IDs should be the same for
identical messages. SQLStreamStore ships with a helper class
`DeterministicGuidGenerator` that can create GUIDs based on the message and
stream it is being appended to. When creating a determinisitic generator you are
required to supply a unique namespace that prevents other generators creating
the same GUIDs with the same input. You typically hard code the namespace in
your application and should never change it.

    var generator = new DeterministicGuidGenerator(Guid.Parse("C27B665E-AD32-4BBA-YOUR-OWN-VALUE"))

Creating a deterministic GUID:

    var streamId = "stream-1";
    var expectedVersion = 2; // This can be also ExpectedVersion.Any or ExpectedVersion.NoStream
    var messageId = generate.Create(streamId, expectedVersion, jsonData);

You then use this `messageId` when creating a `NewStreamMessage`.