# Simple Message

- [Simple Message](#simple-message)
  - [General](#general)
  - [Header](#header)
    - [Packet Type](#packet-type)
  - [Power On Request](#power-on-request)
  - [Discovery Packet](#discovery-packet)
    - [Discovery Request](#discovery-request)
      - [Client Type](#client-type)
    - [Discovery Response](#discovery-response)
      - [Certificate](#certificate)
  - [Connect Packet](#connect-packet)
    - [Connect Request](#connect-request)
      - [Public Key Type](#public-key-type)
      - [Fragmenting Request](#fragmenting-request)
      - [Request Group explaination](#request-group-explaination)
    - [Connect Response](#connect-response)
      - [Connect Result](#connect-result)

## General

Simple Messages are used for the most basic tasks in SmartGlass

- Discovering consoles on the network
- Waking up consoles (Power On)
- Connecting to consoles

## Header

| Offset (hex) | Offset (dec) | Type   | Description                |
| -----------: | -----------: | ------ | -------------------------- |
|         0x00 |            0 | uint16 | Packet Type                |
|         0x02 |            2 | uint16 | Unprotected Payload Length |
|         0x04 |            4 | uint16 | \*Protected Payload Length |
| 0x06 or 0x08 |       6 or 8 | uint16 | Version                    |

> NOTE: The header only contains the protected
> payload length-field if packet actually has such
> payload:
>
> SimpleMessage containing protected payload:
>
> - [Connect Request](#connect-request)
> - [Connect Response](#connect-response)

**Packet Type**: See [Packet Type](#packet-type)
**Version**: Usually 2, [Discovery](#discovery-packet) has set this to 0

### Packet Type

| Value  | Description                               |
| :----- | ----------------------------------------- |
| 0xCC00 | [Connect Request](#connect-request)       |
| 0xCC01 | [Connect Response](#connect-response)     |
| 0xDD00 | [Discovery Request](#discovery-request)   |
| 0xDD01 | [Discovery Response](#discovery-response) |
| 0xDD02 |  [Power On Request](#power-on-request)     |

> [Message](message.md) packets (type: `0xD00D`) have a different
> header and are not considered `SimpleMessage`

## Power On Request

**Packet Type**: `0xDD02`
**Response**: None

This packet is sent over broadcast / multicast and directly
to the console's IP address.

- Broadcast (255.255.255.255)
- Multicast (239.255.255.250)
- Specific IP of the console (if available)

If the console is in standby-mode, it will process that
packet and power on.

> NOTE: You may need to send the packet multiple times to make
> sure it is actually received by the console.
>
> Power On request packets only have an `Unprotected payload`

| Offset (hex) | Offset (dec) | Type     | Description |
| -----------: | -----------: | -------- | ----------- |
|         0x00 |            0 | SGString | Live ID     |

## Discovery Packet

Discovery packet, as the name suggests, is used for finding
active consoles on the network.

Same as [Power On Request](#power-on-request), the packet is sent to 3 remote endpoints:

- Broadcast (255.255.255.255)
- Multicast (239.255.255.250)
- Specific IP of the console (if available)

The console will respond by sending a [Discovery Response](#discovery-response) to the sending IP.

> Discovery packets set Header->Version: 0
>
> Discovery packets only have an `Unprotected payload`

### Discovery Request

**Packet Type**: `0xDD00`
**Response**: [Discovery Response](#discovery-response)

Discovers active consoles on the network

| Offset (hex) | Offset (dec) | Type   | Description     |
| -----------: | -----------: | ------ | --------------- |
|         0x00 |            0 | uint32 | Flags           |
|         0x04 |            4 | uint16 | Client Type     |
|         0x06 |            6 | uint16 | Minimum Version |
|         0x08 |            8 | uint16 | Maximum Version |

**Flags**: Unknown, always 0
**Client Type**: See [Client Type](#client-type)
**Minimum Version**: Minimum version to discover: 0
**Maximum Version**: Maximum version to discover: 2

#### Client Type

| Type            | Value |
| --------------- | ----- |
| Xbox One        | 0x01  |
| Xbox 360        | 0x02  |
| Windows Desktop | 0x03  |
| Windows Store   | 0x04  |
| Windows Phone   | 0x05  |
| iPhone          | 0x06  |
| iPad            | 0x07  |
| Android         | 0x08  |

### Discovery Response

**Packet Type**: `0xDD01`
**Response**: None

Console info for the client

| Offset (hex) | Offset (dec) | Type       | Description          |
| -----------: | -----------: | ---------- | -------------------- |
|         0x00 |            0 | uint32     | Primary Device Flags |
|         0x04 |            4 | uint16     | Type                 |
|         0x08 |            8 | SGString   | ConsoleName          |
|         0x?? |            ? | SGString   | UUID                 |
|         0x?? |            ? | uint32     | Last Error           |
|         0x?? |            ? | uint16     | Certificate Length   |
|         0x?? |            ? | char\[len] | Certificate          |

**Primary Device Flags**:
| Flag                      | Bits                  | Mask
\| ------------------------- \| --------------------- \| ------
| Allow Console Users       | `0000 0000 0000 0001` | 0x01
| Allow Authenticated Users | `0000 0000 0000 0010` | 0x02
| Allow Anonymous Users     | `0000 0000 0000 0100` | 0x04
| Certificate Pending       | `0000 0000 0000 1000` | 0x08

**Type**: See [Client Type](#client-type)
**ConsoleName**: Console Name
**UUID**: Console UUID aka Hardware Id
**Last Error**: Last error
**Certificate**: See [Certificate](#certificate)

#### Certificate

Certificate is X509 ASN.1 / DER formatted.

**Fields**:

> **Version**: 3
> **Serial Number**: Console serial number
> **Issuer**: `CN=XBL Smart Glass Issuing CA`
> **Validity**: Validity timespan
> **Subject**: Console's LiveId (`CN=<LiveId>`)
> **Subject Public Key Info**: Console's public key
>
> TIP: You can get a human-readable output for the cert
> via openssl:
>
> `openssl x509 -inform der -in <smartglass certificate> -text -noout`

## Connect Packet

### Connect Request

**Packet Type**: `0xCC00`
**Response**: [Connect Response](#connect-response)

The request is sent from client to console.

Based on the previously received [Discovery Response](#discovery-response), a shared secret is
calculated and the request is encrypted using that (See [Cryptography](cryptography.md)).

##### Public Key Type

Public Key identifier, sent with [Connect Request](#connect-request) packet.

| Type       | Value | Size |
| ---------- | ----- | ---- |
| EC DH P256 | 0x00  | 65   |
| EC DH P384 | 0x01  | 97   |
| EC DH P521 | 0x02  | 133  |

#### Fragmenting Request

When connecting authenticated, the `Authentication data`
(userhash, authorization token) won't fit inside a single packet,
which means the packet will need to be fragmented.

1. Calculate payload length (plaintext unprotected + plaintext
   protected) of assembled [Connect Request](#connect-request) using
   empty `Userhash`("") and `Auth Token`("").
2. Calculate available space by `available space = (1024 - payload length)`
3. Assemble first fragment by copying whole `Userhash` and filling up
   the rest of available space with `Auth Token`.
4. Assemble rest of fragments by just filling them with `Auth Token` chunks.

> NOTE: Make sure to use correct Request (Group) numbering, see
> [Request Group explaination](#request-group-explaination)

**Unprotected Payload**

| Offset (hex) | Offset (dec) | Type      | Description     |
| -----------: | -----------: | --------- | --------------- |
|         0x00 |            0 | byte\[16] | SmartGlass UUID |
|         0x10 |           16 | uint16    | Public Key Type |
|         0x12 |           18 | byte\[??] | Public Key      |
|         0x52 |           82 | byte\[16] | IV              |

**SmartGlass UUID**: Random UUID generated by the client
**Public Key Type**: See [Public Key Type](#public-key-type)
**Public Key**: Public Key, size depends on [Public Key Type](#public-key-type)
**IV**: Initialization Vector of the encrypted packet

**Protected Payload**

> Because strings are of variable length it's not
> possible to give absolute offsets here

| Offset (hex) | Offset (dec) | Type     | Description         |
| -----------: | -----------: | -------- | ------------------- |
|          0x0 |            0 | SGString | Userhash            |
|          0x? |            ? | SGString | Auth Token          |
|          0x? |            ? | uint32   | Request Number      |
|          0x? |            ? | uint32   | Request Group Start |
|          0x? |            ? | uint32   | Request Group End   |

**Userhash**: Xbox Live Userhash (`uhs`)
**Auth Token**: Xbox Live Auth token (`XSTS`)
**Request Number**: Current request number
**Request Group Start**: First request number of this group
**Request Group End**: Last request number of that group **+ 1**

#### Request Group explaination

> **Example**: If you just have a single connect fragment, assume
> request number 0:
>
> Request Number: 0, Request Group Start: 0, Request Group End: 1
>
> Next connect attempt, if needed, would be:
>
> Request Number: 1, Request Group Start: 1, Request Group End: 2
>
> **Example 2**: You have 3 connect fragments, starting again with request number 0:
>
> Fragment #1: Request Number: 0, Request Group Start: 0, Request Group End: 3
>
> Fragment #2: Request Number: 1, Request Group Start: 0, Request Group End: 3
>
> Fragment #3: Request Number: 2, Request Group Start: 0, Request Group End: 3

### Connect Response

**Packet Type**: `0xCC01`
**Response**: None

If the request was formatted and encrypted properly, the console responds
with the following packet:

**Unprotected Payload**

| Offset (hex) | Offset (dec) | Type      | Description |
| -----------: | -----------: | --------- | ----------- |
|         0x00 |            0 | byte\[16] | IV          |

**IV**: Initialization Vector of the encrypted
packet

**Protected Payload**

| Offset (hex) | Offset (dec) | Type   | Description    |
| -----------: | -----------: | ------ | -------------- |
|          0x0 |            0 | uint32 | Connect Result |
|          0x4 |            4 | uint32 | Pairing State  |
|          0x8 |            8 | uint32 | Participant ID |

**Connect Result**: Indicates status of the connect
attempt, see [Connect Result](#connect-result)

**Pairing State**: Pairing state of the client

**Participant Id**: Auto-incremented index given to
the client by the console. Received [Messages](message.md) in the
active session have to match that Id in their `Target
Participant Id` field.

#### Connect Result

| Result  | Value |
| ------- | ----- |
| Success | 0x00  |
| Pending | 0x01  |

| ---- Errors -----
| Unknown                       | 0x02
| Anonymous Connection Disabled | 0x03
| Device Limit Exceeded         | 0x04
| SmartGlass disabled           | 0x05
| User Auth failed              | 0x06
| User SignIn failed            | 0x07
| User SignIn timeout           | 0x08
| User SignIn required          | 0x09
