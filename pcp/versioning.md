Message Versioning
===

This section describes how the Puppet Communications Protocol handles clients
attempting to communicate using different versions of the protocol.

### How clients and the pcp-broker determine protocol version

When a pcp-broker starts up it will mount a URI for each of the
versions it support. Consider the following scenario:

A pcp-broker, broker.example.com, supports versions 1, 2 and 3 of the PCP. It 
will mount the following URI's.

- wss://broker.example.com/v1
- wss://broker.example.com/v2
- wss://broker.example.com/v3

A client that wishes to communicate using version 2 of the protocol will connect
to the version 2 URI, `wss://broker.example.com/v2`. After establishing this
connection all communications MUST happen using version 2 of the protocol. 

When the broker receives a message on this URI it will attempt to parse the
version field of the message and validate it against the expected version. The
pcp-broker will respond with a
(http://puppetlabs.com/version_error)[#version_error messages] message if the
versions do not match.

Two versions of the protocol can be compatible to the point where translation 
from one to the other is possible. The following two sections describe the 
intended behaviour of the pcp-broker in those cases.

### Two protocol versions are compatible

When two versions of the protocol are declared as compatible, it is the 
responsibility of the pcp-broker to translate a message from the one version to 
the other.

Consider the following scenario:

Client A connects to the pcp-broker and states it's capability to send and 
receive messages using version 1 of the PCP. Client B connects to the 
pcp-broker states it's capability to send and receive messages using version 2 
of the PCP. How the client and server determines version capabilities are up the
client and broker implementations.

Version 1 and 2 are considered compatible versions of the protocol.

Client A sends a message intended for Client B. In this case, the pcp-broker 
MUST translate the message from version 1 to version 2.

### Two protocol versions are not compatible

When two version of the protocol are declared as not compatible, it is the
responsibility of the pcp-broker to send a message of type 
`http://puppetlabs.com/version_error` to the sender.

Consider the following scenario:

Client A connects to the pcp-broker and states it's capability to send and 
receive messages using version 1 of the PCP. Client B connects to the 
pcp-broker states it's capability to send and receive messages using version 3
of the PCP. How the client and server determines version capabilities are up the
client and broker implementations.

Version 1 and 3 are considered not compatible versions of the protocol.

Client A sends a message intended for Client B. In this case, the pcp-broker 
MUST respond with a message of type
`http://puppetlabs.com/version_error` to Client A.

### version_error messages

`http://puppetlabs.com/version_error` messages are described by the following
JSON schema:


```
{
    "properties" : {
        "id" : { "type" : string" },
        "target" : { "type" : "string" },
        "reason" : { "type" : "string" }
    },
    "required" : ["id", "target", "reason"],
    "additionalProperties" : false
}
```

The fields in the message are defined as:

* id : The id of the message this message is in response to
* target : The URI of the URI the message was targeted at
* reason : The reason a version_error message was sent