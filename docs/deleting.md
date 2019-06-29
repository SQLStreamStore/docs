# Deleting a Stream

The method to delete a stream is:

```csharp
    public Task DeleteStream(
        StreamId streamId,
        int expectedVersion = ExpectedVersion.Any,
        CancellationToken cancellationToken = default)
```

1. The `streamId` is a value object that wraps a string ensuring non-null and no
      whitespace. `StreamId` lengths should not exceed the limits set by underlying
      store.
2. The `expectedVersion` parameter is used for concurrency checking. You can
   supply a specific that you expect the stream to be at and if the stream is at
   a different version, a `WrongExpectedVersionException` is thrown.
   Alternatively you can supply `ExpectedVersion.Any` if you don't care what the
   current stream version is (including if it doesn't yet exist).

# Deleting a Stream Message

Sometimes, for operational reasons, or model reasons, you may need to
remove a message from the middle of a stream. SQL Stream Store provides an API for this
scenario but it _must be used with the utmost caution_.

The method to delete a stream message is:

```csharp
    Task DeleteMessage(
        StreamId streamId,
        Guid messageId,
        CancellationToken cancellationToken = default)
```

1. The `streamId` is a value object that wraps a string ensuring non-null and no
      whitespace. `StreamId` lengths should not exceed the limits set by underlying
      store.
2. The `messageId` is the `Guid` of the message you want to delete.