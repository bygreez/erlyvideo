## Design Ideas ##

Another one of my rather messy design idea documents. This time I have spent quite a bit of time trying to get my head to see the world in an Elrang way. Everything is a process, no shared memory, pure message passing etc. I have tried to break the system down into black boxes.

### Diagrams ###

![http://erlyvideo.googlecode.com/svn/wiki/images/on_demand_streaming.jpg](http://erlyvideo.googlecode.com/svn/wiki/images/on_demand_streaming.jpg)

![http://erlyvideo.googlecode.com/svn/wiki/images/stream_recording.jpg](http://erlyvideo.googlecode.com/svn/wiki/images/stream_recording.jpg)

![http://erlyvideo.googlecode.com/svn/wiki/images/live_streaming.jpg](http://erlyvideo.googlecode.com/svn/wiki/images/live_streaming.jpg)

### Binary Efficiency ###

Efficient use of binaries is critical to the performance of the server. Erlang does not create a new binary for every variable pointing to it, instead it creates a reference and increments a counter. The binary data is stored on a separate heap. Thus there should only need to be one copy of each binary per node. This means providing we don't chop up the binary we can pass it efficiently around the node. The binary will be garbage collected when there are no more references to it. Of course when it comes to passing the binary between processes on different nodes the binary must be sent over the wire. I'm assuming that if you send the same binary in separate messages between nodes that is passed each time. This assumption should be tested**, since if its not the case then the design may change. For the sake of this design I assume its efficient to minimize the number of times a binary is passed between nodes. Reading from memory on the local node should always be the preferred route.**

**Have checked with the erlang list, and the binary will be copied for each message. So it is important to take this into account with our design.**

### Stream FSM ###

In the current design we have a FSM for the clients connection. This deals primarily with the state of the RTMP protocol, namely the transition from the handshake to the connected active state. It also controls playing and recording of streams. IMHO this lumps to separate state machines together. I suggest we separate out the RTMP state and create a separate process to handle the stream with its own state machine. The state transitions would be along the lines of: ready -> playing, playing -> paused, paused -> playing, playing -> seeking, seeking -> playing, playing -> stopped.. etc. There could be separate states for playing of on demand media and receiving of a live broadcast. Since with a live you cannot seek, pause, etc.

### Media Broker ###

When a stream needs to access a media object, it sends its request to the Media Broker. The broker gets a list of the possible sources and works out the cost of access for each source. It then chooses the cheapest (most efficient) option. The cost of access is based on the following formula.

  1. It costs more of read media from disk.
  1. It costs more to access media from another node.
  1. The cost of access on any node depends on that nodes resource usage.

Using this sort of formula we should be able to design the cluster to adapt to changing load with a bit of tweaking.

Another possible action of the broker would be to refuse access via this node, and issue a redirect to another node. AFAIK this is available in the next version of FMS so should be possible to handle at the RTMP level. This would help spread the load across the cluster in situations where the cluster is not behind a load balancer.

### (Cached) Media Source ###

A media source can be thought of as a process which caches part or all of a media object. A media source has the following properties.

  1. Maintains an index of frame types, offsets, and positions.
  1. Maintains a cache of frames in an array.
  1. Random access to frames.
  1. Supports reading of ranges.
  1. Supports actions such as nearest keyframe.
  1. Can be chained to a media reader or another media source.
  1. Can be expired after a certain period of inactivity to free up memory.

### Media Reader and Media Writer ###

Components which read and write media objects to storage.
Convert between the media file format and media frames.
Can be disk or HTTP versions.

### Media Frame ###

A media frame can be video, audio, or data. It contains the header info ( frame\_type, rel\_ts, abs\_ts, size, etc ) and the binary data. I suggest we pre-chunk the data when the media frame is created. This way we can eliminate the need to chop up existing binaries.

### Broadcast Manager ###

The broadcast manager maintains a list of active broadcasts. Think of it like a channel listing. When a stream wants to connect to a broadcast it contacts the manager who looks up the broadcast, returns the Pid for the stream to subscribe. If the broadcast source is not on the save node, it sets up a broadcast repeater.

### Broadcast Source and Broadcast Repeater ###

Streams do not connect directly to the broadcast source if they are not on the same node. Instead a broadcast repeater is connected to the source and only a single stream of binary frames flows between the source and the repeater. This topology will hopefully be more efficient as it should reduce the volume of binary data flowing between the nodes and also support a larger number of subscribers. Consider a cluster with 1000 subscribers connected to a 500kbps stream. If the cluster has 10 nodes and the connections are dispersed between those nodes, thats 100 connections per node. Without repeaters this would be cause 400mbps of data to be sent between the nodes. With repeaters its cut to 2mbps.

### Broadcast Archiver ###

A process you can attach to any broadcast to save the stream to a media writer. The setup could for instance create a new media object for every hour.
