# Ceph-RGW-Prefetching

## Introduction, Vision, and Goals Of The Project

The amount of produced data every day is around 2.5 exabyte of data. This data is generated through the Internet, Social Media, Communication, Digital Photos, IoT and etc. The pace of generating data is only accelerating with the growth of using various digital services. Over the last two years alone 90 percent of the data in the world was generated.

Storing data has evolved during the years in order to accommodate the rising needs of companies and individuals. The traditional approach to storage – a standalone, specialized storage system – no longer works, for both technical and economic reasons. 

The pace of generating data not only needs faster drives and networks, but it also needs a new approach to store data. Distributed Storage System (DSS) is a storing method which solves most of the present problems. We can define distributed data storage as “Storing data on a multitude of standard servers, which behave as one storage system although data is distributed between these servers.”

A distributed storage system can relate to any of the 3 types of storage: block, file, and object. In the case of block-level storage systems “distributed data storage” typically relates to one storage system in a tight geographical area, usually located in one data center, since performance demands are very high. In the case of object-storage systems – they can be both in one location or more locations and here geographically a distributed storage system could work, as the requirements on performance are not as high as for block-level storage. File storage falls in between, depending on the workload the user of the system is running.


## How does a distributed storage system alleviate the current problems?

### 1- Flexibility and scalability
Distributed storage systems use standard servers. It no longer requires a specialized box, to handle just the storage function. This allows scaling by adding more servers and thus increasing capacity and performance linearly. Also, DSS not only allows to have converged/hyper-converged infrastructure, but also allows to keep compute or storage separate on different nodes as well.

### 2- Speed
In a distributed storage system, any node can read and write in parallel increasing to overall performance of the storage system comparing to a standalone system.

### 3- Cost
 DSS uses standard servers, drives, and network, which are less expensive. In addition, DSS is simpler to manage, which means less staff would be required to run the IT infrastructure.

## Ceph
One of the distributed storage systems is Ceph. it is an open source storage platform, implements object storage on a single distributed computer cluster, and provides interfaces for object-, block- and file-level storage. Ceph aims primarily for completely distributed operation without a single point of failure, scalable to the exabyte level, and freely available.

Ceph replicates data and makes it fault-tolerant, using commodity hardware and requiring no specific hardware support. As a result of its design, the system is both self-healing and self-managing, aiming to minimize administration time and other costs.




### Ceph Object Storage
The Ceph Object Storage daemon, radosgw (RGW), is a FastCGI service that provides a RESTful HTTP API to access the data. It layers on top of the Ceph Storage Cluster with its own data formats, and maintains its own user database, authentication, and access control. The RGW uses a unified namespace, which means users can use either the OpenStack Swift-compatible API or the Amazon S3-compatible API. For example, the user can write data using the S3-compatible API with one application and then read data using the Swift-compatible API with another application.

### Making Ceph faster
Due to the spatial locality and temporal locality of data, caching and prefetching are effective methods to improve the I/O performance. Prefetching the data and then caching them on the clients can effectively reduce the number of data requests and dramatically cutting down on the latency to access data, thus resulting in an overall better quality of service (QoS).

Unfortunately, Ceph does not support caching data. As a result, a team of students in Mass Open Cloud (MOC) designed and developed a new two-layer caching system to make Ceph more efficient. This system deploys caching system in RGW machine improving overall Ceph performance.

## Goals of this Project
The natural step after developing the caching system for Ceph is to develop prefetching mechanism. Since the majority of data has spatial locality, prefetching next (sequentially located) data can increase the Ceph performance. In this project, we will develop:
- A simple prefetching system for Ceph. This system will figure out which file is accessed and will prefetch the remaining parts of the file before actual request. By the time of receiving the request for the remaining parts, the data is ready and therefore the user wait time will be reduced. This system will be a part of upstream Ceph code.
- A mechanism to evaluate the overall performance of Ceph while the prefetching system is in place. We will design and deploy a scenario to find out how good the new prefetching system is.
- A system to check the content of cache. It would be very helpful for system administrator and users (if they had the access right) to check the content of cache and analyze the access pattern. This part of project can help to design a more intelligent caching and prefetching systems.

The final implementation will be a part of Ceph project through upstream process. As a result, the team will follow production level coding as much as possible. In this manner, the coding quality should be acceptable by the open source community.


## Users/Personas Of The Project
User - Human - Users that wish to store a file in a distributed storage system.

Developer - Human - A developer that handles the design of the prefetching mechanism and ensures its streamliness with the current caching system.

Prefetching mechanism - Nonhuman - Synchronizes the remaining parts of the accessed file to the cache making available per request.

User interface - Nonhuman - A graphical interface to allow system admins to view the status of the cache.





## Scope and Features Of The Project
To develop a prefetching mechanism which intends to improve the current state of reading files from DSS. The goal for this project is to provide a mechanism that will complement the current performance of the cache by retrieving the chunks of the accessed file preceding the request operation for that particular file. These two approaches a cache working standalone or a cache integrated with a prefetching mechanism should be compared and evaluated in terms of performance. In addition, a more interactive interface for system admins should be developed which determines the status of the cache.

## Solution Concept

CEPH provides end-user REST API to store, retrieve and update data. This API used RADOS gateway to interact with the storage clusters. And after discussing with mentors we realized that this would be the best place to implement prefetching. We are planning to implement prefetching in the following way; 

- Intercept the incoming request 
- Compute the IDs of the next blocks
- Fetch the data predicted in 2
- Update the cache records/lookup tables

To achieve the above, we first need to have a fine understanding of the source code of the RADOS gateway that implements the storage/retrieves data from the storage. 


## Acceptance criteria

Adding a prefetching system should increase the overall performance of Ceph storage. Since cache space is limited and valuable, prefetching wrong data can result in wasting cache space and eventually degradation of the caching system. 
We argue that spatial locality is true for the majority of datasets but not for all of them. Therefore, we should see a higher performance for the majority of applications, however, few applications may have a worse performance with prefetching comparing to having an only caching system. 

## Release Planning
To finish the project, we consider the following steps:
- Get acquaintance with Ceph, its code, and structure. At the end of this step, we should have a good knowledge about Ceph storage system while having a Ceph system deployed.
- Reading developed caching system code and learn how it does work.
- Designing the prefetching system based on the Ceph and the developed caching system.
- Developing the designed prefetching system on top of the developed caching system.
- Evaluating the implemented prefetching system.
- Developing a mechanism (including an interface) to report the content of the cache to the system admin and the users.






