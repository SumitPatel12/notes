_____
**Created**: 25-10-2024 09:15 am
**Status**: In Progress
**Tags**: #Kafka [[Kafka]]
**References**: [Kafka Wire Protocol](https://kafka.apache.org/protocol.html)
______

### Preliminaries
#### Network
Kafka uses a binary protocol over TCP. The protocol defines all APIs as request response message pairs. All messages are *size delimited*, and are made up of some primitive types.

The client initiates a socket connection and then writes a sequence of request messages and reads back the corresponding response message. No handshake is required during the connection or disconnection, we maintain a persistent connection.

The server guarantees serialisability for single TCP connection, i.e. requests will be processed in order and responses will follow in the same order. The brokers request processing allows only a single in-flight request per connection in to guarantee this ordering.

The server has a configurable hard limit on the request size, exceeding it will result in the connection being dropped.

#### Partitioning and Bootstrapping
Kafka partitions systems so not all servers have the complete data. Topics are split into a pre-defined number of partitions, with each being replicated a certain number of times.
Kafka clients are responsible for choosing the correct partitions for their requests, the broker plays no role in this. The requests these clients make must go through the active leader. To find out what partitions exist, where they exist and other metadata things each broker is configured to entertain such queries.
Commonly the clients don't make this metadata request very often, they make it at the time of startup and cache the data, the caches are invalidated and updated whenever the client receives and error that the metadata is out of date.

