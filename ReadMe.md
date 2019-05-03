## Introduction, Vision, and Goals Of The Project


Around 2.5 exabytes of data is being generated every day. This data is generated through the Internet, Social Media, Communication, Digital Photos, IoT and etc. 

Storing data has evolved during the years in order to accommodate the rising needs of applications using this data. The traditional approach to storage – a standalone, specialized storage system – no longer works, for both technical and economic reasons. 

Distributed Storage System (DSS) is an efficient and economical way of managing data. In distributed storage systems, data is stored on multiple commodity/commercial servers. 

## How does a distributed storage system (DSS) alleviate the current problems?


1- Flexibility and scalability
Distributed storage systems use standard servers. It no longer requires a specialized box, to handle just the storage function. This allows scaling by adding more servers and thus increasing capacity and performance linearly. Also, DSS not only allows to have converged/hyper-converged infrastructure but also allows to keep compute or storage separate on different nodes as well.


2- Speed
In a distributed storage system, any node can read and write in parallel increasing to the overall performance of the storage system compared to a standalone system.


3- Cost
DSS uses standard servers, drives, and network, which are less expensive. In addition, DSS is simpler to manage, which means less staff would be required to run the IT infrastructure.


## Ceph
One of the distributed storage systems is Ceph. it is an open source storage platform, which implements object storage on a single distributed computer cluster, and provides interfaces for object-, block- and file-level storage. Ceph aims primarily for completely distributed operation without a single point of failure, scalable to the exabyte level, and freely available.


Ceph replicates data and makes it fault-tolerant, using commodity hardware and requiring no specific hardware support. As a result of its design, the system is both self-healing and self-managing, aiming to minimize administration time and other costs.


### How does CEPH works

CEPH exposes an interface to the client through a gateway called radosgw (RGW). It layers on top of the Ceph Storage Cluster with its own data formats and maintains its own user database, authentication, and access control. The RGW uses a unified namespace, which means users can use either the OpenStack Swift-compatible API or the Amazon S3-compatible API. For example, the user can write data using the S3-compatible API with one application and then read data using the Swift-compatible API with another application.



<p align="center">
  <img src="presentations/ceph.png" width="350" height="400" title="hover text">
</p>

#### Making Ceph faster
Due to the spatial locality and temporal locality of data, caching and prefetching are effective methods to improve the I/O performance. Prefetching the data and then caching them on the clients can effectively reduce the number of data requests and dramatically cutting down on the latency to access data, thus resulting in overall better quality of service (QoS).

Unfortunately, Ceph does not support caching data. As a result, a team of students in Mass Open Cloud (MOC) designed and developed a new two-layer caching system to make Ceph more efficient. 


## Goal of this Project (Prefetching)
Current caching scheme of CEPH is re-active and data is brought into the cache only when a user sends a request for the particular data. Our goal is to make CEPH-RGW cache more pro-active. We are aiming to implement an interface where depending on the current access pattern RGW can prefetch more data into the cache so future requests can be served from the cache. In this project, we are aiming to implement two types of prefetching

1. User-directed prefetching: A user explicitly tells rgw to bring some "data" into the cache.
2. Automatic prefetching in RGW: RGW prefetches the data into the cache on its own.
* If the time allows, we will also implement a cache monitoring tool.



## Scope and Features Of The Project
To develop a prefetching mechanism which intends to improve the current state of reading data from CEPH. The goal for this project is to provide a mechanism that will complement the current performance of the cache by retrieving the chunks of the accessed file preceding the request for that particular file from the user. These two approaches a cache working standalone or a cache integrated with a prefetching mechanism should be compared and evaluated in terms of performance. In addition, a user interface for system admins should be developed which can help to determine the status of the cache.


## Solution Concept

#### User-Directed Prefetching 
We are hoping to implement user-directed prefetching by overloading the normal GET operation. A user will send a special header in the normal GET request and upon receiving this request, RGW should prefetch the data into the cache and reply the user with success message. RGWOp class, currently implements all the request related functionalities, so we will start from there and modify the current implementation of how the GET request works.
#### Automatic Prefetching in RGW
To implement the automatic prefetching in RGW, we are hoping to modify the current cache implementation. After a request has been served, we will check if it was for the particular byte-range from the file and prefetch the remaining file into the cache. We will implement a new class for prefetching, which will have all of the logic of prefetching and will also have pluggable prefetching-scheme such as ML-based etc. Our main consideration will be that prefetching doesn't affect normal CEPH operations or traffic. 



## Acceptance criteria
An RGW cache coupled with prefetching scheme should improve the overall system throughput as compared to standard CEPH with the cache. Moreover user-directed prefetching should reduce the serving time of an object if it was already brought into the cache.

## Release Planning
Out project timeline is as follows: 
1. Get acquaintance with Ceph, its code, and structure. At the end of this step, we should have good knowledge about the Ceph storage system while having a Ceph system deployed. (Sprint 1)
2. Reading developed caching system code and learn how it does work. (Sprint 2)
3. Designing the prefetching system based on the Ceph and the developed caching system. (Sprint 3)
4. Developing the designed prefetching system on top of the developed caching system. (Sprint 4)
5. Evaluating the implemented prefetching system. (Sprint 5)
6. Developing a mechanism (including an interface) to report the content of the cache to the system admin and the users (Stretch Goal)
