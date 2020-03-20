# Message

After the initial connection is made, the actual communication happens
via packets of type `0xD00D`. We refer to them as messages of type `Message`.

- [Message](#message)
  - [Header](#header)
    - [Header Flags](#header-flags)
  - [Message Types](#message-types)
  - [Message Payloads](#message-payloads)
    - [Local Join](#local-join)
      - [Device Capabilities](#device-capabilities)
      - [Examples](#examples)
    - [Channel Start Request](#channel-start-request)
    - [Channel Start Response](#channel-start-response)
    - [Channel Stop](#channel-stop)
    - [Console Status](#console-status)
      - [Active Title](#active-title)
      - [Active Title Location](#active-title-location)
    - [Active Surface Change](#active-surface-change)
      - [Active Surface Type](#active-surface-type)
    - [Fragment](#fragment)
    - [Acknowledgement](#acknowledgement)
    - [Json](#json)
      - [Json Fragmentation](#json-fragmentation)
    - [Auxiliary Stream](#auxiliary-stream)
      - [Endpoint](#endpoint)
    - [Disconnect](#disconnect)
      - [Disconnect Reason](#disconnect-reason)
    - [Power Off](#power-off)
    - [Game DVR Record](#game-dvr-record)
    - [Unsnap](#unsnap)
    - [Gamepad](#gamepad)
      - [Gamepad Button](#gamepad-button)
    - [Paired Identity State Changed](#paired-identity-state-changed)
      - [Paired Identity State](#paired-identity-state)
    - [Media State](#media-state)
      - [Sound Level](#sound-level)
      - [Media Playback Status](#media-playback-status)
      - [Media Transport State](#media-transport-state)
      - [Media Type](#media-type)
      - [Media Metadata](#media-metadata)
    - [Media Controller Removed](#media-controller-removed)
    - [Media Command Result](#media-command-result)
    - [Media Command](#media-command)
      - [Media Control Command](#media-control-command)
    - [Orientation](#orientation)
    - [Compass](#compass)
    - [Inclinometer](#inclinometer)
    - [Gyrometer](#gyrometer)
    - [Accelerometer](#accelerometer)
    - [Touch](#touch)
      - [Touch Action](#touch-action)
      - [Touchpoint](#touchpoint)
    - [Title Launch](#title-launch)
    - [System Text Done](#system-text-done)
      - [Text Result](#text-result)
    - [System Text Acknowledge](#system-text-acknowledge)
    - [System Text Input](#system-text-input)
      - [Text Delta](#text-delta)
    - [Title Text Selection](#title-text-selection)
    - [Title Text Input](#title-text-input)
    - [Text Configuration](#text-configuration)
      - [Text Input Scope](#text-input-scope)
      - [Text Option Flags](#text-option-flags)

## Header

| Offset (hex) | Offset (dec) | Type   | Description                                   |
| ------------ | ------------ | ------ | --------------------------------------------- |
| 0x00         | 0            | uint16 | Packet Type                                   |
| 0x02         | 2            | uint16 | Protected Payload Length                      |
| 0x04         | 4            | uint32 | Sequence Number                               |
| 0x08         | 8            | uint32 | Target Participant Id                         |
| 0x0C         | 12           | uint32 | Source Participant Id                         |
| 0x10         | 16           | uint16 | Flags (Version, NeedAck, IsFragment, MsgType) |
| 0x12         | 18           | uint64 | Channel Id                                    |

- **Packet Type**: Always `0xD00D` for message packet
- **Protected Payload Length**: Payload length before encryption **excluding**
  padding. That is, the length of the plaintext
- **Sequence Number**: Incrementing sequence number - if packet
  was not acknowledged even if requested, message gets sent
  again with same sequence number. Start index is `1`.
- **Target Participant Id**: Target Id as seen from sender, client sets this to 0
- **Source Participant Id**:  Id of sender, client gets that from
  [Connect Response](simple_message.md#connect-response)
- **Flags**: See [Header Flags](#header-flags)
- **Channel Id**: Negotioated Channel Id (see
  [Channel Start Response](#channel-start-response))

### Header Flags

| Flag                 | Bits                  | Mask   |
| -------------------- | --------------------- | ------ |
| Version              | `0011 0000 0000 0000` | 0x3000 |
| Need Acknowledgement | `0100 0000 0000 0000` | 0x4000 |
| Is Fragment          | `1000 0000 0000 0000` | 0x8000 |
| Message Type         | `0000 1111 1111 1111` | 0x0FFF |

- **Version**: Always `2`
- **Need Acknowledgement**: Indicates if the message needs to
  be acknowledged by a message of type [Acknowledgement](#acknowledgement)
- **Is Fragment**: Indicates fragmented payload, see [Fragment](#fragment)
- **Message Type**: See [Message Types](#message-types)

## Message Types

Direction as seen from the client perspective:

- → Sent from client to console
- ← Sent from console to client
- ↔ Both parties send this messagetype
- x Unknown

| Name                                                            | Value | Direction | Channel     |
| --------------------------------------------------------------- | ----- | :-------: | ----------- |
| [Acknowledgement](#acknowledgement)                             | 0x01  |     ↔     | _Various_   |
| Group                                                           | 0x02  |     x     | x           |
| [Local Join](#local-join)                                       | 0x03  |     →     | Core        |
| Stop Activity                                                   | 0x05  |     x     | x           |
| [Auxiliary Stream](#auxiliary-stream)                           | 0x19  |     ↔     | Title       |
| [Active Surface Change](#active-surface-change)                 | 0x1A  |     ←     | Title       |
| Navigate                                                        | 0x1B  |     x     | x           |
| [Json](#json)                                                   | 0x1C  |     ↔     | _Various_   |
| Tunnel                                                          | 0x1D  |     x     | x           |
| [Console Status](#console-status)                               | 0x1E  |     ←     | Core        |
| [Title Text Configuration](#text-configuration)                 | 0x1F  |     ←     | SystemText  |
| [Title Text Input](#title-text-input)                           | 0x20  |     ↔     | SystemText  |
| [Title Text Selection](#title-text-selection)                   | 0x21  |     →     | SystemText  |
| MirroringRequest                                                | 0x22  |     x     | x           |
| [Title Launch](#title-launch)                                   | 0x23  |     →     | Core        |
| [Channel Start Request](#channel-start-request)                 | 0x26  |     →     | Core        |
| [Channel Start Response](#channel-start-response)               | 0x27  |     ←     | Core        |
| [Channel Stop](#channel-stop)                                   | 0x28  |     x     | Core        |
| System                                                          | 0x29  |     x     | x           |
| [Disconnect](#disconnect)                                       | 0x2A  |     →     | Core        |
| [Title Touch](#touch)                                           | 0x2E  |     →     | SystemInput |
| [Accelerometer](#accelerometer)                                 | 0x2F  |     →     | SystemInput |
| [Gyrometer](#gyrometer)                                         | 0x30  |     →     | SystemInput |
| [Inclinometer](#inclinometer)                                   | 0x31  |     →     | SystemInput |
| [Compass](#compass)                                             | 0x32  |     →     | SystemInput |
| [Orientation](#orientation)                                     | 0x33  |     →     | SystemInput |
| [Paired Identity State Changed](#paired-identity-state-changed) | 0x36  |     ←     | Core        |
| [Unsnap](#unsnap)                                               | 0x37  |     →     | Core        |
| [Game DVR Record](#game-dvr-record)                             | 0x38  |     →     | Core        |
| [Power Off](#power-off)                                         | 0x39  |     →     | Core        |
| [Media Controller Removed](#media-controller-removed)           | 0xF00 |     ←     | SystemMedia |
| [Media Command](#media-command)                                 | 0xF01 |     →     | SystemMedia |
| [Media Command Result](#media-command-result)                   | 0xF02 |     ←     | SystemMedia |
| [Media State](#media-state)                                     | 0xF03 |     ←     | SystemMedia |
| [Gamepad](#gamepad)                                             | 0xF0A |     →     | SystemInput |
| [System Text Configuration](#text-configuration)                | 0xF2B |     ←     | SystemText  |
| [System Text Input](#system-text-input)                         | 0xF2C |     ↔     | SystemText  |
| [System Touch](#touch)                                          | 0xF2E |     →     | SystemInput |
| [System Text Acknowledge](#system-text-acknowledge)             | 0xF34 |     ↔     | SystemText  |
| [System Text Done](#system-text-done)                           | 0xF35 |     ↔     | SystemText  |

## Message Payloads

### Local Join

**Message Type**: `0x03`
**Response**: None
**Requests Ack**: `YES`

Pair client to console.

| Offset (hex) | Offset (dec) | Type     | Description         |
| -----------: | -----------: | -------- | ------------------- |
|         0x00 |            0 | uint16   | Device Type         |
|         0x02 |            2 | uint16   | Native Width        |
|         0x04 |            4 | uint16   | Native Height       |
|         0x06 |            6 | uint16   | DPI X               |
|         0x08 |            8 | uint16   | DPI Y               |
|         0x0A |           10 | uint64   | Device Capabilities |
|         0x12 |           18 | uint32   | Client Version      |
|         0x16 |           22 | uint32   | OS Major Version    |
|         0x1A |           26 | uint32   | OS Minor Version    |
|         0x1E |           30 | SGString | Display Name        |

- **Device Type**: See [Client Type](simple_message.md#client-type)
- **Native Width**: Display resolution width from connecting client
- **Native Height**: Display resolution height from connecting client
- **DPI X**: Pixel Density on X-axis from client display
- **DPI Y**: Pixel Density on Y-axis from client display
- **Device Capabilities**: See [Device Capabilities](#device-capabilities)
- **Client Version**: SmartGlass client version
- **OS Major Version**: Operating System major version
- **OS Minor Version**: Operating System minor version
- **Display Name**: Client's display name

#### Device Capabilities

| Capability    | Bits                  | Mask               |
| ------------- | --------------------- | ------------------ |
| None          | `0000 0000 0000 0000` | 0x00               |
| Streaming     | `0000 0000 0000 0001` | 0x01               |
| Audio         | `0000 0000 0000 0010` | 0x02               |
| Accelerometer | `0000 0000 0000 0100` | 0x04               |
| Compass       | `0000 0000 0000 1000` | 0x08               |
| Gyrometer     | `0000 0000 0001 0000` | 0x10               |
| Inclinometer  | `0000 0000 0010 0000` | 0x20               |
| Orientation   | `0000 0000 0100 0000` | 0x40               |
| All           | `1111 1111 1111 1111` | 0xFFFFFFFFFFFFFFFF |

#### Examples

**Windows**

```ini
DeviceType = ClientType.WindowsStore
NativeWidth = 1080
NativeHeight = 1920
DpiX = 96
DpiY = 96
DeviceCapabilities = DeviceCapabilities.All
ClientVersion = 15
OSMajor = 6
OSMinor = 2
DisplayName = "SmartGlass-PC"
```

**Android**

```ini
DeviceType = ClientType.Android
# Resolution is portrait mode
NativeWidth = 720
NativeHeight = 1280
DpiX = 160
DpiY = 160
DeviceCapabilities = DeviceCapabilities.All
ClientVersion = 151117100  # v2.4.1511.17100-Beta
OSMajor = 22  # Android 5.1.1 - API Version 22
OSMinor = 0
DisplayName = "com.microsoft.xboxone.smartglass.beta"
```

### Channel Start Request

**Message Type**: `0x26`
**Response**: [Channel Start Response](#channel-start-response)
**Requests Ack**: `NO`

Start opening a `Service Channel`.

| Offset (hex) | Offset (dec) | Type      | Description          |
| -----------: | -----------: | --------- | -------------------- |
|         0x00 |            0 | uint32    | Channel Request Id   |
|         0x04 |            4 | uint32    | Title Id             |
|         0x08 |            8 | byte\[16] | Service Channel GUID |
|         0x18 |           24 | uint32    | Activity Id          |

- **Channel Request Id**: Incrementing number, it's used to
  match with [Channel Start Response](#channel-start-response)
- **Title Id**: Set for [Title channel](channels.md#title-channel), otherwise `0`
- **Service Channel GUID**: See [Service Channels](channels.md#service-channels)
- **Activity Id**: Always `0`

### Channel Start Response

**Message Type**: `0x27`
**Response**: None
**Requests Ack**: `NO`

Response to [Channel Start Request](#channel-start-request).

| Offset (hex) | Offset (dec) | Type   | Description        |
| -----------: | -----------: | ------ | ------------------ |
|         0x00 |            0 | uint32 | Channel Request Id |
|         0x04 |            4 | uint64 | Target Channel Id  |
|         0x0C |           12 | uint32 | Result             |

- **Channel Request Id**: Matches with
  [Channel Start Request](#channel-start-request)
- **Target Channel Id**: Assigned `Channel Id` to be used in
  [Message Header](#header)
- **Result**: Result code, `0` on success

### Channel Stop

**Message Type**: `0x28`
**Response**: None
**Requests Ack**: `NO`

Stop an opened `Service Channel`

| Offset (hex) | Offset (dec) | Type   | Description       |
| -----------: | -----------: | ------ | ----------------- |
|         0x00 |            0 | uint64 | Target Channel Id |

- **Target Channel Id**: `Channel Id` received by
  [Channel Start Response](#channel-start-response)

### Console Status

**Message Type**: `0x1E`
**Response**: None
**Requests Ack**: `YES`

Informs client about running titles and Xbox OS version.

| Offset (hex) | Offset (dec) | Type                | Description        |
| -----------: | -----------: | ------------------- | ------------------ |
|         0x00 |            0 | uint32              | Live TV Provider   |
|         0x04 |            4 | uint32              | Major Version      |
|         0x08 |            8 | uint32              | Minor Version      |
|         0x0C |           12 | uint32              | Build Number       |
|         0x10 |           16 | SGString            | Locale             |
|         0x?? |            ? | uint16              | Active Title Count |
|         0x?? |            ? | ActiveTitle\[count] | Active Titles      |

- **Live TV Provider**: Live TV provider Id
- **Major Version**: Major OS version
- **Minor Version**: Minor OS version
- **Build Number**: OS builder number
- **Locale**: Locale string
- **Active Title Count**: Number of `Active Titles`
- **Active Titles**: Array of [Active Title](#active-title)

#### Active Title

| Offset (hex) | Offset (dec) | Type      | Description       |
| -----------: | -----------: | --------- | ----------------- |
|         0x00 |            0 | uint32    | Title Id          |
|         0x04 |            4 | uint16    | Title Disposition |
|         0x06 |            6 | byte\[16] | Product Id        |
|         0x16 |           22 | byte\[16] | Sandbox Id        |
|         0x26 |           38 | SGString  | AUM Id            |

- **Title Id**: Title Id
- **Title Disposition**: 1 bit: HasFocus-Flag, 15 bits: [Active Title Location](#active-title-location)
- **Product Id**: Product Id
- **Sandbox Id**: Sandbox Id
- **AUM Id**: Application User Model Id

#### Active Title Location

| Location   | Value |
| ---------- | ----- |
| Full       | 0x00  |
| Fill       | 0x01  |
| Snapped    | 0x02  |
| Start View | 0x03  |
| System UI  | 0x04  |
| Default    | 0x05  |

### Active Surface Change

**Message Type**: `0x1A`
**Response**: None
**Requests Ack**: `NO`

Informs client about surface change, used in auxiliary-stream context.

| Offset (hex) | Offset (dec) | Type      | Description        |
| -----------: | -----------: | --------- | ------------------ |
|         0x00 |            0 | uint16    | Surface Type       |
|         0x02 |            2 | uint16    | Server TCP Port    |
|         0x04 |            4 | uint16    | Server UDP Port    |
|         0x06 |            6 | byte\[16] | Session Id         |
|         0x16 |           22 | uint16    | Render Width       |
|         0x18 |           24 | uint16    | Render Height      |
|         0x1A |           26 | byte\[16] | Master Session Key |

- **Surface Type**: See [Active Surface Type](#active-surface-type)
- **Server TCP Port**: Used with `Auxiliary Stream`
- **Server UDP Port**: Used with `Auxiliary Stream`
- **Session Id**: Used with `Auxiliary Stream`
- **Render Width**: Used with `Auxiliary Stream`
- **Render Height**: Used with `Auxiliary Stream`
- **Master Session Key**: Used with `Auxiliary Stream`

#### Active Surface Type

| Type              | Value |
| ----------------- | ----- |
| Blank             | 0x00  |
| Direct            | 0x01  |
| HTML              | 0x02  |
| Title Text Entity | 0x03  |

### Fragment

**Message Type**: _variable_
**Response**: _variable_
**Requests Ack**: _variable_
**Is Fragment**: `YES`

Used for messages that need to be fragmented.
When all fragments are received, concatenate the data blobs and parse
the assembled data as indicated `Message Type`.

| Offset (hex) | Offset (dec) | Type       | Description    |
| -----------: | -----------: | ---------- | -------------- |
|         0x00 |            0 | uint32     | Sequence Begin |
|         0x04 |            4 | uint32     | Sequence End   |
|         0x08 |            8 | uint16     | Data length    |
|         0x0A |           10 | byte\[len] | Data           |

- **Sequence Begin**: First sequence number of the fragment-set
- **Sequence End**: Last sequence number (+1) of the fragement-set
- **Data**: Data fragment

### Acknowledgement

**Message Type**: `0x01`
**Response**: None
**Requests Ack**: `NO`

Acknowledge a message from sender, alternatively used to request hearbeat from peer.

| Offset (hex) | Offset (dec) | Type         | Description           |
| -----------: | -----------: | ------------ | --------------------- |
|         0x00 |            0 | uint32       | Low Watermark         |
|         0x04 |            4 | uint32       | Processed List Length |
|         0x08 |            8 | uint32\[len] | Processed List        |
|         0x?? |            ? | uint32       | Rejected List Length  |
|         0x?? |            ? | uint32\[len] | Rejected List         |

- **Low Watermark**: Last received/processed sequence number
- **Processed List**: Processed sequence numbers (array of `uint32`)
- **Rejected List**: Rejected sequence numbers (array of `uint32`)

### Json

**Message Type**: `0x1C`
**Response**: _variable_
**Requests Ack**: _variable_

Used to transfer commands or info in text-form.

| Offset (hex) | Offset (dec) | Type     | Description |
| -----------: | -----------: | -------- | ----------- |
|         0x00 |            0 | SGString | Text        |

- **Text**: JSON Body

#### Json Fragmentation

In case the `Protected Payload` of a Json Message exceeds `1024 bytes`,
the message gets fragmented.

Fragmentation is done by base64-encoding and splitting-up the Json string, the maximum
fragment length being `905 bytes` (When serializing without spaces).

Example fragment-set:

```json
# Fragment #0
{"datagram_size":"24","datagram_id":"1","fragment_offset":"0","fragment_length":"12","fragment_data":"eyJ0ZXN0Ijoi"}
# Fragment #1
{"datagram_size":"24","datagram_id":"1","fragment_offset":"12","fragment_length":"12","fragment_data":"dmFsdWUifQ=="}

# After concatenation
"eyJ0ZXN0IjoidmFsdWUifQ=="

# After decoding
'{"test":"value"}'
```

- **datagram_size**: Total base64-string length
- **datagram_id**: Identifier of the fragment-set
- **fragment_offset**: Position of the current fragment
- **fragment_length**: Length of the current fragment
- **fragment_data**: Base64 string

Receiving participant checks if a set of fragments is received completely by summarizing `fragment_length` fields
for the specific `datagram_id` and checking it against `datagram_size`.

When all fragments are received, they are ordered by `fragment_offset` and the `fragment_data` is concatenated and
base64-decoded.

### Auxiliary Stream

**Message Type**: `0x19`
**Response**: None
**Requests Ack**: `YES`

Used for _SmartGlass Experience_ aka. game companion stuff.
The only known utilization is in Fallout 4.

| Offset (hex) | Offset (dec) | Type            | Description                   |
| -----------: | -----------: | --------------- | ----------------------------- |
|         0x00 |            0 | byte            | Connection Info Flag          |
|              |              |                 | If Connection Info Flag == 1: |
|         0x01 |            1 | uint16          | AES Key length                |
|         0x03 |            3 | byte\[len]      | AES Key                       |
|           ?? |           ?? | uint16          | Server IV length              |
|           ?? |           ?? | byte\[len]      | Server IV                     |
|           ?? |           ?? | uint16          | Client IV length              |
|           ?? |           ?? | byte\[len]      | Client IV                     |
|           ?? |           ?? | uint16          | HMAC Key length               |
|           ?? |           ?? | byte\[len]      | HMAC Key                      |
|           ?? |           ?? | uint16          | Endpoints Size                |
|           ?? |           ?? | Endpoint\[size] | Endpoints                     |

- **Connection Info Flag**: Handshake: `0`, Connection Data: `1`
- **AES Key**: AES-CBC Key
- **Server IV**: Server's Initialization Vector
- **Client IV**: Client's Initialization Vector
- **HMAC Key**: HMAC key
- **Endpoints Size**: Endpoint count
- **Endpoints**: Advertised title endpoints, See [Endpoint](#endpoint)

#### Endpoint

| Type     | Description |
| -------- | ----------- |
| SGString | IP Address  |
| SGString | Port        |

### Disconnect

**Message Type**: `0x2A`
**Response**: None
**Requests Ack**: `NO`

Disconnect client from console.

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Reason      |
|         0x04 |            4 | uint32 |  Error code |

- **Reason**: See [Disconnect Reason](#disconnect-reason)
- **Error code**: Error code

#### Disconnect Reason

| Reason      | Value |
| ----------- | ----- |
| Unspecified | 0x00  |
| Error       | 0x01  |
| Power Off   | 0x02  |
| Maintenance | 0x03  |
| AppClose    | 0x04  |
| SignOut     | 0x05  |
| Reboot      | 0x06  |
| Disabled    | 0x07  |
| Low Power   | 0x08  |

### Power Off

**Message Type**: `0x39`
**Response**: None
**Requests Ack**: `NO`

Send poweroff to the console.

| Offset (hex) | Offset (dec) | Type     | Description |
| -----------: | -----------: | -------- | ----------- |
|         0x00 |            0 | SGString | Live ID     |

- **Live ID**: Live ID of console to power off. This info is stored in the [Discovery Response Certificate](simple_message.md#certificate)

### Game DVR Record

**Message Type**: `0x38`
**Response**: None
**Requests Ack**: `YES`

Save a DVR clip

| Offset (hex) | Offset (dec) | Type  | Description      |
| -----------: | -----------: | ----- | ---------------- |
|         0x00 |            0 | int32 | Start Time Delta |
|         0x04 |            4 | int32 | End Time Delta   |

- **Start Time Delta**: Start time of recording in seconds (e.g. -60 for last minute)
- **End Time Delta**: End time of recording in seconds (e.g. 0 for _now_)

### Unsnap

**Message Type**: `0x37`
**Response**: None
**Requests Ack**: `NO`

Unsnap currently snapped application.

| Offset (hex) | Offset (dec) | Type | Description |
| -----------: | -----------: | ---- | ----------- |
|         0x00 |            0 | byte | Unknown     |

- **Unknown**: Unknown

### Gamepad

**Message Type**: `0xF0A`
**Response**: None
**Requests Ack**: `NO`

Send gamepad control (not for use with low latency _gamestreaming_).

| Offset (hex) | Offset (dec) | Type    | Description        |
| -----------: | -----------: | ------- | ------------------ |
|         0x00 |            0 | uint64  | Timestamp          |
|         0x08 |            8 | uint16  | Buttons            |
|         0x0A |           10 | float32 | Left Trigger       |
|         0x0E |           14 | float32 | Right Trigger      |
|         0x12 |           18 | float32 | Left Thumbstick X  |
|         0x16 |           22 | float32 | Left Thumbstick Y  |
|         0x1A |           26 | float32 | Right Thumbstick X |
|         0x1E |           30 | float32 | Right Thumbstick Y |

- **Timestamp**: Timestamp
- **Buttons**: See Flags [Gamepad Button](#gamepad-button)
- **Left Trigger**: Left Trigger value
- **Right Trigger**: Right trigger value
- **Left Thumbstick X**: Left thumbstick x-axis value
- **Left Thumbstick Y**: Left thumbstick y-axis value
- **Right Thumbstick X**: Right thumbstick x-axis value
- **Right Thumbstick Y**: Right thumbstick y-axis value

#### Gamepad Button

| Flag             | Bits                  | Mask   |
| ---------------- | --------------------- | ------ |
| Clear            | `0000 0000 0000 0000` | 0x00   |
| Enroll           | `0000 0000 0000 0001` | 0x01   |
| Nexus            | `0000 0000 0000 0010` | 0x02   |
| Menu             | `0000 0000 0000 0100` | 0x04   |
| View             | `0000 0000 0000 1000` | 0x08   |
| A                | `0000 0000 0001 0000` | 0x10   |
| B                | `0000 0000 0010 0000` | 0x20   |
| X                | `0000 0000 0100 0000` | 0x40   |
| Y                | `0000 0000 1000 0000` | 0x080  |
| D-Pad Up         | `0000 0001 0000 0000` | 0x100  |
| D-Pad Down       | `0000 0010 0000 0000` | 0x200  |
| D-Pad Left       | `0000 0100 0000 0000` | 0x400  |
| D-Pad Right      | `0000 1000 0000 0000` | 0x800  |
| Left Shoulder    | `0001 0000 0000 0000` | 0x1000 |
| Right Shoulder   | `0010 0000 0000 0000` | 0x2000 |
| Left Thumbstick  | `0100 0000 0000 0000` | 0x4000 |
| Right Thumbstick | `1000 0000 0000 0000` | 0x8000 |

### Paired Identity State Changed

**Message Type**: `0x36`
**Response**: None
**Requests Ack**: `NO`

Informs client about paired identity state change.

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint16 | State       |

- **State**: See [Paired Identity State](#paired-identity-state)

#### Paired Identity State

| State      | Value |
| ---------- | ----- |
| Not Paired | 0x00  |
| Paired     | 0x01  |

### Media State

**Message Type**: `0xF03`
**Response**: None
**Requests Ack**: `YES`

Informs client about media playback state.

| Offset (hex) | Offset (dec) | Type                | Description      |
| -----------: | -----------: | ------------------- | ---------------- |
|         0x00 |            0 | uint32              | Title Id         |
|         0x08 |            8 | SGString            | AUM Id           |
|         0x0A |           10 | SGString            | Asset Id         |
|         0x0E |           14 | uint16              | Media Type       |
|         0x12 |           18 | uint16              | Sound Level      |
|         0x16 |           22 | uint32              | Enabled commands |
|         0x1A |           26 | uint32              | Playback status  |
|         0x1E |           30 | float32             | Rate             |
|         0x22 |           34 | uint64              | Position         |
|         0x2A |           42 | uint64              | Media Start      |
|         0x32 |           50 | uint64              | Media End        |
|         0x38 |           58 | uint64              | Min Seek         |
|         0x42 |           66 | uint64              | Max Seek         |
|         0x4A |           74 | uint16              | Metadata Length  |
|         0x4C |           76 | MediaMetadata\[len] | Metadata         |

- **Title Id**: Title Id of media
- **AUM Id**: Application User Model Id of media
- **Asset Id**: Asset Id
- **Media Type**: See [Media Type](#media-type)
- **Sound Level**: See [Sound Level](#sound-level)
- **Enabled commands**: See [Media Control Command](#media-control-command)
- **Playback status**: See [Media Playback Status](#media-playback-status)
- **Rate**: Playback rate
- **Position**: Current media position (nanoseconds)
- **Media Start**: Media start (nanoseconds)
- **Media End**: Media end (nanoseconds)
- **Min Seek**: Minimal seek position (nanoseconds)
- **Max Seek**: Maximal seek position (nanoseconds)
- **Metadata Length**: Length of Metadata array
- **Metadata**: Array of [Media Metadata](#media-metadata)

#### Sound Level

| Level | Value |
| ----- | ----- |
| Muted | 0x00  |
| Low   | 0x01  |
| Full  | 0x02  |

#### Media Playback Status

| Status   | Value |
| -------- | ----- |
| Closed   | 0x00  |
| Changing | 0x01  |
| Stopped  | 0x02  |
| Playing  | 0x03  |
| Paused   | 0x04  |

#### Media Transport State

| State     | Value |
| --------- | ----- |
| Invalid   | 0x00  |
| Stopped   | 0x01  |
| Starting  | 0x02  |
| Playing   | 0x03  |
| Paused    | 0x04  |
| Buffering | 0x05  |

#### Media Type

| Type         | Value |
| ------------ | ----- |
| NoMedia      | 0x00  |
| Music        | 0x01  |
| Video        | 0x02  |
| Image        | 0x03  |
| Conversation | 0x04  |
| Game         | 0x05  |

#### Media Metadata

| Offset (hex) | Offset (dec) | Type     | Description |
| -----------: | -----------: | -------- | ----------- |
|         0x00 |            0 | SGString | Name        |
|         0x?? |            ? | SGString | Value       |

### Media Controller Removed

**Message Type**: `0xF00`
**Response**: None
**Requests Ack**: `YES`

Informs client about removed media controller.

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint32 | Title Id    |

- **Title Id**: Title Id of removed media controller

### Media Command Result

**Message Type**: `0xF02`
**Response**: None
**Requests Ack**: `YES`

Informs client wether Media Command succeeded.

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint64 | Request Id  |
|         0x08 |            8 | uint32 | Result      |

- **Request Id**: Match with request Id of [Media Command](#media-command)
- **Result**: Result, `0` is success

### Media Command

**Message Type**: `0xF01`
**Response**: [Media Command Result](#media-command-result)
**Requests Ack**: `NO`

Sends media playback command to console.

| Offset (hex) | Offset (dec) | Type   | Description |
| -----------: | -----------: | ------ | ----------- |
|         0x00 |            0 | uint64 | Request Id  |
|         0x08 |            8 | uint32 | Title Id    |
|         0x0C |           12 | uint32 | Command     |

| If Command == Seek
| 0x10         |  16   | uint32      | Seek position

- **Request Id**: Request Id
- **Title Id**: Title Id of media controller
- **Command**: See [Media Control Command](#media-control-command)
- **Seek position (optional)**: Only set if Command == Seek

#### Media Control Command

| Flag              | Bits                  | Mask   |
| ----------------- | --------------------- | ------ |
| Play              | `0000 0000 0000 0010` | 0x02   |
| Pause             | `0000 0000 0000 0100` | 0x04   |
| Play Pause Toggle | `0000 0000 0000 1000` | 0x08   |
| Stop              | `0000 0000 0001 0000` | 0x10   |
| Record            | `0000 0000 0010 0000` | 0x20   |
| Next Track        | `0000 0000 0100 0000` | 0x40   |
| Previous Track    | `0000 0000 1000 0000` | 0x080  |
| Fast Forward      | `0000 0001 0000 0000` | 0x100  |
| Rewind            | `0000 0010 0000 0000` | 0x200  |
| Channel Up        | `0000 0100 0000 0000` | 0x400  |
| Channel Down      | `0000 1000 0000 0000` | 0x800  |
| Back              | `0001 0000 0000 0000` | 0x1000 |
| View              | `0010 0000 0000 0000` | 0x2000 |
| Menu              | `0100 0000 0000 0000` | 0x4000 |
| Seek              | `1000 0000 0000 0000` | 0x8000 |

### Orientation

**Message Type**: `0x33`
**Response**: None
**Requests Ack**: `NO`

Send orientation message to console.

| Offset (hex) | Offset (dec) | Type    | Description           |
| -----------: | -----------: | ------- | --------------------- |
|         0x00 |            0 | uint64  | Timestamp             |
|         0x08 |            8 | float32 | Rotation Matrix Value |
|         0x0C |           12 | float32 | W                     |
|         0x10 |           16 | float32 | X                     |
|         0x14 |           20 | float32 | Y                     |
|         0x18 |           24 | float32 | Z                     |

- **Timestamp**: Timestamp
- **Rotation Matrix Value**: Rotation Matrix Value
- **W**: Value of W-axis
- **X**: Value of X-axis
- **Y**: Value of Y-axis
- **Z**: Value of Z-axis

### Compass

**Message Type**: `0x32`
**Response**: None
**Requests Ack**: `NO`

Send compass message to console.

| Offset (hex) | Offset (dec) | Type    | Description    |
| -----------: | -----------: | ------- | -------------- |
|         0x00 |            0 | uint64  | Timestamp      |
|         0x08 |            8 | float32 | Magnetic North |
|         0x0C |           12 | float32 | True North     |

- **Timestamp**: Timestamp
- **Magnetic North**: Magnetic North
- **True North**: True North

### Inclinometer

**Message Type**: `0x31`
**Response**: None
**Requests Ack**: `NO`

Send inclinometer message to console.

| Offset (hex) | Offset (dec) | Type    | Description |
| -----------: | -----------: | ------- | ----------- |
|         0x00 |            0 | uint64  | Timestamp   |
|         0x08 |            8 | float32 | Pitch       |
|         0x0C |           12 | float32 | Roll        |
|         0x10 |           16 | float32 | Yaw         |

- **Timestamp**: Timestamp
- **Pitch**: Pitch
- **Roll**: Roll
- **Yaw**: Yaw

### Gyrometer

**Message Type**: `0x30`
**Response**: None
**Requests Ack**: `NO`

Send gyrometer message to console.

| Offset (hex) | Offset (dec) | Type    | Description        |
| -----------: | -----------: | ------- | ------------------ |
|         0x00 |            0 | uint64  | Timestamp          |
|         0x08 |            8 | float32 | Angular Velocity X |
|         0x0C |           12 | float32 | Angular Velocity Y |
|         0x10 |           16 | float32 | Angular Velocity Z |

- **Timestamp**: Timestamp
- **Angular Velocity X**: Angular Velocity X-axis
- **Angular Velocity Y**: Angular Velocity Y-axis
- **Angular Velocity Z**: Angular Velocity Z-axis

### Accelerometer

**Message Type**: `0x`
**Response**: None
**Requests Ack**: `NO`

Send accelerometer message to console.

| Offset (hex) | Offset (dec) | Type    | Description    |
| -----------: | -----------: | ------- | -------------- |
|         0x00 |            0 | uint64  | Timestamp      |
|         0x08 |            8 | float32 | Acceleration X |
|         0x0C |           12 | float32 | Acceleration Y |
|         0x10 |           16 | float32 | Acceleration Z |

- **Timestamp**: Timestamp
- **Acceleration X**: Acceleration X-axis
- **Acceleration Y**: Acceleration Y-axis
- **Acceleration Z**: Acceleration Z-axis

### Touch

**Message Type**: `0x2E` (Title) or `0xF2E` (System)
**Response**: None
**Requests Ack**: `NO`

Send touch input message to console.

| Offset (hex) | Offset (dec) | Type               | Description         |
| ------------ | ------------ | ------------------ | ------------------- |
| 0x00         | 0            | uint32             | Touch Msg Timestamp |
| 0x04         | 4            | uint16             |  Touch Count        |
| 0x06         | 6            | Touchpoint\[count] | Touches             |

- **Touch Msg Timestamp**: Timestamp
- **Touch Count**: Number of touchpoints
- **Touches**: Array of [Touchpoint](#touchpoint)

#### Touch Action

| Action | Value |
| ------ | ----- |
| Down   | 0x01  |
| Move   | 0x02  |
| Up     | 0x03  |
| Cancel | 0x04  |

#### Touchpoint

| Offset (hex) | Offset (dec) | Type   | Description       |
| -----------: | -----------: | ------ | ----------------- |
|         0x00 |            0 | uint32 | Touchpoint Id     |
|         0x04 |            4 | uint16 | Touchpoint Action |
|         0x06 |            6 | uint32 | Touchpoint X      |
|         0x0A |           10 | uint32 | Touchpoint Y      |

- **Touchpoint Id**: Id of Touchpoint
- **Touchpoint Action**: See [Touch Action](#touch-action)
- **Touchpoint X**: Touchpoint X-Axis
- **Touchpoint Y**: Touchpoint X-Axis

### Title Launch

**Message Type**: `0x23`
**Response**: None
**Requests Ack**: `YES`

Launch a title / URL on the console.

| Offset (hex) | Offset (dec) | Type     | Description |
| -----------: | -----------: | -------- | ----------- |
|         0x00 |            0 | uint16   | Location    |
|         0x02 |            2 | SGString | Uri         |

- **Location**: Usually `0`
- **Uri**: Uri to launch

### System Text Done

**Message Type**: `0xF35`
**Response**: None
**Requests Ack**: `YES`

| Offset (hex) | Offset (dec) | Type   | Description     |
| -----------: | -----------: | ------ | --------------- |
|         0x00 |            0 | uint32 | Text Session Id |
|         0x04 |            4 | uint32 | Text Version    |
|         0x08 |            8 | uint32 | Flags           |
|         0x0C |           12 | uint32 | Result          |

- **Text Session Id**: Text session id
- **Text Version**: Text version
- **Flags**: Flags
- **Result**: See [Text Result](#text-result)

#### Text Result

| Result | Value |
| ------ | ----- |
| Cancel | 0x00  |
| Accept | 0x01  |

### System Text Acknowledge

**Message Type**: `0xF34`
**Response**: None
**Requests Ack**: `YES`

| Offset (hex) | Offset (dec) | Type   | Description      |
| -----------: | -----------: | ------ | ---------------- |
|         0x00 |            0 | uint32 | Text Session Id  |
|         0x04 |            4 | uint32 | Text Version Ack |

- **Text Session Id**: Text session id
- **Text Version Ack**: Text version to acknowledge

### System Text Input

**Message Type**: `0xF2C`
**Response**: [System Text Acknowledge](#system-text-acknowledge)
**Requests Ack**: `NO`

| Offset (hex) | Offset (dec) | Type        | Description           |
| -----------: | -----------: | ----------- | --------------------- |
|         0x00 |            0 | uint32      | Text Session Id       |
|         0x04 |            4 | uint32      | Base Version          |
|         0x08 |            8 | uint32      | Submitted Version     |
|         0x0C |           12 | uint32      | Total Text bytelength |
|         0x10 |           16 | uint32      | Selection Start       |
|         0x14 |           20 | uint32      | Selection Length      |
|         0x18 |           24 | uint16      | Flags                 |
|         0x1A |           26 | uint32      | Text Chunk bytestart  |
|         0x1E |           30 | SGString    | Text Chunk            |
|         0x?? |           ?? | uint16      | Delta Length          |
|         0x?? |           ?? | Delta\[len] | Text Delta            |

- **Text Session Id**: Text session id
- **Base Version**: Base version
- **Submitted Version**: Submitted version
- **Total Text bytelength**: Total bytelength of text
- **Selection Start**: Selection start
- **Selection Length**: Selection length
- **Flags**: Flags
- **Text Chunk bytestart**: Bytestart of textchunk
- **Text Chunk**: Actual text chunk to send
- **Delta Length**: Count of Text Delta
- **Text Delta**: See [Text Delta](#text-delta)

#### Text Delta

| Offset (hex) | Offset (dec) | Type     | Description    |
| -----------: | -----------: | -------- | -------------- |
|         0x00 |            0 | uint32   | Offset         |
|         0x04 |            4 | uint32   | Delete Count   |
|         0x08 |            8 | SGString | Insert Content |

### Title Text Selection

**Message Type**: `0x21`
**Response**: None
**Requests Ack**: `YES`

| Offset (hex) | Offset (dec) | Type   | Description         |
| -----------: | -----------: | ------ | ------------------- |
|         0x00 |            0 | uint64 | Text Session Id     |
|         0x08 |            8 | uint32 | Text Buffer Version |
|         0x0C |           12 | uint32 | Start               |
|         0x10 |           16 | uint32 | Length              |

- **Text Session Id**: Text session id
- **Text Buffer Version**: Text buffer version
- **Start**: Start
- **Length**: Length

### Title Text Input

**Message Type**: `0x`
**Response**: None
**Requests Ack**: `YES`

| Offset (hex) | Offset (dec) | Type     | Description         |
| -----------: | -----------: | -------- | ------------------- |
|         0x00 |            0 | uint64   | Text Session Id     |
|         0x08 |            8 | uint32   | Text Buffer Version |
|         0x0C |           12 | uint16   | Result              |
|         0x0E |           14 | SGString | Text                |

- **Text Session Id**: Text session id
- **Text Buffer Version**: Text buffer version
- **Result**: See [Text Result](#text-result)
- **Text**: Actual text

### Text Configuration

**Message Type**: `0x1F` (Title) or `0xF2B` (System)
**Response**: None
**Requests Ack**: `YES`

| Offset (hex) | Offset (dec) | Type     | Description         |
| -----------: | -----------: | -------- | ------------------- |
|         0x00 |            0 | uint64   | Text Session Id     |
|         0x08 |            8 | uint32   | Text Buffer Version |
|         0x0C |           12 | uint32   | Text options        |
|         0x10 |           16 | uint32   | Input Scope         |
|         0x14 |           20 | uint32   | Max Text Length     |
|         0x18 |           24 | SGString | Locale              |
|         0x?? |           ?? | SGString | Prompt              |

- **Text Session Id**: Text session id
- **Text Buffer Version**: Text buffer version
- **Text options**: See [Text Option Flags](#text-option-flags)
- **Input Scope**: See [Text Input Scope](#text-input-scope)
- **Max Text Length**: Maximal text length
- **Locale**: Locale to use
- **Prompt**: Text input prompt

#### Text Input Scope

| Scope                    | Value |
| ------------------------ | ----- |
| Default                  | 0x0   |
| Url                      | 0x1   |
| Full FilePath            | 0x2   |
| File Name                | 0x3   |
| Email UserName           | 0x4   |
| Email SmtpAddress        | 0x5   |
| LogOn Name               | 0x6   |
| Personal FullName        | 0x7   |
| Personal NamePrefix      | 0x8   |
| Personal GivenName       | 0x9   |
| Personal MiddleName      | 0xA   |
| Personal Surname         | 0xB   |
| Personal NameSuffix      | 0xC   |
| Postal Address           | 0xD   |
| Postal Code              | 0xE   |
| Address Street           | 0xF   |
| Address StateOrProvince  | 0x10  |
| Address City             | 0x11  |
| Address CountryName      | 0x12  |
| Address CountryShortName | 0x13  |
| Currency AmountAndSymbol | 0x14  |
| Currency Amount          | 0x15  |
| Date                     | 0x16  |
| Date Month               | 0x17  |
| Date Day                 | 0x18  |
| Date Year                | 0x19  |
| Date MonthName           | 0x1A  |
| Date DayName             | 0x1B  |
| Digits                   | 0x1C  |
| Number                   | 0x1D  |
| OneChar                  | 0x1E  |
| Password                 | 0x1F  |
| Telephone Number         | 0x20  |
| Telephone CountryCode    | 0x21  |
| Telephone AreaCode       | 0x22  |
| Telephone LocalNumber    | 0x23  |
| Time                     | 0x24  |
| Time Hour                | 0x25  |
| Time MinorSec            | 0x26  |
| Number FullWidth         | 0x27  |
| Alphanumeric HalfWidth   | 0x28  |
| Alphanumeric FullWidth   | 0x29  |
| Currency Chinese         | 0x2A  |
| Bopomofo                 | 0x2B  |
| Hiragana                 | 0x2C  |
| Katakana HalfWidth       | 0x2D  |
| Katakana FullWidth       | 0x2E  |
| Hanja                    | 0x2F  |
| Hangul HalfWidth         | 0x30  |
| Hangul FullWidth         | 0x31  |
| Search                   | 0x32  |
| Search TitleText         | 0x33  |
| Search Incremental       | 0x34  |
| Chinese HalfWidth        | 0x35  |
| Chinese FullWidth        | 0x36  |
| NativeScript             | 0x37  |

#### Text Option Flags

| Flag                | Bits                  | Mask   |
| ------------------- | --------------------- | ------ |
| Default             | `0000 0000 0000 0000` | 0x00   |
| Accepts Return      | `0000 0000 0000 0001` | 0x01   |
| Password            | `0000 0000 0000 0010` | 0x02   |
| Multi Line          | `0000 0000 0000 0100` | 0x04   |
| Spell Check Enabled | `0000 0000 0000 1000` | 0x08   |
| Prediction Enabled  | `0000 0000 0001 0000` | 0x10   |
| RTL                 | `0000 0000 0010 0000` | 0x20   |
| Dismiss             | `0100 0000 0000 0000` | 0x4000 |
