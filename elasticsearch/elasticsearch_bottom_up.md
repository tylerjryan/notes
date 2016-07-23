# [Elasticsearch from the bottom up](https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up)

Basic data structure: inverted index
* maps terms to documents (and possibly positions in documents) containing the term
* the "dictionary" contains sorted terms and their frequency (number of documents they appear in); the "postings" contain a list of documents each term appears in
* the terms dictate the operations that we can do efficiently
  * finding based on **prefix** is very efficient
  * finding terms containing "ours" is very inefficient because every term must be traversed

## Building Indexes

Priorities: search speed, index compactness, indexing speed, time for changes to become visible

* to minimize index size, compression techniques are used. Sacrifices the ability for efficient updates. Index files Lucene writes are immutable. Updating a document is a delete + re-index => updating a document is more expensive that creating it in the first place
