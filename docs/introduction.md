# SmartGlass Protocol

- [SmartGlass Protocol](#smartglass-protocol)
  - [Introduction](#introduction)
  - [Capabilities](#capabilities)
  - [Basics](#basics)
    - [Client connection](#client-connection)
    - [Packet layout](#packet-layout)
    - [Strings in SmartGlass packets](#strings-in-smartglass-packets)

## Introduction

SmartGlass is a remote control protocol developed by Microsoft for their Xbox gaming system.
It was originally developed for the Xbox 360, where it relied on an active Xbox Live connection for
communication with the console. With the Xbox One it directly communicates over the local network.

In this documentation only the Xbox One variant of SmartGlass is documented.

## Capabilities

- Start games / apps via TitleID
- Controller input ([Input Channel](channels.md#input-channel))
- Media player control ([Media Channel](channels.md#media-channel))
- Text input ([Text Channel](channels.md#text-channel))
- Live TV streaming ([Stump Channel](channels.md#input-tv-remote-channel))
- Gamestreaming ([Broadcast Channel](channels.md#broadcast-channel) / [NANO](nano.md))
- and more...

## Basics

The SmartGlass protocol communicates over _UDP_ port 5050.
Only the discovery and power on messages are transmitted in plain text, the rest is encrypted (see [Cryptography](cryptography.md)).

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
| Name                 | Note
\| -------------------- \| --------
| Packet Header        | Either [SimpleMessage](simple_message.md) or [Message](message.md)
| Unprotected Payload  | For [Discovery](simple_message.md#discovery-packet), [Connect](simple_message.md#connect-packet) or [Power On](simple_message.md#power-on-request) packet
| Protected Payload    | For [Connect](simple_message.md#connect-packet) or [Message](message.md) packet
\| \*Hash               | Only if packet has `Protected Payload`, see [Message Authentication](cryptography.md#message-authentication)

> NOTE: All numeric values in the SmartGlass Protocol are in network / big-endian byteorder.

### Strings in SmartGlass packets

Usually strings are represented like the following:

| Type            | Description                           |
| --------------- | ------------------------------------- |
| uint16          | String length (excl. null-terminator) |
| uchar \* length | String                                |
| uchar '\\0'     | Null-terminator                       |

In this documentation, these strings are referenced as `SGString`
