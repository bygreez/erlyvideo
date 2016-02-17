# Introduction #

This ia a quick and dirty ToDo list.


## Overview ##

Main Points
  * FLV Playback
  * FLV Recording
  * Live Streams
  * distributed live streams

lesser issues
  * Add specific folder support into gen\_rtmp applications for playing and recording. Do this with user config file

## Details ##


### Stuart's current Items ###

  * check if ems\_amf:numer\_to\_string/1 is used outside of ems\_amf
  * split ems\_app into ems\_app and ems\_sup
  * separate encode/decoud routines into separate modules

### Roberto's current items ###
  * **URGENT:** make the per-connection id creation work
  * fix audio recording
  * fix live streaming
  * use relative timestamp (needs rewrite of channel creation code)
  * major main-FSM refactoring, now it is just a mess of code
  * add distributed media broker (getting media from Cache, filesystem, network filesystem, S3, etc ..
  * replace mnesia hack with Stuarts well thought approach

cosmetics:
  * make this warning go away: src/ems\_demo.erl:38: Warning: behaviour gen\_rtmp undefined (two step compilation or just surpressing it in Emakefile)