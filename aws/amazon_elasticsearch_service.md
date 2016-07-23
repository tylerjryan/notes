# [Amazon Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/)

Amazon Elasticsearch Service is a managed service that makes it easy to deploy, operate, and scale Elasticsearch in the AWS Cloud. Elasticsearch is a popular open-source search and analytics engine for use cases such as log analytics, real-time application monitoring, and click stream analytics. You can set up and configure your Amazon Elasticsearch cluster in minutes from the AWS Management Console. Amazon Elasticsearch Service provisions all the resources for your cluster and launches it. The service automatically detects and replaces failed Elasticsearch nodes, reducing the overhead associated with self-managed infrastructure and Elasticsearch software. Amazon Elasticsearch Service allows you to easily scale your cluster via a single API call or a few clicks in the AWS Management Console. With Amazon Elasticsearch Service, you get direct access to the Elasticsearch open-source API so that code and applications you’re already using with your existing Elasticsearch environments will work seamlessly.

# [Deep Dive Webinar]

Data partitioning for search

* Shard: an instance of Lucene with a portion of an index
* Index: a collection of Data

If your cluster health shows up as yellow, see if you have enough nodes to allow for Elasticsearch to distribute the primary and replica shards across different nodes

How do you split your data into shards:
