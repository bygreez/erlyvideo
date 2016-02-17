# Optimization #

### Luke's ideas (snipped out from e-mail) ###

Its probably too early for this but I did some rough performance tests on my macbook.. with 1 stream ( transformers from red5 ) the cpu is around 4%, if I open up 10 then that jumps to around 20%. I had a play and managed to get fprof working, it seems to say ( if I understand it correctly ) that most of the cpu time is taken doing file IO. Or at least thats where erlang is waiting. This makes sense, but I would expect the file is paged into memory so access should be fast even for concurrent processes. I have some ideas of things we can to improve things.

  * Increase the chunk size of packets we send to the player, checkout w1-dn.cap to see this being done.

  * Index the original FLV creating another binary index file that can be used for random keyframe/tag lookup. This could be read into memory and instead of calling pread for every tag we can call it once to load a number of chunks at once. I guess you could call this a read ahead buffer.

  * Read the FLV tag data into chunks the same size as we use in RTMP. This will eliminate chunking the tag data as its encoded. We can basically just pass them through to so the binary would not be split between being read and sent.

  * Cache chunks in memory using a MRU cache. This way, if 10 clients all access the same file at the same time we only get 1 set of IO. The cache should be independent of the file access so that it can act as a cache for data coming from other sources such as HTTP. We can have a config parameter to set the max size for the cache.