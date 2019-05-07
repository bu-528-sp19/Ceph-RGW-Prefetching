# Implimentaion 

## User-Directed Prefetching 
In user directed prefetching we over load the normal GET request with a special prefetch header which is ``prefetch`` for now.
We check for this header in rgw_op.cc ``RGWGetObj::execute()``. 
This call ``s->info.env->get("HTTP_PREFETCH")`` returns the value of the HTTP header. 
If the header is present we set the object length to be 0. One thing to notice here is that uptill this point the the response headers are ready and buffered. So we set the ``total_len = 0;`` as this the content legth header in the http response and flush the buffer to the clinet. Calling the function ``send_response_data(bl, 0, 0);`` with last argument as 0 always flush the buffer. After this client will exit as it has seen the content-length header to be 0.

We also update the ``RGWGetObj::get_data_cb(bufferlist& bl, off_t bl_ofs, off_t bl_len)``. In case the request had the prefetch header, we don't send the data back to the client and returns without doing anything. Also data has already been cached before the call to ``RGWGetObj::get_data_cb(bufferlist& bl, off_t bl_ofs, off_t bl_len)`` in ``rgw_cache.h``.


## RGW-Authomatic Prefetching
 Code details 

# Evaluation 
We stored 5 files of each 500MB in Ceph and then created 10 requests for each file (1 request for each 50 MB byte-range). Then we sent these request to RGW in sequential and random fashion. As expected sequential read had better performance because after we request the first 50MB RGW automatic-prefetching prefetches the remaining 450MB in the cache as well.

<p align="center">
  <img src="presentations/latency.png" width="400" height="300" title="hover text">
  <img src="presentations/throughput.png" width="400" height="300" title="hover text">
</p>



# Discussion
