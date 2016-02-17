# Summary #

This is an ongoing list of the items that need to be considered during the refactoring of ErlyViveo/ErlMedia.

  * Polish off and test the foundation modules. FLV parsing, RTMP parsing, AMF parsing, etc.
  * Commit FLV code for review. Make any suggested changes.
  * Decouple the RTMP module from the FSM.
  * Refactor the RTMP module so that its only concerned with encoding and decoding. It shouldn't try to handle the commands.
  * Review RTMP module and complete any remaining bits needed.
  * Rename a few things to be more logical names. e.g. amf record is really a call.
  * Write some tests to validate things work as they should.
  * Discuss server design
  * Implement and test performance
  * Scale to multiple nodes
  * Shared Objects. Erlang will really shine with shared object.
  * RTMPT: RTMP tunneled over HTTP using polling. Once you have RTMP working, this is quite straight forward to add.  We can use mochiweb.
  * RTMPS: secure RTMP, this just uses SSL.
  * RTMPE: still an unknown FMS3 is not out yet. But I'm curious. I think its a way of improving the security.
  * ErlWare integration?
  * ems\_proxy for testing and debugging code

