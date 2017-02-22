---
layout: post
title: "Building Scalable Distributed Systems - Pt. 2"
description: ""
category: distributed
tags: [distributed, scalability]
comments: True
---

Let's assume we want to build something that grows very much: the first thing 
we need to consider is to understand how design our application: software 
engineering patterns are not the focus of this post, so we simply thinking 
about \[Micro\]services.

Generally speaking, we can consider a generic {Micro|service} like a block 
that helps to decouple functionality and think about each part of the system 
as its own service with a clearly defined interface (I hate this definition 
because it reminds me the JavaWorld but it makes things more clear).

In the real world, we started thinking about microservices since the SOA (Service 
Oriented Architectures) were born: they are pieces of software that are components 
of the big picture, they have to work to make other pieces of software able to work, 
cooperating in a complementary way that helps to decouple the operation of those pieces 
from one another. 

This abstraction help us to establish clear relationships between the service, its 
underlying environment, and the consumers of that service.
Okay, now it's time to go back to an example: to keep things simple, we suppose that 
all requests to upload and retrieve images are processed by the same server; however, 
as the system needs to scale it makes sense to break out these two functions into their 
own services.

Flash forward: assume that the service is in heavy use; this scenario makes it easy 
to see how longer writes will impact the time it takes to read the images.
Depending on the architecture this effect can be substantial. Even
if the upload and download speeds are the same (which is not true of most IP networks, since most
are designed for at least a 3:1 download-speed:upload-speed ratio), read files will typically be
read from cache, and writes will have to go to disk eventually (and perhaps be written several
times in eventually consistent situations). 
Even if everything is in memory or read from disks (like SSDs), database writes will almost always 
be slower than reads.


Another potential problem with this design is that a classic Web server like Apache or lighttpd
typically has an upper limit on the number of simultaneous connections it can 
maintain and in high traffic, writes can quickly consume all of those. 
Since reads can be asynchronous, or take advantage of other performance optimizations
like gzip compression or chunked transfer encoding, the Web server can switch serve reads faster
and switch between clients quickly serving many more requests per second than the max number of
connections (with Apache and max connections set to 500, it is not uncommon to serve several
thousand read requests per second). 

Writes, on the other hand, tend to maintain an open connection for the duration for the upload, 
so uploading a 1MB file could take more than 1 second on most home networks, so that Web server 
could only handle 500 such simultaneous writes.



Of course, splitting writes on a sharded environment allows us to 
scale horizontally each of them independently, which would make it easier to 
troubleshoot and scale a problem like slow reads.

For example, Flickr solves this read/write issue by distributing users across 
different shards such that each shard can only handle a set number of users, 
and as users increase more shards are added to the cluster 
(see the presentation on 
[Flickr's scaling architecture](http://highscalability.com/flickr-architecture)).



Redundancy
====

Creating redundancy in a system can remove single points of failure and provide 
a backup or spare functionality if needed in a crisis.
Another key part of service redundancy is creating a shared-nothing architecture. 
With this architecture, each node is able to operate independently of one another 
and there is no central "brain" managing state or coordinating activities for the 
other nodes. This helps a lot with scalability since new nodes can be added without 
special conditions or knowledge.

Partitions
====

Scaling vertically means adding more resources to an individual server. So for 
a very large data set, this might mean adding more hard drives so a single server 
can contain the entire data set. 
In each case, vertical scaling is accomplished by making the individual resource 
capable of handling more on its own.

To scale horizontally, on the other hand, is to add more nodes. In the case of the 
large data set, this might be a second server to store parts of the data set.

When it comes to horizontal scaling, one of the more common techniques is to break 
up your services into partitions, or shards. The partitions can be distributed such 
that each logical set of functionality is separate; this could be done by geographic 
boundaries, or by another criteria like non-paying versus paying users. The advantage 
of these schemes is that they provide a service or data store with added capacity.

Of course there are challenges distributing data or functionality across multiple servers. 
One of the key issues is __data locality__; in distributed systems the closer the data to the 
operation or point of computation, the better the performance of the system. Therefore it 
is potentially problematic to have data spread across multiple servers, as any time it is 
needed it may not be local, forcing the servers to perform a costly fetch of the required 
information across the network.
Another potential issue comes in the form of inconsistency. When there are different services 
reading and writing from a shared resource, there is the chance for race conditions and in those 
cases the data is inconsistent.


The Building Blocks of Fast and Scalable Data Access
====

In this scenario, the hardest part is to work and optimize the data access: to keep things 
simple, for the sake of this example, let's assume you have many terabytes (TB) of data and 
you want to allow users to access small portions of that data at random.
Reading from disk is many times slower than from memory. 
This speed difference really adds up for large data sets; in real numbers memory access is 
as little as 6 times faster for sequential reads, or 100,000 times faster for random reads,
than reading from disk.
We need to add more layers to adjust I/O performances in a distributed environment, so the we
can firstly talk about __Caches__.

Caches, caches and caches :D

Caches take advantage of the locality of reference principle: recently requested
data is likely to be requested again. They are used in almost every layer of computing:
hardware, operating systems, Web browsers, Web applications and more. 
A cache is like __short-term memory__: it has a limited amount of space, but is typically 
faster than the original data source and contains the most recently accessed items. 
__Caches can exist at all levels in architecture__, but are often found at the level 
nearest to the front end, where they are implemented to return data quickly without 
taxing downstream levels.

There are a couple of places you can insert a cache. One option is to insert a cache on your 
request layer node.
Placing a cache directly on a request layer node enables the local storage of response data. 
Each time a request is made to the service, the node will quickly return local, cached data 
if it exists. If it is not in the cache, the request node will query the data from disk. 
The cache on one request layer node could also be located both in memory (which is very fast)
and on the node's local disk (faster than going to network storage). 
However, if your load balancer randomly distributes requests across the nodes, the same request 
will go to different nodes, thus increasing cache misses. 

Two choices for overcoming this hurdle are __global caches__ and __distributed caches__.

A __global cache__ is just as it sounds: all the nodes use the same single cache space. 
This involves adding a server, or file store of some sort, faster than your original 
store and accessible by all the request layer nodes. Each of the request nodes queries 
the cache in the same way it would a local one. This kind of caching scheme can get a 
bit complicated because it is very easy to overwhelm a single cache as the number of 
clients and requests increase, but is very effective in some architectures 
(particularly ones with specialized hardware that make this global cache very fast, 
or that have a fixed dataset that needs to be cached).

In a __distributed cache layout__, typically the cache is divided up using a consistent 
hashing function, such that if a request node is looking for a certain piece of data 
it can quickly know where to look within the distributed cache to determine if that 
data is available. In this case, each node has a small piece of the cache, and will 
then send a request to another node for the data before going to the origin. Therefore, 
one of the advantages of a distributed cache is the increased cache space that can be 
had just by adding nodes to the request pool.
A disadvantage of distributed caching is remedying a missing node. Although even if a 
node disappears and part of the cache is lost, the requests will just pull from the 
origin — so it isn't necessarily catastrophic!
One example of a popular open source cache is __Memcached__ (this 
[link](https://www.cs.utah.edu/~stutsman/cs6963/public/papers/memcached.pdf) is very 
useful to have an overview).
Memcached is used in many large Web sites, and even though it can be very powerful, 
it is simply an in-memory key value store, optimized for arbitrary data storage and 
fast lookups (O(1)).
