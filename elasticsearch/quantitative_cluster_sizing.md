# [Quantitative Cluster Sizing](https://www.elastic.co/elasticon/conf/2016/sf/quantitative-cluster-sizing)

How many shards should I have? How many nodes should I have? What about replicas? Do these questions sound familiar? The answer is often ‘it depends’. This talk will outline the factors that affect sizing and walk you through a quantitative approach to estimating the configuration and size of your cluster.

## Notes

* what we want to determine:
  * how much disk storage will N documents require?
  * when is a single shard too big for my requirements?
  * how many active shards saturate my particular hardware?
  * how many shards/nodes will I need to sustain X index rate and Y search response?

* sizing methodology:
  1. determine various disk utilization
  2. determine breaking point of a shard
  3. determine saturation point of a node
  4. test configuration on a small cluster

* experiment 1
  * single node cluster with one index (1 primary, 0 replicas)
  * index a decent amount of data (1 GB or about 10 million docs)
  * calculate storage on disk both as-is and after a "\_forcemerge"
  * repeat the above calculations with different mapping configurations
    * "\_all" both enabled and disabled
    * tweaking settings for each field

* experiment 2
  * same single node
  * index *realistic* data and use *realistic* queries
  * plot index speed and query response time
  * determine where point of diminishing returns is for your requirements

* experiment 3
  * single node cluster with 2 primary shards (no replicas)
  * repeat experiment 2 to see how performance varies
  * keep adding more shards to see when point of diminishing returns occurs

* experiment 4
  * configure a small representative cluster
  * add representative data volume
  * run realistic benchmarks:
    * max indexing rates
    * querying across varying data volumes
    * benchmark concurrent querying and indexing at various levels
  * measure resource usage, overall docs, disk usage, etc.

* write the results at each step to a second Elasticsearch cluster so we can look at them using Kibana
* script the testing process so it can be done again later
