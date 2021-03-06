Message Delivery
===

In this section, we define the behavior of PCP broker when delivering messages.

### Client to client messages

Consider the following example. client A and client B are connected to a server S.
A is identified by the URI _pcp://client_a/controller_ and B by _pcp://client_b/agent_.
A wishes to send a [message][3] via S to B.

```
    client A                    server S                    client B
       |                           |                           |
       |        1 message          |                           |
       |-------------------------->|                           |
       |                           | 2                         |
       |                           |        3 message          |
       |                           |-------------------------->|

```

**A** sends a message to **S**, specifying a single URI in the message envelope's
*targets* field, _pcp://client_b/agent_ (1).

**S** receives the message from **A**, parses it and determines that it is valid.
**S** inspects the *targets* field in the message envelope
and prepares to deliver the message to each of the supplied URIs,
_pcp://client_b/agent_ in this particular case (2).

**S** determines that the URI points to **B**. It then delivers the message to
**B** (3).

### Client to client messages using wildcards

Consider the following example. client A, client B and client C are connected to
a server S. A is identified by the URI _pcp://client_a/controller_, B by
_pcp://client_b/agent_ and C by _pcp://client_c/agent_. A wishes to send a message
to all clients of type agent.

```
    client A                server S                 client B                 client C
       |                       |                       |                        |
       |        1 message      |                       |                        |
       |---------------------->|                       |                        |
       |                       | 2                     |                        |
       |                       |        3 message      |                        |
       |                       |---------------------->|                        |
       |                       |                                                |
       |                       |                     4 message                  |
       |                       |----------------------------------------------->|
       |                                                                        |

```

**A** sends a message to **S**, specifying a single URI in the message envelope's
*targets* field, _pcp://*/agent_. (1)

**S** receives the message from **A**, parses it and determines that it is valid.
**S** inspects the *targets* field in the message envelope and determines that
the URI _pcp://*/agent_ contains a wildcard. The wildcard is expanded to include
all clients that have specified their type as *agent*. (2)

**S** determines that the expanded URI points to both **B** and **C**. It then
delivers one copy of the message to both **B** and **C**. (3), (4)

### Client to server messages

Consider the following example. Client A is connected to a server S. A is identified
by the URI _pcp://client_a/controller_. A wishes to perform an inventory query and
thus needs to send a message to S (refer to the [inventory][2] section).

```
    client A                    server S
       |                           |
       |        1 message          |
       |-------------------------->|
       |                           | 2
       |        3 message          |
       |<--------------------------|

```

**A** sends a message to **S**, specifying a single URI in the message envelope's
*targets* field, _pcp:///server_ (1).

**S** receives the message from **A**, parses it and determines that it is valid.
**S** inspects the *targets* field in the message envelope and determines that
the message was directed at itself.
**S** performs a server specific task, determined by the value supplied in the
*message_type* field in the [message][3] envelope (2).
**S** constructs a resulting message and sends it to **A** using the URI supplied
in the original message's *sender* field, _pcp://client_a/contoller_ (3).

### Server Operation

From the above examples, it is clear that PCP Brokers must perform a number of
operations to deliver a message. In this section we describe the operation
requirements for the server.

#### Destination report

When the server processes a client message and the *destination_report* flag is
set, it must respond to the client with a [destination report message][5]
containing the list of URIs it will be sending the message to in the Data Chunk.

The *destination_report* flag is ignored in case of [inventory requests][2],
which are addressed directly to the server.

#### Message Expiration

The server must notify the sender of an expired message with a [TTL expired][6]
message.

#### Error handling

The server must respond to a client with an [error message][4] in case:

- the message cannot be parsed (see [message][3])
- the message envelope does not match the envelope schema (see [message][3])

The server will not take any action in the case where:

 - none of the recipients of the *targets* envelope entry is registered in the
 server; this include the case of wildcarded PCP URIs that expand into nothing

#### Debug data

When a server processes a client message, it may add a debug chunk with a JSON
`content` containing a *hops* entry that indicates the route the message has
taken so far in the message fabric. The *hops* entry is an array with the
following items:

| name | type | description
|------|------|------------
| server | string | PCP URI of the server that performed the processing hop
| time | string | time entry in ISO8601 format indicating when the processing took place
| stage | string | type of processing (e.g. "accepted", "delivered")

*TODO(ale):* add inter-server routing specs, if necessary once we implement
      distribution

[1]: association.md
[2]: inventory.md
[3]: message.md
[4]: error_handling.md
[5]: destination_report.md
[6]: ttl_expired.md
