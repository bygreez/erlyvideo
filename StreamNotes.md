## Introduction ##

Notes of different parts of RTMP and streaming. Random order at the moment.

The information contained in this doc is based on my observations and thus assumptions should be tested and challenged. There is still quite a bit of stuff which is unknown or not fully understood.

## Low Level Protocol ##

First part of this document details the low level structure of the RTMP protocol.

### Handshake ###

TODO: handshake description

### Headers ###

Packet headers have the following structure with sizes ranging from 1 to 14 bytes. Square brackets used to denote optional parts.

```

HeaderType:2, ChannelId:6, [ChannelIdExtra:8|16,] [Timer:24/unsigned, [Size:24, DataType:8, [StreamId:32/little]]]

```

_Header Type_

The header type is encoded in the first 2 bits. Different types denote different header sizes.

```
Type            Size       Value
--------------------------------
New             12 Bytes   00
Same Stream Id   8 Bytes   01
Timer Change     4 Bytes   10
Continue         1 Byte    11

Note: Size does not include possible extra 1-2 bytes taken by larger channel ids.

```

_Channel Id_

The remaining 6 bits of the first byte are used to store the channel id. With a range 0-63. Ids 0 and 1 are reserved to signal higher channels requiring more bytes to store. If 0 then the id is larger than 63 and takes an additional byte ( 64 + extra:8 ). If 1 then the id is larger than 319 and takes an additional byte ( 64 + extra:16 ).

_Timer_

Timer stores the packet time stamp in a 3 byte unsigned medium integer.
For new headers the time stamp is absolute, otherwise its relative.

_Size_

RTMP body size. If the body size is larger than the chunk size (defaults to 128 bytes) then the body will be split into multiple packets. Often the following chunks will have a 1 byte continue header.

_Data Type_

A single byte is used to store the packet data type.
7 of the 20 types are unknown.

```

Chopped from ems.hrl

%% RTMP data 
-define(RTMP_TYPE_CHUNK_SIZE,     1).
%-define(RTMP_TYPE_UNKNOWN,       2).
-define(RTMP_TYPE_BYTES_READ,     3).
-define(RTMP_TYPE_PING,           4).
-define(RTMP_TYPE_BW_SERVER,      5).
-define(RTMP_TYPE_BW_CLIENT,      6).
%-define(RTMP_TYPE_UNKNOWN,       7).
-define(RTMP_TYPE_AUDIO,          8).
-define(RTMP_TYPE_VIDEO,          9).
%-define(RTMP_TYPE_UNKNOWN,      10).
%-define(RTMP_TYPE_UNKNOWN,      11).
%-define(RTMP_TYPE_UNKNOWN,      12).
%-define(RTMP_TYPE_UNKNOWN,      13).
%-define(RTMP_TYPE_UNKNOWN,      14).
-define(RTMP_FLEX_STREAM_SEND,   15).
-define(RTMP_FLEX_SHARED_OBJECT, 16).
-define(RTMP_FLEX_MESSAGE,       17).
-define(RTMP_TYPE_NOTIFY,        18).
-define(RTMP_TYPE_META_DATA,     18).
-define(RTMP_TYPE_SHARED_OBJECT, 19).
-define(RTMP_TYPE_INVOKE,        20).

```

_Stream Id_

The last part of the header is the stream id, this is stored as an little-endian integer. When the client calls createStream the server responds with the next available stream id. I think this is used by the client to direct the media packets to the correct NetStream object. On the server its used to map the incoming packet to the stream object.

## Channels ##

One of the interesting features of RTMP is that it multiplexes data over multiple channels. Each stream has 3 channels, one each for audio, video, and data.

```

Channel Id, Use

0 Medium (1 byte extra) Channel Id
1 Large (2 byte extra) Channel Id
2 Used for pings, stream bytes read, etc.
3 Used for invoke calls.

After this 5 channels are used each stream streams.

4 Stream 1, Data
5 Stream 1, Video
6 Stream 1, Audio
7 Stream 1, Unknown
8 Stream 1, Unknown

9 Stream 2, Data
...

TODO: Improve map packet types to channel ids.

```

### Chunk Size (1) ###

Packets over a configured size (defaults to 128) are split into chunks. Each with its own header.

### Unknown (2) ###

This type is unknown. There was once a report of the player was sending this packet, I asked for more info but nothing was forthcoming.  Perhaps we will find out what its for one day.

### Stream Bytes Read (3) ###

Stream bytes read packets are sent by the client or server when receiving data. When the client is publishing the server must send these packets at regular intervals otherwise the client will close the connection. I think the default for the flash player is to send every 125000 bytes.

### Pings (Protocol Signals) Packets (4) ###

Historically we called them pings, since the first one we discovered was like the standard ping. However in addition to this they also signal the stream state and buffer length. Not all the types are known.

### Server (5) & Client (6) Bandwidth ###

There are two types for setting bandwidth for upstream and downstream.

### Audio (8) & Video (9) Frames ###

Audio and Video frames are sent with timestamps down channels 4 and above. Encoded in the body of the packet is the raw frame (flv tag) data. The frame contains its own header which tells us what kind of audio of video frame it is. Certain types of frames can be dropped to compensate for slow network connections.

TODO: more info on frame types

### Unknown (10-14) ###

These types are unknown.

### Flex Types (15-17) ###

These types are reserved for Flex usage. Not needed for our implementation.

### Notify / MetaData (18) ###

Notify is used for calls which do need or expect a reply. This makes it useful for sending events to the client.

_Net Status Events_

The server sends back status events that the client listens to. You can think of these as a standard layer over the top of the basic RTMP protocol. They do not effect media output on the client directly. For the full list see: ems.hrl.

TODO: describe structure.

_Meta Data_

Stream metadata is sent down the data channel of the stream to the onMetaData method.

### Shared Object (19) ###

Shared objects are used to connect multiple clients to the same object. Any updates made to the properties of the object are sent to all the clients. Also there is a facility to send method calls to all the listening clients. Shared objects have many applications: Chat rooms, multiplayer games, real time data displays, etc.

TODO: List shared object event types.

### Invoke (20) ###

Invoke is used for calls which expect a response.

TODO: describe invoke, args, result handler, etc.

## Higher Level Protocol ##

Now we have covered the low level protocol we can move onto the higher level commands and sequence of events.

### App Commands ###

Clients call methods when they connect and disconnect. But we cannot rely on disconnect being called since sometimes (in the case of dropped network connection) disconnect will not be called. In order to detect dropped connections we can ping the clients at regular intervals to check they are still responding.

```

  * connect

  * disconnect

```

TODO: connect parms

Most of the other commands are stream related (see below), any thing else, is a custom call using NC.call to the application.

### Stream Commands ###

Stream commands (play, pause, seek, etc) are sent to the application. It's the applications responsibility to pass these onto the stream. Think of this as a chain of responsibility. e.g.

RTMP => App [=> PlayList] [=> ServerStream [=> StreamProvider]]

legend: => message passing, [optional](optional.md)

```

createStream() % reserve a stream id (next in sequence)
  -> nextStreamID()

deleteStream(StreamID) % remove the stream and release id

%% following funs know stream id from header

publish(Name, Mode) % modes: record, append, live 

releaseStream(Name) % release a stream (assumed for published streams)

play(Name, Position, Length, FlagFlushPlayList) % all apart from name optional

pause(Flag, Position) % toggle pause, option jump to position

seek(Position) % jump to nearest keyframe

receiveAudio(Flag) % toggle audio

receiveVideo(Flag) % toggle video

closeStream() pass command to stream

```

### Stream Commands Responses ###

I'm just using a made up notation for now, but we can switch to erlang.

TODO: error responses.
TODO: check using debug proxy.

pause

```
signal(PING_STREAM_PLAYBUFFER_CLEAR)
status(NS_PAUSE_NOTIFY)
```

resume

```
signal(PING_STREAM_RESET)
signal(PING_STREAM_CLEAR)
status(NS_UNPAUSE_NOTIFY)
```

play

```
status(NS_PLAY_RESET)
status(NS_PLAY_START)
frame(KeyFrame)
```

stop

```
signal(PING_STREAM_PLAYBUFFER_CLEAR)
status(NS_PLAY_COMPLETE) if end of playlist
status(NS_PLAY_STOP)
```

seek

```
signal(PING_STREAM_PLAYBUFFER_CLEAR)
signal(PING_STREAM_RESET)
signal(PING_STREAM_CLEAR)
status(NS_SEEK_NOTIFY)
status(NS_PLAY_START)
frame(KeyFrame) or frame(BlankAudio)
```


## Architecture Discussion ##

### Push VS Pull ###

### Provider -> Subscriber ###

### FLV Indexing ###


