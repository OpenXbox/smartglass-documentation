# Nano Protocol

Nano (aka. Nano RDP / Codename: Arcadia) is the protocol Gamestreaming is based on.
You can see strong similarities when looking at the Windows IoT RDP server implementation, _NanoRDPServer.exe_.

It's basically RTP over TCP (configuration / status of session) and UDP (data).

- [Nano Protocol](#nano-protocol)
  - [How it works](#how-it-works)
  - [TCP vs. UDP](#tcp-vs-udp)
  - [Data layout](#data-layout)
    - [TCP Socket](#tcp-socket)
  - [Padding](#padding)
    - [Example](#example)
  - [RTP Header](#rtp-header)
  - [Channels](#channels)
  - [Payload Types](#payload-types)
  - [Streamer Protocol Version](#streamer-protocol-version)
  - [Channel Opening](#channel-opening)
  - [Streamer Handshaking](#streamer-handshaking)
  - [Packets](#packets)
    - [Packet Layout](#packet-layout)
    - [Control Handshake Packet](#control-handshake-packet)
    - [Channel Control Packet](#channel-control-packet)
      - [Channel Control Header](#channel-control-header)
      - [Channel Control Types](#channel-control-types)
      - [Channel Control Payloads](#channel-control-payloads)
        - [Create](#create)
        - [Open](#open)
        - [Close](#close)
    - [UDP Handshake Packet](#udp-handshake-packet)
    - [Streamer Packet](#streamer-packet)
      - [Streamer Flags](#streamer-flags)
      - [Streamer Header](#streamer-header)
      - [Audio Video Streamer Payload Type](#audio-video-streamer-payload-type)
      - [Input Payload Type](#input-payload-type)
  - [Streamer Payloads](#streamer-payloads)
    - [Reference Timestamp](#reference-timestamp)
    - [Timestamp of Data Packets](#timestamp-of-data-packets)
    - [Frame Id](#frame-id)
    - [Audio](#audio)
      - [Audio Format](#audio-format)
      - [Audio Codec](#audio-codec)
      - [Audio Server Handshake](#audio-server-handshake)
      - [Audio Client Handshake](#audio-client-handshake)
      - [Audio Control](#audio-control)
        - [Audio Control Flags](#audio-control-flags)
      - [Audio Data](#audio-data)
    - [Video](#video)
      - [Lost frame reporting](#lost-frame-reporting)
      - [Video Format](#video-format)
      - [Video Codec](#video-codec)
      - [Video Server Handshake](#video-server-handshake)
      - [Video Client Handshake](#video-client-handshake)
      - [Video Control](#video-control)
        - [Video Control Flags](#video-control-flags)
      - [Video Data](#video-data)
        - [Video Data Flags](#video-data-flags)
    - [Input](#input)
      - [Input Button Model](#input-button-model)
      - [Input Analog Model](#input-analog-model)
      - [Input Extension Model](#input-extension-model)
      - [Input Server Handshake](#input-server-handshake)
      - [Input Client Handshake](#input-client-handshake)
      - [Frame Ack](#frame-ack)
      - [Input Frame](#input-frame)
    - [Control Protocol](#control-protocol)
      - [Control Header](#control-header)
      - [Control Payload Type](#control-payload-type)
      - [Session Init](#session-init)
      - [Session Create](#session-create)
      - [Session Create Response](#session-create-response)
      - [Session Destroy](#session-destroy)
      - [Video Statistics](#video-statistics)
      - [Realtime Telemetry](#realtime-telemetry)
        - [Telemetry Field](#telemetry-field)
      - [Change Video Quality](#change-video-quality)
      - [Initiate Network Test](#initiate-network-test)
      - [Network Information](#network-information)
      - [Network Test Response](#network-test-response)
      - [Controller Event](#controller-event)
        - [Controller Event Type](#controller-event-type)

## How it works

1. Client opens [Broadcast Channel](channels.md#broadcast-channel) in SmartGlass Protocol and requests
   start of gamestreaming to initialize and receive the connection-data
   [`TCP Port`, `UDP Port` and `Session ID`](channels.md#initializing).
2. Client creates sockets for `TCP` and `UDP`.
3. Client sends [Control Handshake Packet](#control-handshake-packet) with
   own, randomly generated, `Connection Id`.
4. Host responds with [Control Handshake Packet](#control-handshake-packet)
   and it's own Connection Id.
5. Client starts a background loop, sending [UDP Handshake](#udp-handshake-packet) packets.
6. Host sends [`Channel Create` and `Channel Open`](#channel-control-packet) packets
7. Client replies with [Channel Open](#open) packets for the appropriate channels
8. Host sends [Server Handshake](#audio-video-streamer-payload-type) packets for
   [Video](#video) and [Audio](#audio), announcing it's available formats
9. Client replies with [Client Handshake](#audio-video-streamer-payload-type) packets,
   choosing the desired format.
10. Client sends [Audio / Video Control](#audio-video-streamer-payload-type) packets,
    starting the stream.
11. UDP Data comes flying in.
12. Client stops [UDP Handshake](#udp-handshake-packet) loop.
13. ... Client processes data ...

## TCP vs. UDP

Nano uses these two IP protocols in the following ways:

**TCP**

- [Control Handshaking](#control-handshake-packet)
- [Creating/Opening/Closing Channels](#channel-control-packet)
- Streamer Client/Server **Handshaking**
- Streamer **control** messaging
- [Control Protocol messaging](#control-protocol)

**UDP**

- [UDP handshaking](#udp-handshake-packet)
- Streamer **data** packets

## Data layout

The RTP header is in network / big-endian byteorder, the rest of
the packet is usually in little-endian byteorder.

### TCP Socket

All TCP packets have a nano-packet size prepended
(`uint32`, Little Endian), so several packets can be sent _stacked_.
Example of a single TCP packet with two nano packets inside:

```text
packet size #1: 0x123
nano packet #1: byte[0x123]
packet size #2: 0x321
nano packet #2: byte[0x321]
```

Ideally the chunks are split directly when received from the socket
and then forwarded to the unpacker/processor/parser.

## Padding

Nano, different to SmartGlass core, uses padding of type
[ANSI X.923](https://en.wikipedia.org/wiki/Padding_(cryptography)#ANSI_X.923)
aka. padding with **zero**, the last byte defining the number of padding bytes.

Aligment is **4 bytes**.

### Example

**Plaintext (9 bytes)**
`DE AD BE EF DE AD BE EF DE`

**Padded Plaintext (12 bytes) - 3 bytes padding**
`DE AD BE EF DE AD BE EF DE 00 00 03`

## RTP Header

![Rtp Header](rtpheader.png)

Image Source: <https://de.wikipedia.org/wiki/Real-Time_Transport_Protocol>

Image Copyright License: **CC-by-sa-3.0**

- **Total size**: 0x0C (12)
- **V**: Version (2 bits): Always 2
- **P**: Padding (1 bit): If set, last byte of payload is padding size.
- **X**: Extension (1 bit): If set, variable size header
  extension exists - **Not used by Nano**
- **CSRC Count**: (4 bits) Should be 0 - **Not used by Nano**
- **M**: Marker (1 bit): Indicates a _special packet_ in stream data
- **Payload type**: (7 bits) See [Payload Types](#payload-types)
- **Sequence number**: (16 bits) Incrementing number, indvidual to each
  channel, used to keep track of order and lost packets
- **Timestamp**: (32 bits) Timestamp, unsure how it's actually calculated
- **SSRC**: (32 bits) For Nano its split into 2 x 16 bits:

> NOTE: Only UDP packets use `Connection Id` field, TCP sets it to `0`
>
> - Bits 0-15: Connection Id
> - Bits 16-32: Channel Id

## Channels

Nano communicates over the following channels + a core channel (`0`).
Core Channel handles:

- [Control Handshake](#control-handshake-packet)
- [UDP Handshake](#udp-handshake-packet)
- [Channel Control Packet](#channel-control-packet)

The following listed channels only process messages of type `Streamer`.

> Usage of `TcpBase` has not been spotted in the wild to date.

| Channel        | Id                                                    |
| -------------- | ----------------------------------------------------- |
| Video          | `Microsoft::Rdp::Dct::Channel::Class::Video`          |
| Audio          | `Microsoft::Rdp::Dct::Channel::Class::Audio`          |
| Chat Audio     | `Microsoft::Rdp::Dct::Channel::Class::ChatAudio`      |
| Control        | `Microsoft::Rdp::Dct::Channel::Class::Control`        |
| Input          | `Microsoft::Rdp::Dct::Channel::Class::Input`          |
| Input Feedback | `Microsoft::Rdp::Dct::Channel::Class::Input Feedback` |
| TCP Base       | `Microsoft::Rdp::Dct::Channel::Class::TcpBase`        |

## Payload Types

Payload Type is encoded in the [RTP Header](#rtp-header) `Payload type` field.

| Payload Type    | Value |
| --------------- | ----- |
| Streamer        | 0x23  |
| Control         | 0x60  |
| Channel Control | 0x61  |
| UDP Handshake   | 0x64  |

- **Streamer**: Sending encoded video/audio/input data,
  sending [Control Protocol packets](#control-protocol)
- **Control**: Initial TCP handshake, informs each participants
  about the used Connection Id.
- **Channel Control**: Creating / opening / closing [Channels](#channels).
- **UDP Handshake**: Initial UDP handshake, used to inform
  the host about the used UDP port of the client side.

## Streamer Protocol Version

Currently these are the used streamer protocol version:

- **Video**: Version 5
- **Audio**: Version 4
- **Input**: Version 3

## Channel Opening

By default, console [creates](#create) and [opens](#open) Channels for:

- Audio
- Video
- ChatAudio
- Control

Client needs to respond with a [Channel Open](#open) packet.
After that, handshaking needs to be done, see [Streamer Handshaking](#streamer-handshaking)

> NOTE: `Input` and `Input Feedback` are somewhat special, see
> [Input](#input)

## Streamer Handshaking

Normally, a `Server Handshake` is sent by the console, client has to
respond with a `Client Handshake`.

> NOTE: [Input Feedback](#input) and [Chat Audio](#audio) are special!

## Packets

### Packet Layout

```
    ├── RtpHeader
    └── Payload (RtpHeader.PayloadType)
        ├── Control Handshake
        ├── Channel Control
        │   ├── Channel Create
        │   ├── Channel Open
        │   └── Channel Close
        ├── UDP Handshake
        └── Streamer
            ├── TCP Header
            ├── UDP Header
            ├── Audio Payload
            │   ├── Server Handshake
            │   ├── Client Handshake
            │   ├── Control
            │   └── Data
            ├── Video Payload
            │   ├── Server Handshake
            │   ├── Client Handshake
            │   ├── Control
            │   └── Data
            ├── Input Payload
            │   ├── Server Handshake
            │   ├── Client Handshake
            │   ├── Frame Ack
            │   └── Frame
            └── Control Payload
                └── Control Header
                    ├── Session Init
                    ├── Session Create
                    ├── Session Create Response
                    ├── Session Destroy
                    ├── Video Statistics
                    ├── Realtime Telemetry
                    ├── Change Video Quality
                    ├── Initiate Network Test
                    ├── Network Information
                    ├── Network Test Response
                    └── Controller Event
```

### Control Handshake Packet

| Offset (hex) | Offset (dec) | Type   | Description   |
| -----------: | -----------: | ------ | ------------- |
|         0x00 |            0 | byte   | Type          |
|         0x01 |            1 | uint16 | Connection Id |

**Total size**: 0x03 (3)

- **Type**: Handshake type: SYN: 0 - Sent by client, ACK: 1 - Response from console
- **Connection Id**: Client sends randomly generated connection
  Id, Host responds with it's own.

### Channel Control Packet

#### Channel Control Header

| Offset (hex) | Offset (dec) | Type    | Description          |
| -----------: | -----------: | ------- | -------------------- |
|         0x00 |            0 | uint32  | Channel Control Type |
|         0x04 |            4 | byte\[] | Payload              |

**Total size**: _variable_

- **Channel Control Type**: See [Channel Control Types](#channel-control-types)
- **Payload**: Depending on Type-field

#### Channel Control Types

| Channel Control | Value |
| --------------- | ----- |
| Create          | 0x02  |
| Open            | 0x03  |
| Close           | 0x04  |

#### Channel Control Payloads

##### Create

**Channel Control Type**: `0x02`

| Offset (hex) | Offset (dec) | Type        | Description  |
| -----------: | -----------: | ----------- | ------------ |
|         0x00 |            0 | uint16      | Name length  |
|         0x02 |            2 | uchar\[len] | Channel Name |
|   0x02 + len |      2 + len | uint32      | Flags        |

**Total size**: _variable_

- **Channel Name**: See [Channels](#channels)
- **Flags**: Unknown

##### Open

**Channel Control Type**: `0x03`

| Offset (hex) | Offset (dec) | Type       | Description  |
| -----------: | -----------: | ---------- | ------------ |
|         0x00 |            0 | uint32     | Flags length |
|         0x04 |            4 | byte\[len] | Flags        |

**Total size**: _variable_

- **Flags (optional)**: Depending on prepended size.
  Are sent by the client _as-is_ in the responding [Channel Open](#open) packet.

##### Close

**Channel Control Type**: `0x04`

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Flags       |

**Total size**: 0x04 (4)

- **Flags**: Same flags as sent with `Create` and `Open` packet

### UDP Handshake Packet

| Offset (hex) | Offset (dec) | Type | Description |
| -----------: | -----------: | ---- | ----------- |
|         0x00 |            0 | byte | Type        |

**Total size**: 0x01 (1)

- **Type**: Handshake Type

### Streamer Packet

#### Streamer Flags

| Flag             | Value |
| ---------------- | ----- |
| Got Seq/Prev Seq | 0x01  |
| Unknown          | 0x02  |

#### Streamer Header

UDP usually uses flags: `0x0`.
TCP usually uses flags: `0x3`.
For FEC there are additional flags to process (TODO).

| Offset (hex) | Offset (dec) | Type         | Description                |
| -----------: | -----------: | ------------ | -------------------------- |
|         0x00 |            0 | uint32       | Flags                      |
|       \*0x?? |         \*?? | uint32       | \*Sequence Number          |
|       \*0x?? |         \*?? | uint32       | \*Previous Sequence Number |
|         0x?? |           ?? | uint32       | Streamer Payload Type      |
|       \*0x?? |         \*?? | uint32       | \*Streamer Payload Length  |
|         0x?? |           ?? | byte\[\*len] | Streamer Payload           |

**Total size**: _variable_

> **Streamer Payload Length**: Only used if `Streamer Payload Type`
> is **not** 0 (0: Control packet with own header)

- **Flags**: Depending on used [Channel](#channels)
- **Sequence Number**: If Flag `Got Seq/Prev Seq` is set -> incrementing number,
  specific for channel and participant side
- **Previous Sequence Number**: If Flag `Got Seq/Prev Seq` is set -> previously
  sent `Sequence Number`
- **Streamer Payload Type**:  See [Audio / Video Payload Type](#audio-video-streamer-payload-type)
  and [Input Payload Type](#input-payload-type)
- **Streamer Payload**: Depending on `Streamer Payload Type`

#### Audio Video Streamer Payload Type

| Type             | Value |
| ---------------- | ----- |
| Server Handshake | 0x01  |
| Client Handshake | 0x02  |
| Control          | 0x03  |
| Data             | 0x04  |

#### Input Payload Type

| Type             | Value |
| ---------------- | ----- |
| Server Handshake | 0x01  |
| Client Handshake | 0x02  |
| Frame Ack        | 0x03  |
| Frame            | 0x04  |

## Streamer Payloads

### Reference Timestamp

Reference Timestamp is **milliseconds** since epoch.

### Timestamp of Data Packets

The Timestamp used on each Data/Frame Packet is current time in
**microseconds** -> **relative** to the [Reference Timestamp](#reference-timestamp)

### Frame Id

The initial Frame Id is chosen randomly by the participant that's
sending the handshake.
Frame Ids of the data/frame packet will start with the initial
value and increment on each packet.

### Audio

> NOTE: For [ChatAudio Channel](#channels), the **client** needs to send the [Audio Server Handshake](#audio-server-handshake)
> to the console after [opening](#open) the channel!!!
> Console responds with [Audio Client Handshake](#audio-server-handshake)

#### Audio Format

| Offset (hex) | Offset (dec) | Type   | Description           |
| -----------: | -----------: | ------ | --------------------- |
|         0x00 |            0 | uint32 | Channels              |
|         0x04 |            4 | uint32 | Sample Rate           |
|         0x08 |            8 | uint32 | Audio Codec           |
|              |              |        | If AudioCodec is PCM: |
|         0x0C |           12 | uint32 | Bit Depth             |
|         0x10 |           16 | uint32 | Type                  |

**Total size**: _variable_

- **Channels**: Audio Channels of encoded data
- **Sample Rate**: Samplerate of encoded data
- **Audio Codec**: See [Audio Codec](#audio-codec)
- **Bit Depth (optional)**: If `Audio Codec` is `PCM` this field
  gives the bit depth of encoded data
- **Type (optional)**: If `Audio Codec` is `PCM` this field
  gives the type (`Integer` or `Float`) of encoded data

#### Audio Codec

| Codec | Value |
| ----- | ----- |
| Opus  | 0x00  |
| AAC   | 0x01  |
| PCM   | 0x02  |

#### Audio Server Handshake

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type              | Description         |
| -----------: | -----------: | ----------------- | ------------------- |
|         0x00 |            0 | uint32            | Protocol Version    |
|         0x04 |            4 | uint64            | Reference Timestamp |
|         0x0C |           12 | uint32            | Formats length      |
|         0x10 |           16 | AudioFormat\[len] | Audio Formats       |

**Total size**: _variable_

- **Protocol Version**: See [Streamer Protocol Version](#streamer-protocol-version)
- **Reference Timestamp**: See [Reference Timestamp](#reference-timestamp)
- **Audio Formats**: Available `Audio Formats`, Array of
  [Audio Format](#audio-format)

#### Audio Client Handshake

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type        | Description      |
| -----------: | -----------: | ----------- | ---------------- |
|         0x00 |            0 | uint32      | Initial Frame Id |
|         0x04 |            4 | AudioFormat | Audio Format     |

**Total size**: _variable_

- **Initial Frame Id**: See [Frame Id](#frame-id)
- **Audio Format**: By client desired [Audio Format](#audio-format)

#### Audio Control

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type   | Description         |
| -----------: | -----------: | ------ | ------------------- |
|         0x00 |            0 | uint32 | Audio Control Flags |

**Total size**: 0x04 (4)

- **Audio Control Flags**: See [Audio Control Flags](#audio-control-flags)

##### Audio Control Flags

| Flag         | Bits                  | Mask |
| ------------ | --------------------- | ---- |
| Reinitialize | `0000 0000 0000 0010` | 0x02 |
| Start Stream | `0000 0000 0000 1000` | 0x08 |
| Stop Stream  | `0000 0000 0001 0000` | 0x10 |

#### Audio Data

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type       | Description |
| -----------: | -----------: | ---------- | ----------- |
|         0x00 |            0 | uint32     | Flags       |
|         0x04 |            4 | uint32     | Frame Id    |
|         0x08 |            8 | uint64     | Timestamp   |
|         0x10 |           16 | uint32     | Data length |
|         0x14 |           20 | byte\[len] | Data        |

**Total size**: _variable_

- **Flags**: Unknown, for `AAC` it seems to always be `0x4`
- **Frame Id**: See [Frame Id](#frame-id)
- **Timestamp**: See [Timestamp of Data Packets](#timestamp-of-data-packets)
- **Data**: Encoded data

### Video

#### Lost frame reporting

TODO

#### Video Format

| Offset (hex) | Offset (dec) | Type   | Description           |
| -----------: | -----------: | ------ | --------------------- |
|         0x00 |            0 | uint32 | FPS                   |
|         0x04 |            4 | uint32 | Width                 |
|         0x08 |            8 | uint32 | Height                |
|         0x0C |           12 | uint32 | Video Codec           |
|              |              |        | If VideoCodec is RGB: |
|         0x10 |           16 | uint32 | Bpp                   |
|         0x14 |           20 | uint32 | Bytes                 |
|         0x18 |           24 | uint64 | Red Mask              |
|         0x20 |           32 | uint64 | Green Mask            |
|         0x28 |           40 | uint64 | Blue Mask             |

**Total size**: _variable_

- **FPS**: Frames per second
- **Width**: Video Frame Width
- **Height**: Video Frame Height
- **Video Codec**: See `Video Codec`
- **Bpp (optional)**: If `Video Codec` is `RGB` this field gives
  the bits per pixel (color depth)
- **Bytes (optional)**:  If `Video Codec` is `RGB` this field gives
  the bytes per pixel
- **Red Mask (optional)**: If `Video Codec` is `RGB` this field gives
  the `Red Mask`
- **Green Mask (optional)**: If `Video Codec` is `RGB` this field gives
  the `Green Mask`
- **Blue Mask (optional)**: If `Video Codec` is `RGB` this field gives
  the `Blue Mask`

#### Video Codec

| Codec | Value |
| ----- | ----- |
| H264  | 0x00  |
| YUV   | 0x01  |
| RGB   | 0x02  |

#### Video Server Handshake

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type              | Description         |
| -----------: | -----------: | ----------------- | ------------------- |
|         0x00 |            0 | uint32            | Protocol Version    |
|         0x04 |            4 | uint32            | Width               |
|         0x08 |            8 | uint32            | Height              |
|         0x0C |           12 | uint32            | FPS                 |
|         0x10 |           16 | uint64            | Reference Timestamp |
|         0x18 |           24 | uint32            | Formats length      |
|         0x1C |           28 | VideoFormat\[len] | Video Formats       |

**Total size**: _variable_

- **Protocol Version**: See [Streamer Protocol Version](#streamer-protocol-version)
- **Width**: Video Width
- **Height**: Video Height
- **FPS**: Frames per second
- **Reference Timestamp**: [Reference Timestamp](#reference-timestamp)
- **Video Formats**: Available `Video Formats`, Array of
  [Video Format](#video-format)

#### Video Client Handshake

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type        | Description      |
| -----------: | -----------: | ----------- | ---------------- |
|         0x00 |            0 | uint32      | Initial Frame Id |
|         0x04 |            4 | VideoFormat | Video Format     |

**Total size**: _variable_

- **Initial Frame Id**: See [Frame Id](#frame-id)
- **Video Format**: By client desired [Video Format](#video-format)

#### Video Control

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type   | Description                    |
| -----------: | -----------: | ------ | ------------------------------ |
|         0x00 |            0 | uint32 | Video Control Flags            |
|              |              |        | If "Last displayed frame" set: |
|         0x?? |            ? | uint32 | Last displayed Frame Id        |
|         0x?? |            ? | uint64 | Timestamp                      |
|              |              |        | If "Queue Depth" set:          |
|         0x?? |            ? | uint32 | Queue Depth                    |
|              |              |        | If "Lost frames" set:          |
|         0x?? |            ? | uint32 | First lost Frame               |
|         0x?? |            ? | uint32 | Last lost Frame                |

**Total size**: _variable_

- **Video Control Flags**: See [Video Control Flags](#video-control-flags)
- **Last displayed Frame Id (optional)**: If flag `Last displayed frame` is set, this
  informs the host about the last displayed `Video Frame` by `Frame Id`
- **Timestamp (optional)**: If flag `Last displayed frame` is set, this informs the
  host about the time the frame was displayed
  ([Timestamp of Data Packets](#timestamp-of-data-packets))
- **Queue Depth (optional)**: If flag `Queue Depth` is set: Informs the host about queue depth
- **First lost Frame (optional)**: If flag `Lost frames` is set, this informs the host about
  the first last Frame, given the `Frame Id`
- **Last lost Frame (optional)**: If flag `Lost frames` is set, this informs the host about
  the last last Frame, given the `Frame Id`

##### Video Control Flags

| Flag                 | Bits                  | Mask |
| -------------------- | --------------------- | ---- |
| Request Keyframe     | `0000 0000 0000 0100` | 0x04 |
| Start Stream         | `0000 0000 0000 1000` | 0x08 |
| Stop Stream          | `0000 0000 0001 0000` | 0x10 |
| Queue Depth          | `0000 0000 0010 0000` | 0x20 |
| Lost frames          | `0000 0000 0100 0000` | 0x40 |
| Last displayed frame | `0000 0000 1000 0000` | 0x80 |

#### Video Data

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type       | Description  |
| -----------: | -----------: | ---------- | ------------ |
|         0x00 |            0 | uint32     | Flags        |
|         0x04 |            4 | uint32     | Frame Id     |
|         0x08 |            8 | uint64     | Timestamp    |
|         0x10 |           16 | uint32     | Total size   |
|         0x14 |           20 | uint32     | Packet count |
|         0x18 |           24 | uint32     | Offset       |
|         0x1C |           28 | uint32     | Data length  |
|         0x20 |           32 | byte\[len] | Data         |

**Total size**: _variable_

- **Flags**: See [Video Data Flags](#video-data-flags)
- **Frame Id**: See [Frame Id](#frame-id)
- **Timestamp**: See [Timestamp of Data Packets](#timestamp-of-data-packets)
- **Total size**: Total size of the Video Frame
- **Packet count**: Count of total frame chunks
- **Offset**: Beginning position of current frame chunk
- **Data**: Encoded data

##### Video Data Flags

| Flag     | Bits                  | Mask |
| -------- | --------------------- | ---- |
| Keyframe | `0000 0000 0000 0010` | 0x02 |

### Input

> NOTE: Input channel does not get created by default!
> You need to send a [Controller Event](#controller-event) over
> [Control Protocol](#control-protocol) to make the console
> [Create](#create) the [Input Channel](#channels).
>
> After client sends the [Open Input Channel](#open) packet to the console, console
> sends an [Open Input Channel](#open) packet packet too - then console
> [creates](#create) and [opens](#open) the [Input Feedback Channel](#channels).
>
> Client [opens](#open) [Input Feedback Channel](#channels).
> Client sends the [Input Server Handshake](#input-server-handshake) for
> `Input Feedback` - console responds with [Input Client Handshake](#input-client-handshake).
>
> Console will further send a [Input Server Handshake](#input-server-handshake)
> for `Input` which client needs to respond with [Input Client Handshake](#input-client-handshake)

#### Input Button Model

This structure starts with `all-zero`.
When a button-state is changed, the appropriate field is incremented by one.

**Advice**: Keep track of the button-state, don't increment the value if new button-
state is the same as the old one - otherwise you will provoke a _stuck button_
in case an [Input Frame](#input-frame) packet is lost.

**Example**

```C
// Initial value - button is not pressed
button_state.A = state.RELEASED
button_model.A = 0

SendPacket()

while (getting_input_data) {
  // A button is pressed
  if (new_state != button_state.A) {
    button_model.A++
    button_state.A = new_state

    SendPacket()
  }
}

// State can be checked from value like:
// bool released = (button_model.A % 2) == 0 || button_model.A == 0
```

| Offset (hex) | Offset (dec) | Type | Description      |
| -----------: | -----------: | ---- | ---------------- |
|         0x00 |            0 | byte | D-Pad Up         |
|         0x01 |            1 | byte | D-Pad Down       |
|         0x02 |            2 | byte | D-Pad Left       |
|         0x03 |            3 | byte | D-Pad Right      |
|         0x04 |            4 | byte | Start            |
|         0x05 |            5 | byte | Back             |
|         0x06 |            6 | byte | Left thumbstick  |
|         0x07 |            7 | byte | Right thumbstick |
|         0x08 |            8 | byte | Left shoulder    |
|         0x09 |            9 | byte | Right shoulder   |
|         0x0A |           10 | byte | Guide            |
|         0x0B |           11 | byte | Unknown          |
|         0x0C |           12 | byte | A                |
|         0x0D |           13 | byte | B                |
|         0x0E |           14 | byte | X                |
|         0x0F |           15 | byte | Y                |

**Total size**: 0x10 (16)

#### Input Analog Model

| Offset (hex) | Offset (dec) | Type   | Description          |
| -----------: | -----------: | ------ | -------------------- |
|         0x00 |            0 | byte   | Left Trigger         |
|         0x01 |            1 | byte   | Right Trigger        |
|         0x02 |            2 | uint16 | Left thumbstick X    |
|         0x04 |            4 | uint16 | Left thumbstick Y    |
|         0x06 |            6 | uint16 | Right thumbstick X   |
|         0x08 |            8 | uint16 | Right thumbstick Y   |
|         0x0A |           10 | byte   | Left Rumble trigger  |
|         0x0B |           11 | byte   | Right Rumble trigger |
|         0x0C |           12 | byte   | Left Rumble handle   |
|         0x0D |           13 | byte   | Right Rumble handle  |

**Total size**: 0x0E (14)

#### Input Extension Model

| Offset (hex) | Offset (dec) | Type | Description            |
| -----------: | -----------: | ---- | ---------------------- |
|         0x00 |            0 | byte | Unknown 1              |
|         0x01 |            1 | byte | Unknown 2              |
|         0x02 |            2 | byte | Left Rumble trigger 2  |
|         0x03 |            3 | byte | Right Rumble trigger 2 |
|         0x04 |            4 | byte | Left Rumble handle 2   |
|         0x05 |            5 | byte | Right Rumble handle 2  |
|         0x06 |            6 | byte | Unknown 3              |
|         0x07 |            7 | byte | Unknown 4              |
|         0x08 |            8 | byte | Unknown 5              |

**Total size**: 0x09 (9)

- **Unknown 1**: Set to 1 for gamepad

#### Input Server Handshake

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type   | Description      |
| -----------: | -----------: | ------ | ---------------- |
|         0x00 |            0 | uint32 | Protocol Version |
|         0x04 |            4 | uint32 | Desktop Width    |
|         0x08 |            8 | uint32 | Desktop Height   |
|         0x0C |           12 | uint32 | Max Touches      |
|         0x10 |           16 | uint32 | Initial frame Id |

**Total size**: 0x14 (20)

- **Protocol Version**: See [Streamer Protocol Version](#streamer-protocol-version)
- **Desktop Width**: Host's display width
- **Desktop Height**: Host's display height
- **Max Touches**: `0`
- **Initial Frame Id**: [Frame Id](#frame-id)

#### Input Client Handshake

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type   | Description         |
| -----------: | -----------: | ------ | ------------------- |
|         0x00 |            0 | uint32 | Max Touches         |
|         0x04 |            4 | uint64 | Reference Timestamp |

**Total size**: 0x0C (12)

- **Max Touches**: Input: `10`, Input Feedback: `0`
- **Reference Timestamp**: See [Reference Timestamp](#reference-timestamp)

#### Frame Ack

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Acked frame |

**Total size**: 0x04 (4)

- **Acked frame**: `Frame Id` of [Input Frame](#input-frame) to acknowledge

#### Input Frame

Header: [Streamer Header](#streamer-header)

| Offset (hex) | Offset (dec) | Type      | Description           |
| -----------: | -----------: | --------- | --------------------- |
|         0x00 |            0 | uint32    | Frame Id              |
|         0x04 |            4 | uint64    | Timestamp             |
|         0x0C |           12 | uint64    | Created Timestamp     |
|         0x14 |           20 | byte\[16] | Input Button Model    |
|         0x24 |           36 | byte\[14] | Input Analog Model    |
|              |              |           | If remaining data:    |
|         0x32 |           50 | byte\[9]  | Input Extension Model |

**Total size**: 0x32 (50) or 0x3B (59)

- **Frame Id**: See [Frame Id](#frame-id)
- **Timestamp**: See [Timestamp of Data Packets](#timestamp-of-data-packets)
- **Created Timestamp**: See [Timestamp of Data Packets](#timestamp-of-data-packets)
- **Input Button Model**: See [Input Button Model](#input-button-model)
- **Input Analog Model**: See [Input Analog Model](#input-analog-model)
- **Input Extension Model (optional)**: [Input Extension Model](#input-extension-model)

### Control Protocol

Header: [Streamer Header](#streamer-header)
Control Protocol packets have `Streamer Payload Type` set to `0`

#### Control Header

| Offset (hex) | Offset (dec) | Type    | Description              |
| -----------: | -----------: | ------- | ------------------------ |
|         0x00 |            0 | uint32  | Previous Sequence (DUPE) |
|         0x04 |            4 | uint16  | Unknown 1                |
|         0x06 |            6 | uint16  | Unknown 2                |
|         0x08 |            8 | uint16  | Control Payload Type     |
|         0x0A |           10 | byte\[] | Control Payload          |

**Total size**: _variable_

- **Previous Sequence**: Previous Sequence Number (DUPE)
- **Unknown 1**: TODO: `1`
- **Unknown 2**: TODO: `1406`
- **Control Payload Type**: See [Control Payload Type](#control-payload-type)
- **Control Payload**: Depends on [Control Payload Type](#control-payload-type)

#### Control Payload Type

| Type                    | Value |
| ----------------------- | ----- |
| Session Init            | 0x01  |
| Session Create          | 0x02  |
| Session Create Response | 0x03  |
| Session Destroy         | 0x04  |
| Video Statistics        | 0x05  |
| Realtime Telemetry      | 0x06  |
| Change Video Quality    | 0x07  |
| Initiate Network Test   | 0x08  |
| Network Information     | 0x09  |
| Network Test Response   | 0x0A  |
| Controller Event        | 0x0B  |

#### Session Init

| Offset (hex) | Offset (dec) | Type    | Description |
| -----------: | -----------: | ------- | ----------- |
|         0x00 |            0 | byte\[] | Unknown     |

#### Session Create

| Offset (hex) | Offset (dec) | Type          | Description |
| -----------: | -----------: | ------------- | ----------- |
|         0x00 |            0 | uint32        | Length      |
|         0x04 |            4 | byte\[length] | Unknown     |

#### Session Create Response

| Offset (hex) | Offset (dec) | Type    | Description |
| -----------: | -----------: | ------- | ----------- |
|         0x00 |            0 | byte\[] | Unknown     |

#### Session Destroy

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Unknown 1   |
|         0x04 |            4 | uint32 | Length      |
|         0x08 |            8 | uint32 | Unknown 2   |

#### Video Statistics

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Unknown 1   |
|         0x04 |            4 | uint32 | Unknown 2   |
|         0x08 |            8 | uint32 | Unknown 3   |
|         0x0C |           12 | uint32 | Unknown 4   |
|         0x10 |           16 | uint32 | Unknown 5   |
|         0x14 |           20 | uint32 | Unknown 6   |

#### Realtime Telemetry

| Offset (hex) | Offset (dec) | Type                   | Description      |
| -----------: | -----------: | ---------------------- | ---------------- |
|         0x00 |            0 | uint16                 | Field count      |
|         0x02 |            2 | TelemetryField\[count] | Telemetry Fields |

##### Telemetry Field

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint16 | Key         |
|         0x02 |            2 | uint64 | Value       |

#### Change Video Quality

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Unknown 1   |
|         0x04 |            4 | uint32 | Unknown 2   |
|         0x08 |            8 | uint32 | Unknown 3   |
|         0x0C |           12 | uint32 | Unknown 4   |
|         0x10 |           16 | uint32 | Unknown 5   |
|         0x14 |           20 | uint32 | Unknown 6   |

#### Initiate Network Test

| Offset (hex) | Offset (dec) | Type    | Description |
| -----------: | -----------: | ------- | ----------- |
|         0x00 |            0 | byte\[] | Unknown     |

#### Network Information

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint64 | Unknown 1   |
|         0x08 |            8 | byte   | Unknown 2   |
|         0x09 |            9 | uint32 | Unknown 3   |

#### Network Test Response

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Unknown 1   |
|         0x04 |            4 | uint32 | Unknown 2   |
|         0x08 |            8 | uint32 | Unknown 3   |
|         0x0C |           12 | uint32 | Unknown 4   |
|         0x10 |           16 | uint32 | Unknown 5   |
|         0x14 |           20 | uint64 | Unknown 6   |
|         0x1C |           28 | uint64 | Unknown 7   |
|         0x24 |           36 | uint32 | Unknown 8   |

#### Controller Event

By sending a `Controller Event` packet to the console, the console
will open the `Input`/`Input Feedback` channels and send the
appropriate `Channel Open` packets to the client.

| Offset (hex) | Offset (dec) | Type | Description       |
| -----------: | -----------: | ---- | ----------------- |
|         0x00 |            0 | byte | Event             |
|         0x01 |            1 | byte | Controller Number |

- **Event**: [Controller Event Type](#controller-event-type)
- **Controller Number**: Null-indexed controller number

##### Controller Event Type

| Event   | Value |
| ------- | ----- |
| Removed | 0x00  |
| Added   | 0x01  |
