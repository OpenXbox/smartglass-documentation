# SmartGlass Protocol

- [SmartGlass Protocol](#smartglass-protocol)
  - [Basic communication](#basic-communication)
    - [Client connection](#client-connection)
    - [Packet layout](#packet-layout)
    - [Strings in SmartGlass packets](#strings-in-smartglass-packets)

## Basic communication

The SmartGlass protocol communicates over _UDP_ port 5050.
Only the discovery and power on messages are transmitted in plain text, the rest is encrypted (see [Cryptography](cryptography.md)).

Nano protocol uses dynamic ports (UDP/TCP), negotioated via SmartGlass protocol's
broadcast channel.

### Client connection

1. Console might be powered on by [Power On packet](simple_message.md#power-on-request)
2. Client sends [Discovery Request](simple_message.md#discovery-request) to console
3. Console responds with [Discovery Response](simple_message.md#discovery-response)
4. Client parses [Certificate](simple_message.md#certificate) in that response
5. Client sets up a [Crypto Context](cryptography.md) using the console's public key
6. Client sends a [Connect Request](simple_message.md#connect-request) to the console
7. Console responds with [Connect Response](simple_message.md#connect-response)
8. Client sends a [Local Join Message](message.md#local-join), announcing itself
9. Upon [Acknowledgement](message.md#acknowledgement) client opens several [Channels](channels.md)
10. Received / sent [Heartbeat](channels.md#acknowledging-messages) packets ensure that
    client/host is alive

### Packet layout

General packet layout looks like the following:

| Name                | Note                                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------------------------ |
| Packet Header       | Either [SimpleMessage](simple_message.md#header) or [Message](message.md#header)                             |
| Unprotected Payload | For all [SimpleMessage packets](simple_message.md)                                                           |
| Protected Payload   | For [Connect](simple_message.md#connect-packet) or [Message](message.md) packet                              |
| \*Hash              | Only if packet has `Protected Payload`, see [Message Authentication](cryptography.md#message-authentication) |

> NOTE: All numeric values in the SmartGlass Protocol are in network / big-endian byteorder.

### Strings in SmartGlass packets

Usually strings are represented like the following:

| Type            | Description                           |
| --------------- | ------------------------------------- |
| uint16          | String length (excl. null-terminator) |
| uchar \* length | String                                |
| uchar '\\0'     | Null-terminator                       |

In this documentation, these strings are referenced as `SGString`
