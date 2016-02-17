## Introduction ##

This document contains notes on running the server and testing.

### Getting the code ###

Get the code using from svn:
```
svn co http://erlyvideo.googlecode.com/svn/trunk erlyvideo
```
### Compiling and configuring the server ###
```
cd erlyvideo
make 
```
### Running the server ###
```
make run
```
This will start the server in an interactive erlang shell.

### Regenerating the documentation ###

'make doc' does currently not work, run from the Erlang shell:
```
edoc:application('ErlyVideo', ".", [no_packages]). 
```
### Applications For Testing ###

There are a number of sample that come with Red5 that can be used to test the server. Initially we are using the echo sample to test calls and AMF datatypes and the publisher application to test streaming. You can get download the apps from.

http://svn1.cvsdude.com/osflash/red5/java/server/trunk/swf/DEV_Deploy/

You can also find some simple samples here.

http://svn1.cvsdude.com/osflash/red5/java/server/trunk/swf/samples/

### Connecting to an application ###

Load up the publisher and connect to rtmp://localhost:1935/ems\_demo

### Changing the code and reloading ###

To compile and changes and reload run this command in the Erlang console.
```
make:all([load]). 
```
### Debugging ###

Currently we debug using trace statements. It would be nice to have the server make binary dumps of the incoming and outgoing data. This could then be read through the decoder to output the events in the console.

### Using RTMP Debug Proxy ###

Hidden in the depths of Red5 is a debug proxy I wrote to aid testing, this can be very useful for debugging the sequence of events. It can be configured to sit between the flash client and the RTMP server and print out all the packets flowing between the two. You can also point it at RTMP servers online to validate behavior and make observations. I do this by redirecting the connection by adding the servers domain in my hosts file then pointing the proxy at the servers IP.

TODO: more info on debug proxy