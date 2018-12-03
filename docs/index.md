# Introduction

SQL Stream Store is a .NET library to assist with developing applications that use
event sourcing or wish to use stream based patterns over a relational database
and existing operational infrastructure.

This documentation assumes you already have some knowledge of event sourcing. If
not, there is a good guide on [Event Sourcing
Basics](https://eventstore.org/docs/event-sourcing-basics/index.html) by the
EventStore team.

The reasons for creating this and a comparison with NEventStore and EventStore
can be viewed
[here](https://github.com/SQLStreamStore/SQLStreamStore/issues/108#issuecomment-348154346).

# Things you need to know before adopting

- This is foremost a _library_ to help with working with stream based concepts
  implemented on an RDMBS infrastructure. It has no intention of becoming a full
  blown application/database server.

- While it helps you with working with stream concepts over a relational
  databases, you must still be have mechanical apathy with the underlying
  database such as limits, log growth, performance characteristics, exception
  handling etc.

- SQLStreamStore / relational databases will never be be as fast as custom
  stream / event based databases (e.g EventStore, Kafka). For DDD applications
  (aka "collaborative domains") that would otherwise use a traditional RDBMS
  databases (and ORMs) it should perform within expectations.

- Subscriptions (and thus projections) are eventually consistent and always will
  be.

- You must understand your application's characteristics in terms of load, data
  growth, acceptable latency for eventual consistency etc.
  
- Message metadata payloads are strings only and expected to be JSON format.
  This is for operational reasons to support splunking a database using it's
  standard administration tools. Other serialization formats (or compression)
  are not support (strictly speaking JSON isn't _enforced_ either).

- No support for ambient `System.Transaction` scopes enforcing the concept of
  the stream as the consistency and transactional boundary.