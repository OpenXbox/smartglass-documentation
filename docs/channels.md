# Channels

There are messages and status reports which service a specific area, they
communicate on their own Channel (indicated by the [Service Channel](#service-channels) field
in the message header).

All channel communication is of type [Message](message.md).

- [Channels](#channels)
  - [Core Channel](#core-channel)
  - [Acknowledging Messages](#acknowledging-messages)
  - [Service Channels](#service-channels)
    - [Acquiring a channel](#acquiring-a-channel)
    - [Input Channel](#input-channel)
    - [Input TV Remote Channel](#input-tv-remote-channel)
      - [Stump Message Type](#stump-message-type)
      - [Message Id Generation](#message-id-generation)
      - [Assemble HTTP Streaming URL](#assemble-http-streaming-url)
      - [HDMI GUID](#hdmi-guid)
      - [Stump Source](#stump-source)
      - [Stump Quality](#stump-quality)
      - [Stump Notification](#stump-notification)
      - [Stump Filter Type](#stump-filter-type)
      - [Stump Base Message](#stump-base-message)
        - [Request](#request)
        - [Response](#response)
      - [Get Configuration](#get-configuration)
      - [Get Headend Info](#get-headend-info)
      - [Get Live TV Info](#get-live-tv-info)
      - [Get Program Info](#get-program-info)
      - [Get Tuner Lineups](#get-tuner-lineups)
      - [Get AppChannel Lineups](#get-appchannel-lineups)
      - [Get AppChannel Program Data](#get-appchannel-program-data)
      - [Get AppChannel Data](#get-appchannel-data)
      - [Ensure Streaming started](#ensure-streaming-started)
      - [Set Channel Params](#set-channel-params)
      - [Get recent Channels](#get-recent-channels)
      - [Send Key](#send-key)
    - [Media Channel](#media-channel)
    - [Text Channel](#text-channel)
    - [Broadcast Channel](#broadcast-channel)
      - [How to start gamestreaming?](#how-to-start-gamestreaming)
      - [Broadcast Message Type](#broadcast-message-type)
      - [Messages](#messages)
        - [Gamestream Start Message](#gamestream-start-message)
        - [Gamestream Stop Message](#gamestream-stop-message)
        - [Gamestream State Message](#gamestream-state-message)
          - [Gamestream State](#gamestream-state)
          - [Initializing](#initializing)
          - [Started](#started)
          - [Stopped](#stopped)
          - [Paused](#paused)
        - [Gamestream Enabled Message](#gamestream-enabled-message)
        - [Gamestream Error Message](#gamestream-error-message)
          - [Gamestream Error](#gamestream-error)
        - [Gamestream Telemetry Message](#gamestream-telemetry-message)
        - [Gamestream Preview Status Message](#gamestream-preview-status-message)

## Core Channel

Core Channel is `0`.

## Acknowledging Messages

Console sends packets of type [Acknowledgement](message.md#acknowledgement)
on channel `0x1000000000000000`, Client sends `Acks` on [Core Channel](#core-channel).

## Service Channels

Click on the channel in the following table to get more information.

| Channel                                         | GUID                                  |
| ----------------------------------------------- | ------------------------------------- |
| [SystemInput](#input-channel)                   | `fa20b8ca-66fb-46e0-adb60b978a59d35f` |
| [SystemInputTVRemote](#input-tv-remote-channel) | `d451e3b3-60bb-4c71-b3dbf994b1aca3a7` |
| [SystemMedia](#media-channel)                   | `48a9ca24-eb6d-4e12-8c43d57469edd3cd` |
| [SystemText](#text-channel)                     | `7af3e6a2-488b-40cb-a93179c04b7da3a0` |
| [SystemBroadcast](#broadcast-channel)           | `b6a117d8-f5e2-45d7-862e8fd8e3156476` |

### Acquiring a channel

After [Local Join](message.md#local-join) is done successfully, client
can start additional [Service Channel](#service-channels).

**Example**
The following messaging happens on [Core Channel](#core-channel).

1. Start with `Request Id` 0
2. Client sends [Channel Start Request](message.md#channel-start-request) with
   current `Request Id` and [Target Service Channel GUID](#service-channels)
   to console
3. Console replies with [Channel Start Response](message.md#channel-start-response)
4. Make sure `Request Id` in response matches the request
5. Check result code of response, has to be `0` for success
6. Use `Target Channel Id`, delivered in the response, to communicate
   with that channel
7. ... Send next [Channel Start Request](message.md#channel-start-request) with
   incremented `Request Id` ...

### Input Channel

Control console via gamepad controls, touch, accelerometer, gyroscope etc.

Used messages:

- [Gamepad](message.md#gamepad)
- [Title Touch](message.md#touch)
- [System Touch](message.md#touch)
- [Accelerometer](message.md#accelerometer)
- [Gyrometer](message.md#gyrometer)
- [Inclinometer](message.md#inclinometer)
- [Compass](message.md#compass)
- [Orientation](message.md#orientation)

### Input TV Remote Channel

Also known as `Stump`, controls configured infrared devices via Xbox's IR Blaster
and makes it possible to stream LiveTV and maybe HDMI IN.
Communication is completely [JSON](message.md#json)-based.

#### Stump Message Type

| Name                        | String Value             |
| --------------------------- | ------------------------ |
| Error                       | Error                    |
| Ensure Streaming Started    | EnsureStreamingStarted   |
| Get Configuration           | GetConfiguration         |
| Get Headend Info            | GetHeadendInfo           |
| Get LiveTV Info             | GetLiveTVInfo            |
| Get Program Info            | GetProgrammInfo          |
| Get Recent Channels         | GetRecentChannels        |
| Get Tuner Lineups           | GetTunerLineups          |
| Get AppChannel Data         | GetAppChannelData        |
| Get AppChannel Lineups      | GetAppChannelLineups     |
| Get AppChannel Program Data | GetAppChannelProgramData |
| Send Key                    | SendKey                  |
| Set Channel                 | SetChannel               |

#### Message Id Generation

An example of a messageid (Android client): `7edcf9d0.14`.

- Generate a random int, called `Message Id Prefix`, from `0` to `0x7FFFFFFF`.
- Initialize a `Message Id Index` with `1`
- Concat both values, splitted by `.`: Prefix: `7edcf9d0`, Index: `1` => `7edcf9d0.1`
- For each new request that's sent to the console, `Message Id Index` gets
  incremented: `7edcf9d0.2`, `7edcf9d0.3`

> NOTE: AppChannel stuff uses a **GUID** as `Message Id`

#### Assemble HTTP Streaming URL

TODO

#### HDMI GUID

`BA5EBA11-DEA1-4BAD-BA11-FEDDEADFAB1E`

#### Stump Source

| Name         | String Value |
| ------------ | ------------ |
| HDMI         | hdmi         |
| USB TV Tuner | tuner        |

#### Stump Quality

| Name           | String Value |
| -------------- | ------------ |
| Low Quality    | low          |
| Medium Quality | medium       |
| High Quality   | high         |
| Best Quality   | best         |

#### Stump Notification

| Name                  | String Value         |
| --------------------- | -------------------- |
| Streaming Error       | StreamingError       |
| Channel Changed       | ChannelChanged       |
| Channel Type Changed  | ChannelTypeChanged   |
| Configuration Changed | ConfigurationChanged |
| Device UI Changed     | DeviceUIChanged      |
| Headend Changed       | HeadendChanged       |
| Video Format Changed  | VideoFormatChanged   |
| Program Changed       | ProgrammChanged      |
| Tunerstate Changed    | TunerStateChanged    |

#### Stump Filter Type

| Name      | String Value |
| --------- | ------------ |
| All       | ALL          |
| HD and SD | HDSD         |
| Only HD   | HD           |

#### Stump Base Message

##### Request

Direction: _Client -> Console_

```json
{
    "request": "Stump Message Type goes here",
    "msgid": "Generated Message Id goes here",
    "params": "Either null/None or some other parameters"
}
```

##### Response

Direction: _Console -> Client_

**Regular response**

```json
{
    "response": "Same as request value",
    "msgid": "Same as request value",
    "params": "Requested data"
}
```

**Notification**

```json
{
    "notification": "notification value",
    // Maybe some other value?
}
```

**Error**

```json
{
    "response": "Error",
    "msgid": "Same as request value",
    "error": "Some error text"
}
```

#### Get Configuration

Value: `GetConfiguration`

**Request Parameters**
`None`

**Response Parameters**

```json
[
    {
        "device_id": "0",
        "device_type": "tv",
        "buttons": {
            "btn.back": "Back",
            "btn.up": "Up",
            "btn.red": "Red",
            // and more...
        }
    },
    {
        "device_id": "1",
        "device_type": "stb",
        "buttons": {}
    },
    {
        // ...
        "device_type": "tuner"
    }
]
```

#### Get Headend Info

Value: `GetHeadendInfo`

**Request Parameters**
`None`

**Response Parameters**

```json
{
    "headendId": "dbd2530a-fcd5-8ff0-b89d-20cd7e021502",
    "providerName": "Sky Deutschland",
    "headendLocale": "de-DE",
    "providers": [
        {
            "headendId": "DBD2530A-FCD5-8FF0-B89D-20CD7E021502",
            "providerName": "Sky Deutschland",
            "source": "hdmi",
            "titleId": "162615AD",
            "filterPreference": "HDSD",
            "canStream": "false"
        },
        {
            "headendId": "D130DB0D-6CBC-328C-A30F-A514C0D0F377",
            "providerName": "Unity Media",
            "source": "tuner",
            "titleId": "162615AD",
            "filterPreference": "ALL",
            "canStream": "true"
        }
    ],
    "blockExplicitContentPerShow": false,
    "dvrEnabled": false
}
```

#### Get Live TV Info

Value: `GetLiveTVInfo`

**Request Parameters**
`None`

**Response Parameters**

```json
{ "inHdmiMode": "1" }
```

#### Get Program Info

Value: `GetProgrammInfo`

**Request Parameters**
`None`

**Response Parameters**

```json
// Unknown
```

#### Get Tuner Lineups

Value: `GetTunerLineups`

**Request Parameters**
`None`

**Response Parameters**

```json
{
    "providers":[
        {
        "headendId":"D130DB0D-6CBC-328C-A30F-A514C0D0F377",
        "cqsChannels":[

        ],
        "foundChannels":[

        ]
        }
    ]
}
```

#### Get AppChannel Lineups

Value: `GetAppChannelLineups`

**Request Parameters**
`None`

**Response Parameters**

```json
[
    {
        "id":"LiveTvHdmiProvider",
        "providerName":"OneGuide",
        "primaryColor":"ff107c10",
        "secondaryColor":"ffebebeb",
        "titleId":"0BB0AAE5",
        "channels":[

        ]
    },
    {
        "id":"LiveTvPlaylistProvider",
        "providerName":"OneGuide",
        "primaryColor":"ff107c10",
        "secondaryColor":"ffebebeb",
        "titleId":"0BB0AAE5",
        "channels":[
        {
            "id":"PromoAppChannel",
            "name":"More app channels"
        }
        ]
    },
    {
        "id":"LiveTvUsbProvider",
        "providerName":"OneGuide",
        "primaryColor":"ff107c10",
        "secondaryColor":"ffebebeb",
        "titleId":"0BB0AAE5",
        "channels":[
        ]
    }
]
```

#### Get AppChannel Program Data

Value: `GetAppChannelProgramData`

**Request Parameters**

```json
{
    "providerId": "provider id goes here",
    "programId": "program id goes here"
}
```

**Response Parameters**

```json
// Unknown
```

#### Get AppChannel Data

Value: `GetAppChannelData`

**Request Parameters**

```json
{
    "providerId": "provider id goes here",
    "channelId": "channel id goes here",
    "id": "channel id again"
}
```

**Response Parameters**

```json
// Unknown
```

#### Ensure Streaming started

Value: `EnsureStreamingStarted`

**Request Parameters**

```json
{ "source": "stump source goes here"}
```

**Response Parameters**

```json
// Unknown
```

#### Set Channel Params

Value: `SetChannel`

**Request Parameters**
By Id:

```json
{ "channelId": "target Channel Id", "lineupInstanceId": "target Lineup Id" }
```

By Name:

```json
{ "channel_name": "target channel name" }
```

**Response Parameters**

```json
// Unknown
```

#### Get recent Channels

Value: `GetRecentChannels`

**Request Paramters**

```json
{ "startindex": 0, "count": 50 }
```

**Response Parameters**

```json
[ "list", "of", "channels"]
```

#### Send Key

Value: `SendKey`

**Request Parameters**

```json
{
    "button_id": "button id string",
    // optional:
    // "device_id": "target device id"
}
```

**Response Parameters**
`True` on success, `False` on error

### Media Channel

Get info about currently playing media and control the playback.

Used messages:

- [Media Controller Removed](message.md#media-controller-removed)
- [Media Command](message.md#media-command)
- [Media Command Result](message.md#media-command-result)
- [Media State](message.md#media-state)

### Text Channel

Input Text whenever a text-dialog is shown on Xbox.

Used messages:

- [Title Text Configuration](message.md#text-configuration)
- [Title Text Input](message.md#title-text-input)
- [Title Text Selection](message.md#title-text-selection)
- [System Text Configuration](message.md#text-configuration)
- [System Text Input](message.md#system-text-input)
- [System Text Acknowledge](message.md#system-text-acknowledge)
- [System Text Done](message.md#system-text-done)

### Broadcast Channel

Initializing of Gamestreaming is done over this channel.
Communication is completely [JSON](message.md#json)-based.
Actual streaming is done over [Nano](nano.md).

Messages actively sent from client to console:

- [Gamestream Start Message](#gamestream-start-message)
- [Gamestream Stop Message](#gamestream-stop-message)

The other messages are basically status messages sent from console to client.

#### How to start gamestreaming?

1. Open the [SystemBroadcast](#acquiring-a-channel) channel.
2. Console sends a [Gamestream Enabled Message](#gamestream-enabled-message).
3. Check the field `enabled` if it's possible to use Gamestreaming (e.g. if it's activated).
4. Send a [Gamestream Start Message](#gamestream-start-message) to the console.
5. Console sends some [Gamestream State Messages](#gamestream-state-message).
6. Get the connection data from [State: Initializing](#initializing).
7. Use [Nano](nano.md) to connect to the console.

#### Broadcast Message Type

| Type                                                 | Value | Sent by |
| ---------------------------------------------------- | ----- | ------- |
| [Start Gamestream](#gamestream-start-message)        | 0x01  | Client  |
| [Stop Gamestream](#gamestream-stop-message)          | 0x02  | Client  |
| [Gamestream State](#gamestream-state-message)        | 0x03  | Console |
| [Gamestream Enabled](#gamestream-enabled-message)    | 0x04  | Console |
| [Gamestream Error](#gamestream-error-message)        | 0x05  | Console |
| [Telemetry](#gamestream-telemetry-message)           | 0x06  | Console |
| [Preview Status](#gamestream-preview-status-message) | 0x07  | Console |

#### Messages

##### Gamestream Start Message

Type: `0x1`

Direction: _Client -> Console_

> NOTE: Some values _could_ be int or bool rather than string
> but the following example is the original format that gets sent
> from the original Windows 10 client

```json
{
    "type": 1,
    "configuration": {
        "urcpType": "0",
        "urcpFixedRate": "-1",
        "urcpMaximumWindow": "1310720",
        "urcpMinimumRate": "256000",
        "urcpMaximumRate": "10000000",
        "urcpKeepAliveTimeoutMs": "0",
        "audioFecType": "0",
        "videoFecType": "0",
        "videoFecLevel": "3",
        "videoPacketUtilization": "0",
        "enableDynamicBitrate": "false",
        "dynamicBitrateScaleFactor": "1",
        "dynamicBitrateUpdateMs": "5000",
        "sendKeyframesOverTCP": "false",
        "videoMaximumWidth": "1280",
        "videoMaximumHeight": "720",
        "videoMaximumFrameRate": "60",
        "videoPacketDefragTimeoutMs": "16",
        "enableVideoFrameAcks": "false",
        "enableAudioChat": "true",
        "audioBufferLengthHns": "10000000",
        "audioSyncPolicy": "1",
        "audioSyncMinLatency": "10",
        "audioSyncDesiredLatency": "40",
        "audioSyncMaxLatency": "170",
        "audioSyncCompressLatency": "100",
        "audioSyncCompressFactor": "0.99",
        "audioSyncLengthenFactor": "1.01",
        "enableOpusAudio": "false",
        "enableOpusChatAudio": "true",
        "inputReadsPerSecond": "120",
        "udpMaxSendPacketsInWinsock": "250",
        "udpSubBurstGroups": "0",
        "udpBurstDurationMs": "12"
    },
    "reQueryPreviewStatus": false
}
```

##### Gamestream Stop Message

Type: `0x2`

Direction: _Client -> Console / never seen in the wild_

```json
{
    "type": 2
}
```

##### Gamestream State Message

Type: `0x3`

Direction: _Console -> Client_

All gamestream state messages use a `state` field,
declared in the following table.

###### Gamestream State

| State        | Value |
| ------------ | ----- |
| Initializing | 0x01  |
| Started      | 0x02  |
| Stopped      | 0x03  |
| Paused       | 0x04  |

###### Initializing

```json
{
    "type": 3,
    "state": 1,
    "sessionId": "{123E4567-E89b-12D3-A456-426655440000}",
    "udpPort": 55068,
    "tcpPort": 52556
}
```

###### Started

```json
{
    "type": 3,
    "state": 2,
    "sessionId": "{123E4567-E89b-12D3-A456-426655440000}",
    "isWirelessConnection": false,
    "wirelessChannel": 0,
    "transmitLinkSpeed": 1000000000
}
```

###### Stopped

```json
{
    "type": 3,
    "state": 3,
    "sessionId": "{123E4567-E89b-12D3-A456-426655440000}"
}
```

###### Paused

```json
// Never seen in the wild
```

##### Gamestream Enabled Message

Type: `0x4`

Direction: _Console -> Client_

```json
{
    "type": 4,
    "enabled": true,
    "canBeEnabled": true,
    "majorProtocolVersion": 6,
    "minorProtocolVersion": 0
}
```

##### Gamestream Error Message

Type: `0x5`

Direction: _Console -> Client_

```json
{
    "type": 5,
    "errorType": 1,
    "errorValue": 1234
}
```

###### Gamestream Error

| Error                   | Value |
| ----------------------- | ----- |
| General                 | 0x01  |
| Failed to instantiate   | 0x02  |
| Failed to initialize    | 0x03  |
| Failed to start         | 0x04  |
| Failed to stop          | 0x05  |
| No Controller           | 0x06  |
| Different MSA active    | 0x07  |
| DRM Video               | 0x08  |
| HDCP Video              | 0x09  |
| Kinect Title            | 0x0A  |
| Prohibited Game         | 0x0B  |
| Poor network connection | 0x0C  |
| Streaming disabled      | 0x0D  |
| Cannot reach console    | 0x0E  |
| Generic Error           | 0x0F  |
| Version mismatch        | 0x10  |
| No profile              | 0x11  |
| Broadcast in progress   | 0x12  |

##### Gamestream Telemetry Message

Type: `0x6`

Direction: _Unknown / never seen in the wild_

```json
{
    "type": 6
}
```

##### Gamestream Preview Status Message

Type: `0x7`

Direction: _Console -> Client_

```json
{
    "type": 7,
    "isPublicPreview": false,
    "isInternalPreview": false
}
```
