## Introduction ##

The file ems\_proxy.erl is a TCP server that forwards packets on to another server, allowing for those packets to be captured and processed by the RTMP and AMF encode/decode rotines. It also allows for the DUMP files to be saves to examine differences between the respones on different servers.

## Basic Operation ##

The Debug proxy is started by using one of three funcations start\_link/1, start\_link/2 or startlink/3.

@spec ()