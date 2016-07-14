# Elasticsearch Configuration 

This guide discusses memory management and performance tuning from multiple difference sources (which will be cited along the way). 


## [Sizing Elasticsearch](https://www.elastic.co/blog/found-sizing-elasticsearch)

*(From Elastic Documentation)*

Approaches to indexing and sharding can have a major impact on the ability of the cluster to handle growth. If not configured correctly, adding more hardware won't necessarily solve the problem. Resources estimation is also highly dependent on the purpose, such as real-time reporting vs. slower search on archived data. 


### Partitioning and Sharding

The atomic scaling unit for an Elasticsearch index is a **shard**, and it should be considering indivisible for scaling purposes. They can only be moved around. Under the hood, a shard is actually a complete Lucene index. 

A shard must be small enough so that the hardware handling it will cope. There is a limit to how big a shard can be with respect to the hardware, use case, and performance requirements. But this limit is hard to exactly estimate. Approaches to finding the limits are discussed in the later section on testing.

If a shard gets too big, you can either upgrade the hardware to scale vertically, or you can rebuild the entire Elasticsearch index with more shards, to scale horizontally onto more machines of the same kind. 

An ES index with two shards is conceptually the same as two ES indexes with one shard each. But there are practical differences:

* Sharding comes with a cost; the same amount of data in two Lucene indexes is MORE than twice as expensive as storing the same data in a single index
    * this is because index internal like term dictionaries will be duplicated, and there are more files to maintain
* Searching more shared takes more time because there is more data to process; possibly including several trips over the shards

So while you may want to "over-shard" and have more shards than nodes when starting, you can't simply make a huge number of shards and expect everything to work great. 


### Data Flows and their Partitioning Strategies

Different types of data lend themselves to different partitioning approaches.

#### Timestamped Data

For time oriented data, a common approach is to partition data into indexes that hold data for a certain time range (ie. a day, or a week). This allows the index to be optimized, since it will never have data added to it again. It also makes it easier to delete old data, because it is much easier to delete an entire index than large parts of a bigger index. 

This also allows for a focused search over just the relevant indices when searching in a time range. 

For sharding, the sharding strategy can be applied to future indices to accomodate growth, and the old indices can be left alone. 

#### Index per User

If a user only ever searches his or her own data, it *can* make sense to create one index per user. But there is a high cost of having a large number of indices that can outweigh the benefits IF your average user has a small amount of data. If you apply **appropriate filters**, Lucene is so fast that it is usually okay to search an index with ALL user data. 

It may make sense to have an index per user when a user has significantly more data than average. Maybe you have a few users that are power users compared to your average casual user. You can use **filtered index aliases** to allow batches of users to share the same index. This can be problematic if you have a big number of index aliases, because it is part of the cluster state.  

#### Over-allocation and Routing

The only real difference between using multiple indexes and multiple shards is the convenience provided by ES in the form of routing. When a document is indexed, it is *routed* to a shard. The default strategy is to evenly distribute data across shards. 

You could use custom routing to control which shard data goes to. For example, you could force all data for a particular user to go to the same shard. But then you have to issue of uneven data distribution. You could combine this technique with custom indexes to alleviate this problem.  

## Node Usage

Different use cases have different demands on the underlying hardware running the nodes. It is important to know what has high memory, CPU, and/or I/O demands. 

### Searching

Regular searches need to look up relevant terms and their postings in the index. For returned results, the stored fields (such as `_source`) must be fetched as well. Much of this information should be in the operating system's "page cache", so this should not impact disk space. 

Search often follows a Zipfian distribution. This means that 80% of your searches can be answered by 20% of your index. So simple searching is not very demanding on **memory**. You want SSDs, which can serve random reads efficiently. 

For search-heavy applications, you want page cache and I/O to be able to server random reads. Unless custom scoring and sorting is used, heap space usage is fairly limited. Similarly to when you aggregate on a field, sorting and scripting/scoring on fields require rapid access to documents' values given their IDs.

### Analytics

Analytics have a very different memory profile from search. When we search, we often just want the top *n* results. When we analyze, we aggregate over possibly billions of records. 

To do this, ES needs **a lot of memory**. It juggles many caches for near realtime analytics, and it is problematic to keep this entirely in memory. 

### Field Data and Document Values

All of a documents' values for a field are loaded entirely into memory the first time you try to use it for aggregations, sorting, or scripting. 

When peforming a search, the inverted index allows ES to find the seach term and immediately get all the documents that contain the term. Sorting, aggregatings, and field access require the OPPOSITE data access pattern. They need to look up the document and find the terms in a field. 

`doc_values` are an **on-disk** data structure, built when the document is indexed, that enables this data access pattern. `doc_values` can be used on almost any field type, excepted for `analyzed string` fields. The must be used on `keyword` or `exact match` fields. 

If you are sure that you don't need to sort or aggregate on a field, or access the field value from a script, you can disable doc values in order to save disk space. 

Using `doc_values` can relieve **heap space** of memory pressure when performing sorting, aggregation, etc operations. This is because instead of loading all the data into heap space, some data can by provided quickly by the storage. 

The result in bigger on-disk indexes and slightly more overhead when searching, but less heap space spent on field caches. This is nice if you only ever use a small fraction of the values. For example, if your queries and filters typically work with a small sub-set of your entire index, then the remaining unused and possible majority of data does not cost you any memory.

If instead you use almost all data all the time, `doc_values` won't necessarily help you. You will still need a lot of memory. Nevertheless, having the data off the heap can massively reduce garbage collection pressure. As noted in Elasticsearch in Production, garbage collection can become a problem with excessively big heaps. You cannot scale a single node's heap to infinity, but conversely, you cannot have too much page cache.


## Testing

### Test with Real Data and Searches

To get real estimates, you need to test as realistically as possible. This means BOTH the data and the searches should resemble what you expect to happen. You won't be answers that can be answered with cache, and the occasional cache miss that will inevitably occur in real life. 

Existing logs (if you have them), can be very valuable as you can easily replay them onto the cluster. Consider that searches do tend to have a Zipf distribution, and don't be overly pessimistic with you tests. Though admittedly, if you can afford it, being pessimistic is better than over-optimistic. 

### Checking Resource Usage

ES has many endpoints for inspecting resource usage:

* [cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html)
* [nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-stats.html)
* [indexes and shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-stats.html#indices-stats)
* [segments](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html)

You can also monitor resources using Marvel and Kibana. 

Things to pay attention to:

* total heap space used
* memory usage of:
    * field caches
    * filter caches
    * ID caches
    * completion suggesters
* garbage collection statistics
    * if your node spends a lot of time garbage collecting, it's a sign you need more memory and/or more nodes
* memory usage growth
    * if you see a sawtooth that hits the ceiling more and more frequently, with less and less memory freed up each time the garbage collector runs, this is a problem

### Start Big, Scale Down

Pay for a big cluster for a few hours once you have your realistic tests setup, and monitor very closely. Then, if the cluster is overkill, scale down. Don't do it the other way, because you will end up wasting time not being able to tell what the actual problem is. 

---

# [Elasticsearch in Production](https://www.elastic.co/blog/found-elasticsearch-in-production)

*(from Elastic blog)*

The article serves to give an overview of the important aspects of running and maintaining Elasticsearch clusters. It also explains the importance of having enough memory and how to achieve high availability. 


## Memory 

Search needs to be FAST. As a result, we need to keep as much data as possible in memory, but this can be challange. Thus, a lot of effort is put into maintaining various caches:

* page cache - hot parts of index structures, such as term dictionaries are storing in the operating system's page cache. This requires that we cannot allocate everything to Elasticsearch.
* field cache - field values used for faceting, sorting or scripting
* filter cache - filters can be cache as bitmaps, making further use of the filter blazingly fast

Building index structures also requires a lot of memory. Inverted indexes are typically built in segments by indexing as much as possible until some threshold is reached, and then flushing the segment to storage. A written segment is immutable. 

## Running out of Memory

Running out of memory can manifest itself in many ways:

* A search request can result in an attempt to load way too large fields into memory, e.g. because it's faceted, sorted or scripted on. Note: be careful before you unleash queries with unknown memory profile on your production cluster!
* Too many fields are attempted to be kept in memory. Note: if you offer a ton of faceting opportunities, make sure you have enough memory to cope with all of them at once.
* Waiting too long before flushing an in-progress index segment can cause it to outgrow the heap space.
* New documents can cause previously cached fields to become too large.
* Indexes grow so large that not enough can fit in the page cache …
* … or the page cache is continuously "poisoned" or invalidated due to other things happening on the system.

If you outgrow the page cache, you will experience a gradual slowdown. If you outgrow the heap space dedicated to your node, things can *crash suddenly* when an `OutOfMemory` error occurs. 

## OutOfMemory-caused Crashes

Elasticsearch does not look before it leaps. It ASSUMES you have provided it with enough memory. 





