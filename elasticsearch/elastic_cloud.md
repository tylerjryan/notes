# Elastic Cloud

This document contains highlights about the [Elastic Cloud service](https://www.elastic.co/cloud) from the [getting started video](https://www.elastic.co/webinars/getting-started-with-found), as well the [Docker Case Study](https://www.elastic.co/use-cases/docker), and [documentation](https://www.elastic.co/guide/en/cloud/current/index.html). 

## Getting Started with Found, Hosted & Managed Elasticsearch

From the site:

> Found, our Elasticsearch as a Service offering, is now GA and we’d like to give you the official tour. Found is a fully hosted, fully managed service that provides the real-time search and analytics capabilities of Elasticsearch and integrates across the entire Elastic ecosystem.

> In this webinar, two of members of the Found team will walk you through the product’s ins and outs, including:

> * Spinning up a new cluster, small or large
> * Setting up Logstash and Kibana
> * Customizing your cluster
> * Found as a Service architecture

### Notes

* Found was an independent company started in 2012, had built a service very similar to Elasticsearch. Decided to scrap code and go all-in with Elasticsearch.
* Launched a "hosted Elasticsearch" service in 2012. Objective to hide complexity while remaining flexible
* Elastic acquired Found in 2015
* Found: Elasticsearch as a service
* Found's responsibilities 
    * failover: when nodes fail, reroute and replace nodes seamlessly
    * resilience: shield customer from infrastructure problems
    * backup: every 30 minutes to highly durable storage
    * easy upgrades: single click, zero-downtime upgrades
    * support: appropriate, timely customer service
* Customer responsibilities:
    * "make sure you are using Elasticsearch correctly"  
    * coding: build an application that meets your expectations
    * sizing: provide sufficient CPU, memory, and replicas for your needs
    * usage: expensive searches. Retry on failure. 
    * security: use Found's security feature to protect the cluster
* Plans
    * see slides for full table of comparison (see tables starting at 8:00)
    * standard: infrastructure only
        * limited access control lists
        * no access to premium software Elastic support
    * premium - gold
    * premium - platinum
* Demo
    * create cluster:
        * configure the cluster size
        * fault tolerance (number of data centers)
        * version of Elasticsearch (can always upgrade)
        * Sharding (if you know what you're doing)
        * automatic index creation
        * how careful with deletion
        * (with premium) custom plugins/dictionaries/scripts
        * can choose available plugins
        * cluster name
    * view cluster:
        * endpoints to connect to cluster
        * dashboards like Kibana
        * live logs
    * data centers will be in the same region
    * if a node goes down, Found will redirect to the most available node automatically. A new node will be spun up and the data will be replicated again. This would be very hard to configure on your own. 
* Questions
    * you can clone your cluster to a cluster with a new version to test things. There is also a migration plugin to get a list of things to think about before doing a major version upgrade.
    * for minor version upgrade, you shouldn't need to do anything at all
    * if you have 2 nodes in the same availability zone on Amazon, you cannot be sure that they are not actually using the same hardware; this is why multiple data center span availability zones
    * try to pick your availability zone close to the users
    * how does backup work? backups to S3. Elasticsearch has a "snapshots" restore feature. Each customers get a dedicated S3 bucket. You could move the S3 bucket to your own if you would like to take it with you. 
    * can clone across regions
    * every request to a Found hosted cluster will go through their load balancers, so you always connect to the cluster endpoint, and never an individual node. 
    * infrastructure problems will be monitored even on the standard plan, but not usage problems (ie. cpu and memory); need to have premium support to get support for that. 
    * the support plans are the same for Found as the subscriptions for Elasticsearch 
    * free trial to create a 2 zone cluster with 1GB RAM and 8GB RAM
    * any cluster created on found has Shield enabled; but to be able to utilize entire feature set, need premium plans
    * they give Kibana 4 
    * billing is by the hour; can spin and do some testing and then turn it off; billing is done every month for standard plans; annual plans for premium level; can scale at any time, even for a short period of time
    * to do a major version upgrade, run two cluster at the same time, and move to the other one once you are ready 
    * how do I decid number of nodes in the cluster; Found picks for you based on the capacity and availability configuration of the cluster; this prevent sub-optimal configurations with small; 
        * generally if cluster is 32GB or less, will be on a single node per availability zone 
    

# [Elastic Cloud Documentation](https://www.elastic.co/guide/en/cloud/current/index.html)

This section highlights the official Elastic Cloud Documentation. 


## [Memory Pressure](https://www.elastic.co/guide/en/cloud/current/memory-pressure.html)

The EC console includes a memory pressure indicator, which indicates the fill percentage of the "old generation pool". There are separate "pools" in the heap for new objects and old objects. Most new objects are short-lived and are quickly made available for garbage collection. Each time the collector runs, you'll see a drop in memory, and the surviving objects are moved into the "old objects pool", which is garbage collected less frequently. This is because it's less likely that there be a substantial amount of garbage there. 

More information can be found in [Understanding Memory Pressure](https://www.elastic.co/blog/found-understanding-memory-pressure-indicator).

Below 75% percent means everything is good. Above 75%, you should monitor things more closely, and see if there are optimizations that can be done. Check your mappings, shard count, and cache utilization. If for some reason Elasticsearch seems to use more heap than you have data, you probably should check your mappings and your sharding strategy. 

Summary:

* Below 75%: No worries.
* 75% to 85%: Acceptable if performance is satisfactory and load is expected to be stable.
* Above 85%: Now is the time for action. Either reduce memory consumption or add more memory.


## [Preflight Checklist](https://www.elastic.co/guide/en/cloud/current/preflight-checklist.html)

Follow this checklist to maximize cluster performance, reliability, and security. 

* do you need to scale your cluster? => set your cluster configuration
* is your data readily available? => set your fault tolerance; 2 data centers recommended for production apps
* is your data secure? => ensure your app is accessing ES via SSL. Be sure to use HTTPS. If your URL is leaked other will have access to it unless you set Access Control. Keep your URL safe! Specify access rights to the cluster. For detailed information, see [this post](https://www.elastic.co/blog/found-elasticsearch-security).
* is your cluster healthy? => check Paramedic, which tracks disk utilization over shards, CPU usage, heap (memory) usage, and disk space used by the app. Make sure there is enough margin to handle spikes in traffic and projected growth. Try scaling up and down if you need to perform some tests. 
* does your Elasticsearch client have the "right stuff"? => is your client properly configured? You need to be aware of things like retries, connection pooling, and bulk operations. See [client considerations](https://www.elastic.co/blog/found-elasticsearch-in-production?q=elasticsearch%20in%20prod#client-considerations). 

If you have taken care of all these things, then you are good to go!


## [Cluster Configuration](https://www.elastic.co/guide/en/cloud/current/cluster-config.html)

This section describes all the configuration that can be done on a cluster.

### Region

Choose region to be as close to your application servers as possible in order to minimize network delay.

**Cannot be changed once created.**

### Capacity and SSD

Memory and hard disk ultimately need to be determining through testing. Sizes start at 1GB up to 256GB memory, with 16 GB of SSD for each GB of memory. 

### Fault Tolerance

Fault tolerance allows you to specify number of data centers for your cluster. The capacity above is PER data center. EC configures nodes such that the cluster will automatically allocate replicas between data centers. 

### Version

Elasticsearch version. At any given time, the two latest minor versions will be available for deployments on new clusters.

#### Upgrading

You can upgrade the cluster with one click. Minor upgrades will have no downtime, while major upgrades will require the cluster to stop briefly. Customers are given 6 months to upgrade existing clusters when a minor ES version is removed from deployment. 

### Dynamic Scripts

`script_fields` make cluster flexible, but they allow arbitrary code execution, so they can be potentially dangerous. If you plan to delegate access to other users, especially anonymous users, this should be disabled unless you have specific need for it and have setup appropriate access controls. 

### Custom Plugins/Dictionaries/Scripts

*(Premium customers only)*

Enable custom plugins, dictionaries, or scripts. 

### Plugins

Enable/disable officially provided plugins such as the Mapper attachment. 

### Kibana 4

Enable/disable Kibana 4. It is disabled by default. Once enabled, you need to provide the enpoint with credentials that have access to ES resources, such as granting read access to relevant Elasticsearch indexes and granting update access to the .kibana index.

### Name

Set a name for the cluster. 


## [Version Policy](https://www.elastic.co/guide/en/cloud/current/version-policy.html)

See this page for information regarding versions, upgrades, support, and deprecation. 


## [Monitoring with Marvel](https://www.elastic.co/guide/en/cloud/current/marvel.html)

Marvel enables you to easily monitor Elasticsearch through Kibana. You can view your cluster’s health and performance in real time as well as analyze past cluster, index, and node metrics.

This page describes how to enable and use Marvel on Elastic Cloud. For full documentation of Marvel, see [Marvel’s documentation](https://www.elastic.co/guide/en/marvel/current/introduction.html).

Marvel consists of two components: a Marvel agent that runs on each node in your cluster, and a Marvel application in Kibana.

The Marvel agent collects and indexes metrics from Elasticsearch into Elasticsearch, and you visualize the data through the Marvel dashboards in Kibana.

### The Marvel Agent

Configure Marvel to monitor the desired cluster so it knows where to ships its metrics. You can have the agent send metrics to itself or an external ES cluster. 

**Production cluster should log metrics to an external cluster**. 

### Marvel's Kibana App

Access Marvel through the Kibana app drawer. You will need to access Marvel through Kibana running on the cluster that is receiving the metrics. 

## Monitoring in Production

Marvel indexes metric into Elasticsearch itself, and these indexes consume storage, memory, and CPU cycle like any other index. The indexes are *not automatically cleaned up*, though the capability is coming soon. You should clean these up, and can use [Curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html) to do it. 

It is recommened to ship your production cluster metrics to a dedicated Marvel cluster, which can likely be pretty small. 


