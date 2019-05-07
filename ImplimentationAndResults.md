# Implimentaion 

## User-Directed Prefetching 
In user directed prefetching we over load the normal GET request with a special prefetch header which is ``prefetch`` for now.
We check for this header in rgw_op.cc ``RGWGetObj::execute()``. 
This call ``s->info.env->get("HTTP_PREFETCH")`` returns the value of the HTTP header. 
If the header is present we set the object length to be 0. One thing to notice here is that uptill this point the the response headers are ready and buffered. So we set the ``total_len = 0;`` as this the content legth header in the http response and flush the buffer to the clinet. Calling the function ``send_response_data(bl, 0, 0);`` with last argument as 0 always flush the buffer. After this client will exit as it has seen the content-length header to be 0.

We also update the ``RGWGetObj::get_data_cb(bufferlist& bl, off_t bl_ofs, off_t bl_len)``. In case the request had the prefetch header, we don't send the data back to the client and returns without doing anything. Also data has already been cached before the call to ``RGWGetObj::get_data_cb(bufferlist& bl, off_t bl_ofs, off_t bl_len)`` in ``rgw_cache.h``.


## RGW-Authomatic Prefetching

### The flow:
When a read request arrives, we check to see if it asks for a whole object or a ``range`` of it. If it is a range, then we calculate what part of it can be prefetched. Based on this information, we create a new request and submit it to the prefetching thread pool. The new thread executes ``run`` function which sends an ``HTTP request``. 
When an HTTP request arrives at RGW (R1), it checks to find out if the data is in ``L1 cache`` or not. If the data is present in L1, R1 returns it. If not, R1 will check which RGW is the ``home node (Rx)``. Then, R1 sends an ``L2 HTTP request`` to Rx. Upon receiving the HTTP request, Rx repeats the same flow as above. 

### Implementation details:
1- ``DataCache::issue_prefetch()`` is the function which creates a ``HttpPrefetchRequest`` and adds it to thread pool. It is called in ``RGWDataCache<T>::iterate_obj()`` function.
2- ``HttpPrefetchRequest::run()`` is the function which submits the http request to the home node.
3- We have implemented ``struct PrefetchReq, class HttpPrefetchRequest`` in ``rgw_cache.h`` file.
4- We added ``obj_size(0)`` argument in ``get_obj_data::get_obj_data(CephContext *_cct)`` function to decide which part of object should be prefetched. It is defined as size_t obj_size in rgw_rados.h, and is  filled ``data->obj_size = *params.obj_size`` in int ``iterate`` function.


# Evaluation 
We stored 5 files of each 500MB in Ceph and then created 10 requests for each file (1 request for each 50 MB byte-range). Then we sent these request to RGW in sequential and random fashion. As expected sequential read had better performance because after we request the first 50MB RGW automatic-prefetching prefetches the remaining 450MB in the cache as well.

<p align="center">
  <img src="presentations/latency.png" width="400" height="300" title="hover text">
  <img src="presentations/throughput.png" width="400" height="300" title="hover text">
</p>

