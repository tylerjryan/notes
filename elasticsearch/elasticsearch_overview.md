# [Elasticsearch Getting Started Guide highlights](https://www.elastic.co/guide/en/elasticsearch/guide/current/getting-started.html)

## [Talking to Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/guide/current/_talking_to_elasticsearch.html)

HTTP Request format:
> curl -X\<VERB> '\<PROTOCOL>://\<HOST>:\<PORT>/\<PATH>?\<QUERY_STRING>' -d '\<BODY>'

* VERBS: GET, POST, PUT, HEAD, DELETE
* PROTOCOL: http or https (if you have an htts proxy in front of Elasticsearch)
* HOST: The hostname of any node in your Elasticsearch cluster, or localhost for a node on your local machine.
* PORT: The port running the Elasticsearch HTTP service, which defaults to 9200 
* PATH: API Endpoint (for example \_count will return the number of documents in the cluster). Path may contain multiple components, such as \_cluster/stats or \_nodes/stats/jvm
* QUERY_STRING: Any optional query-string parameters (for example ?pretty will pretty-print the JSON response to make it easier to read.)
* BODY: A JSON-encoded request body (if the request needs one.)

To get HTTP headers in the curl response, request them with `-i`:
> curl -i -XGET 'localhost:9200/'

## [Document Oriented](https://www.elastic.co/guide/en/elasticsearch/guide/current/_document_oriented.html)

Elasticsearch is document oriented, meaning that it stores entire objects or documents. It not only stores them, but also indexes the contents of each document in order to make them searchable. In Elasticsearch, you index, search, sort, and filter documents—not rows of columnar data. This is a fundamentally different way of thinking about data and is one of the reasons Elasticsearch can perform complex full-text search.

Elasticsearch uses JSON, which has become the standard in the NoSQL movement. 

## [Finding Your Feet](https://www.elastic.co/guide/en/elasticsearch/guide/current/_finding_your_feet.html)

We happen to work for Megacorp, and as part of HR’s new "We love our drones!" initiative, we have been tasked with creating an employee directory. The directory is supposed to foster employer empathy and real-time, synergistic, dynamic collaboration, so it has a few business requirements:

* Enable data to contain multi value tags, numbers, and full text.
* Retrieve the full details of any employee.
* Allow structured search, such as finding employees over the age of 30.
* Allow simple full-text search and more-complex phrase searches.
* Return highlighted search snippets from the text in the matching documents.
* Enable management to build analytic dashboards over the data.

---
START OF TUTORIAL

## [Indexing Employee Documents](https://www.elastic.co/guide/en/elasticsearch/guide/current/_indexing_employee_documents.html)

* A single `document` represents a single employee. 
* A `document` belongs to a `type`
* `types` live inside an `index`
* a `cluster` can contain multiple `indices` (think databases)

Index (verb):
> To index a document is to store a document in an index (noun) so that it can be retrieved and queried. It is much like the INSERT keyword in SQL except that, if the document already exists, the new document would replace the old.

## [Search Lite](https://www.elastic.co/guide/en/elasticsearch/guide/current/_search_lite.html)

* By default, a search will return the top 10 results


## [Full-Text Search](https://www.elastic.co/guide/en/elasticsearch/guide/current/_full_text_search.html)

By default, Elasticsearch sorts matching results by their relevance score, that is, by how well each document matches the query. The first and highest-scoring result is obvious: John Smith’s about field clearly says “rock climbing” in it.

But why did Jane Smith come back as a result? The reason her document was returned is because the word “rock” was mentioned in her about field. Because only “rock” was mentioned, and not “climbing,” her \_score is lower than John’s.

This is a good example of how Elasticsearch can search within full-text fields and return the most relevant results first. This concept of relevance is important to Elasticsearch, and is a concept that is completely foreign to traditional relational databases, in which a record either matches or it doesn’t.


## [Highlighting Intro](https://www.elastic.co/guide/en/elasticsearch/guide/current/highlighting-intro.html)

See more info about highlighting in the [complete highlighting reference guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-highlighting.html).


## [Analytics](https://www.elastic.co/guide/en/elasticsearch/guide/current/_analytics.html)

Elasticsearch has functionality called `aggregations`, which allow you to generate sophisticated analytics over your data. It is similar to GROUP BY in SQL, but much more powerful.

END OF TUTORIAL
---


# More Detailed Guide to Elasticsearch

This section of the book goes into more detail about each aspect of Elasticsearch, to help you go from novice to expert.

## [What Is A Document?](https://www.elastic.co/guide/en/elasticsearch/guide/current/document.html)

Most entities or objects in most applications can be serialized into a JSON object, with keys and values. A key is the name of a field or property, and a value can be a string, a number, a Boolean, another object, an array of values, or some other specialized type such as a string representing a date or an object representing a geolocation:

```
{
    "name":         "John Smith",
    "age":          42,
    "confirmed":    true,
    "join_date":    "2014-06-01",
    "home": {
        "lat":      51.5,
        "lon":      0.1
    },
    "accounts": [
        {
            "type": "facebook",
            "id":   "johnsmith"
        },
        {
            "type": "twitter",
            "id":   "johnsmith"
        }
    ]
}
```

Often, we use the terms object and document interchangeably. However, there is a distinction. An object is just a JSON object—similar to what is known as a hash, hashmap, dictionary, or associative array. Objects may contain other objects. In Elasticsearch, the term document has a specific meaning. It refers to the top-level, or root object that is serialized into JSON and stored in Elasticsearch under a unique ID.


## [Document Metadata](https://www.elastic.co/guide/en/elasticsearch/guide/current/_document_metadata.html)

A document doesn’t consist only of its data. It also has metadata—information about the document. The three required metadata elements are as follows:

* \_index: Where the document lives. See [index management](https://www.elastic.co/guide/en/elasticsearch/guide/current/index-management.html).
* \_type: The class of object that the document represents. See [types and mappings](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping.html).
* \_id: The unique identifier for the document. 

**Note**: "\_index" + "\_type" + "\_id" = unique identifier for a document in Elasticsearch


## [Indexing a Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/index-doc.html)

If your document has a natural identifier, use it for the "\_id" field:
```
PUT /{index}/{type}/{id}
{
  "field": "value",
   ...
}
```

Sample reponse:
```
{
    "_index":    "website",
    "_type":     "blog",
    "_id":       "123",
    "_version":  1,
    "created":   true
}
```

Every document in Elasticsearch has a version number. Every time a change is made to a document (including deleting it), the \_version number is incremented. In Dealing with Conflicts, we discuss how to use the \_version number to ensure that one part of your application doesn’t overwrite changes made by another part.

### Autogenerating IDs

If you want to let Elasticsearch generate IDs for you, use `POST` instead of `PUT`! Example:
```
POST /website/blog/
{
    "title": "My second blog entry",
    "text":  "Still trying this out...",
    "date":  "2014/01/01"
}
```

Autogenerated IDs are 20 character long, URL-safe, Base64-encoded string universally unique identifiers, or UUIDs.


## [Retrieving a Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/get-doc.html)

* When using `GET` to retrive a document, you will get both metadata and the original JSON that describes its contents under the `\_source` field.
* If a `GET` request does not find a document, there will still be a response, but with `{"found": false}`.

Use the `\_source` endpoint to get just a subset of the fields: 
> GET /website/blog/123?_source=title,text

Or get the entire `\_source` JSON:
> GET /website/blog/123?_source


## [Checking Whether a Document Exists](https://www.elastic.co/guide/en/elasticsearch/guide/current/doc-exists.html)

Use the `HEAD` method to check whether a document exists. These requests don't return a body; just HTML headers:
> curl -i -XHEAD http://localhost:9200/website/blog/123

Response: Elasticsearch will return a 200 OK status code if the document exists:
```
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

Response: And a 404 Not Found if it doesn’t exist:
```
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```


## [Updating a Whole Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/update-doc.html)

Documents in Elasticsearch are immutable; we cannot change them. Instead, if we need to update an existing document, we reindex or replace it, which we can do using the same index API that we have already discussed in Indexing a Document.

```
PUT /website/blog/123
{
    "title": "My first blog entry",
    "text":  "I am starting to get the hang of this...",
    "date":  "2014/01/02"
}
```

In the response, we can see that Elasticsearch has incremented the \_version number:
```
{
    "_index" :   "website",
    "_type" :    "blog",
    "_id" :      "123",
    "_version" : 2,
    "created":   false 
}
```
**Note**: The created flag is set to false because a document with the same index, type, and ID already existed.

Internally, Elasticsearch has marked the old document as deleted and added an entirely new document. The old version of the document doesn’t disappear immediately, although you won’t be able to access it. Elasticsearch cleans up deleted documents in the background as you continue to index more data.

Later in this chapter, we introduce the update API, which can be used to make partial updates to a document. This API appears to change documents in place, but actually Elasticsearch is following exactly the same process as described previously:

1. Retrieve the JSON from the old document
2. Change it
3. Delete the old document
4. Index a new document

The only difference is that the update API achieves this through a single client request, instead of requiring separate get and index requests.


## [Creating a New Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/create-doc.html)

To ensure a new document is created (meaning you don't want to check if the document already exists), allow Elasticsearch to set the ID for you:
> POST /website/blog/
> {...}

If you already have an id, use the `\_create` endpoint to create new documents:
> PUT /website/blog/123/_create

If the request succeeds in creating a new document, Elasticsearch will return the usual metadata and an HTTP response code of 201 Created.

On the other hand, if a document with the same \_index, \_type, and \_id already exists, Elasticsearch will respond with a 409 Conflict response code, and an error message like the following:
```
{
  "error" : "DocumentAlreadyExistsException[[website][4] [blog][123]:
             document already exists]",
  "status" : 409
}
```


## [Deleting a Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/delete-doc.html)

Delete a document using the `DELETE` method:
> DELETE /website/blog/123

If the document is found, Elasticsearch will return an HTTP response code of 200 OK and a response body like the following. Note that the \_version number has been incremented:
```
{
    "found" :    true,
    "_index" :   "website",
    "_type" :    "blog",
    "_id" :      "123",
    "_version" : 3
}
```

If the document isn’t found, we get a 404 Not Found response code and a body like this:
```
{
    "found" :    false,
    "_index" :   "website",
    "_type" :    "blog",
    "_id" :      "123",
    "_version" : 4
}
```

Even though the document doesn’t exist (found is false), the \_version number has still been incremented. This is part of the internal bookkeeping, which ensures that changes are applied in the correct order across multiple nodes.


## [Partial Updates to Documents](https://www.elastic.co/guide/en/elasticsearch/guide/current/partial-updates.html)

This provides a way to "update a document in place", without needing to update the document with a new `PUT` request. Instead, we can use `POST` with the `\_update` endpoint. 

The simplest form of the update request accepts a partial document as the doc parameter, which just gets merged with the existing document. Objects are merged together, existing scalar fields are overwritten, and new fields are added. For instance, we could add a tags field and a views field to our blog post as follows:
```
POST /website/blog/1/_update
{
    "doc" : {
        "tags" : [ "testing" ],
        "views": 0
        }
}
```

If the request succeeds, we see a response similar to that of the index request:
```
{
    "_index" :   "website",
    "_id" :      "1",
    "_type" :    "blog",
    "_version" : 3
}
```

You can you Groovy scripts to make more complex updates. See [scripting reference documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html).

### Updating a document that may not exist

If you try to update a page and it doesn't exist, it will fail. 

### Handling Conflicts

This can be done automatically by setting the `retry_on_conflict` parameter to the number of times that update should retry before failing; it defaults to 0.


## [Retrieving Multiple Documents](https://www.elastic.co/guide/en/elasticsearch/guide/current/_retrieving_multiple_documents.html)

As fast as Elasticsearch is, it can be faster still. Combining multiple requests into one avoids the network overhead of processing each request individually. If you know that you need to retrieve multiple documents from Elasticsearch, it is faster to retrieve them all in a single request by using the multi-get, or mget, API, instead of document by document.

The mget API expects a docs array, each element of which specifies the \_index, \_type, and \_id metadata of the document you wish to retrieve. You can also specify a \_source parameter if you just want to retrieve one or more specific fields.

See "sense_reference" for examples.

You can also search inside an individual index or type. Even if you do, you can override type in the individual requests.

If a document is not found, you will get a reponse with `"found" : false` in it. This will NOT affect the retrieval of other documents. 


## [Cheaper in Bulk](https://www.elastic.co/guide/en/elasticsearch/guide/current/bulk.html)

In the same way that mget allows us to retrieve multiple documents at once, the bulk API allows us to make multiple create, index, update, or delete requests in a single step. This is particularly useful if you need to index a data stream such as log events, which can be queued up and indexed in batches of hundreds or thousands.

The bulk request body has the following, slightly unusual, format:
```
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```

For more info on the formatting, see [here](https://www.elastic.co/guide/en/elasticsearch/guide/current/distrib-multi-doc.html#bulk-format).

This format is like a stream of valid one-line JSON documents joined together by newline (\n) characters. Two important points to note:

* Every line must end with a newline character (\n), including the last line. These are used as markers to allow for efficient line separation.
* The lines cannot contain unescaped newline characters, as they would interfere with parsing. This means that the JSON must not be pretty-printed.

See the link in the header for more details on how to constuct the bulk body. The `action` can be `create, index, update, or delete`. A request body is required for all actions except `delete`.

The metadata should specify the \_index, \_type, and \_id of the document to be indexed, created, updated, or deleted.

Each subrequest is executed independently, so the failure of one subrequest won’t affect the success of the others. If any of the requests fail, the top-level error flag is set to true and the error details will be reported under the relevant request.

The bulk API is used like this:
```
POST /_bulk
...bulk request body...
```

You can also POST to indexes or types. 

See the sense_reference for an example of using the BULK API.

**Size of requests?**: It is often useful to keep an eye on the physical size of your bulk requests. One thousand 1KB documents is very different from one thousand 1MB documents. A good bulk size to start playing with is around 5-15MB in size.  A good place to start is with batches of 1,000 to 5,000 documents or, if your documents are very large, with even smaller batches.


## [Searching -- The Basic Tools](https://www.elastic.co/guide/en/elasticsearch/guide/current/search.html)

The real power of Elasticsearch comes from its ability to turn Big Data into Big Information. 

Every field in a document is indexed and can be queried. And it’s not just that. During a single query, Elasticsearch can use all of these indices, to return results at breath-taking speed. That’s something that you could never consider doing with a traditional database.

A search can be any of the following:

* A structured query on concrete fields like gender or age, sorted by a field like join_date, similar to the type of query that you could construct in SQL
* A full-text query, which finds all documents matching the search keywords, and returns them sorted by relevance
* A combination of the two

While many searches will just work out of the box, to use Elasticsearch to its full potential, you need to understand three subjects:

* Mapping
    * How the data in each field is interpreted
* Analysis
    * How full text is processed to make it searchable
* Query DSL
    * The flexible, powerful query language used by Elasticsearch

See sense_reference for full example that follows this chapter.


## [The Empty Search](https://www.elastic.co/guide/en/elasticsearch/guide/current/empty-search.html)

The most basic form of the search API is the empty search, which doesn’t specify any query but simply returns all documents in all indices in the cluster:
```
GET /_search
```

### hits

The most important section of the response is hits, which contains the total number of documents that matched our query, and a hits array containing the first 10 of those matching documents—the results.

Each result in the hits array contains the \_index, \_type, and \_id of the document, plus the \_source field. This means that the whole document is immediately available to us directly from the search results. This is unlike other search engines, which return just the document ID, requiring you to fetch the document itself in a separate step.

Each element also has a \_score. This is the relevance score, which is a measure of how well the document matches the query. By default, results are returned with the most relevant documents first; that is, in descending order of \_score. In this case, we didn’t specify any query, so all documents are equally relevant, hence the neutral \_score of 1 for all results.

The max\_score value is the highest \_score of any document that matches our query.

### took

The took value tells us how many milliseconds the entire search request took to execute.


### shards

The \_shards element tells us the total number of shards that were involved in the query and, of them, how many were successful and how many failed. We wouldn’t normally expect shards to fail, but it can happen. If we were to suffer a major disaster in which we lost both the primary and the replica copy of the same shard, there would be no copies of that shard available to respond to search requests. In this case, Elasticsearch would report the shard as failed, but continue to return results from the remaining shards.


## [Multi-index, Multitype](https://www.elastic.co/guide/en/elasticsearch/guide/current/multi-index-multi-type.html)

You can search across one or more specific indexes or types simultaneously by specifying in the URL:

* `/_search`
    * Search all types in all indices
* `/gb/_search`
    * Search all types in the gb index
* `/gb,us/_search`
    * Search all types in the gb and us indices
* `/g*,u*/_search`
    * Search all types in any indices beginning with g or beginning with u
* `/gb/user/_search`
    * Search type user in the gb index
* `/gb,us/user,tweet/_search`
    * Search types user and tweet in the gb and us indices
* `/_all/user,tweet/_search`
    * Search types user and tweet in all indices


## [Pagination](https://www.elastic.co/guide/en/elasticsearch/guide/current/pagination.html)

How can we see more than just the 10 documents that are returned from using `_search`? 

Elasticsearch accepts `from` and `size` parameters! 

* `size`
    * Indicates the number of results that should be returned, defaults to 10
* `from`
    * Indicates the number of initial results that should be skipped, defaults to 0

Beware of paging too deep or requesting too many results at once. Results are sorted before being returned. But remember that a search request usually spans multiple shards. Each shard generates its own sorted results, which then need to be sorted centrally to ensure that the overall order is correct.o


## [Search *Lite*](https://www.elastic.co/guide/en/elasticsearch/guide/current/search-lite.html)

There are two forms of the search API: a “lite” query-string version that expects all its parameters to be passed in the query string, and the full request body version that expects a JSON request body and uses a rich search language called the query DSL.

See [Query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax) for more details on embedded queries in the query string as part of the request.

### The _all Field

This simple search returns all documents that contain the word mary:
```
GET /_search?q=mary
```

In the previous examples, we searched for words in the tweet or name fields. However, the results from this query mention mary in three fields:

* A user whose name is Mary
* Six tweets by Mary
* One tweet directed at @mary

How has Elasticsearch managed to find results in three different fields?

When you index a document, Elasticsearch takes the string values of all of its fields and concatenates them into one big string, which it indexes as the special \_all field. For example, when we index this document:
```
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```
it’s as if we had added an extra field called \_all with this value:
> "However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
The query-string search uses the \_all field unless another field name has been specified.


# [Mapping and Analysis](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping-analysis.html)

(See sense_reference for examples)

The way our data is indexed in the \_all field is different from how it is indexed in a particular field. So let’s take a look at how Elasticsearch has interpreted our document structure, by requesting the mapping (or schema definition) for the tweet type in the gb index:
```
GET /gb/_mapping/tweet
```

The `_all` field is a string concatenation of all fields of a documents, so it is always of type "string". But a field with type "date", for example, will be indexed differently than a string, and thus can be searched differently.

The biggest difference is between fields that represent *exact* values and fields that represent *full text*. See the next section for more info.


## [Exact Values Versus Full Text](https://www.elastic.co/guide/en/elasticsearch/guide/current/_exact_values_versus_full_text.html)

Data in Elasticsearch can be broadly divided into two types: exact values and full text.

Exact values are exactly what they sound like. Examples are a date or a user ID, but can also include exact strings such as a username or an email address. The exact value Foo is not the same as the exact value foo. The exact value 2014 is not the same as the exact value 2014-09-15.

Full text, on the other hand, refers to textual data—usually written in some human language — like the text of a tweet or the body of an email.

Querying full-text data is much more subtle. We are not just asking, “Does this document match the query” but “How well does this document match the query?” In other words, how relevant is this document to the given query?

We seldom want to match the whole full-text field exactly. Instead, we want to search within text fields. Not only that, but we expect search to understand our intent:

* A search for UK should also return documents mentioning the United Kingdom.
* A search for jump should also match jumped, jumps, jumping, and perhaps even leap.
* johnny walker should match Johnnie Walker, and johnnie depp should match Johnny Depp.
* fox news hunting should return stories about hunting on Fox News, while fox hunting news should return news stories about fox hunting.

To facilitate these types of queries on full-text fields, Elasticsearch first analyzes the text, and then uses the results to build an inverted index. We will discuss the inverted index and the analysis process in the next two sections.


## [Inverted Index](https://www.elastic.co/guide/en/elasticsearch/guide/current/inverted-index.html)

This whole page is amazing. Just read it if you want more info. 

The "inverted index" is an index that Elasticsearch produces to determine textual similarity, including case handling, mapping conjugations to stem/root words, and identifying synonyms. The result is the ability to not only find relevant results, but also to score them by relevance. 


## [Analysis and Analyzers](https://www.elastic.co/guide/en/elasticsearch/guide/current/analysis-intro.html)

Analysis is a process that consists of the following:

* First, tokenizing a block of text into individual terms suitable for use in an inverted index,
* Then normalizing these terms into a standard form to improve their “searchability,” or recall

This job is performed by analyzers. An analyzer is really just a wrapper that combines three functions into a single package:

* Character filters
    * First, the string is passed through any character filters in turn. Their job is to tidy up the string before tokenization. A character filter could be used to strip out HTML, or to convert & characters to the word and.
* Tokenizer
    * Next, the string is tokenized into individual terms by a tokenizer. A simple tokenizer might split the text into terms whenever it encounters whitespace or punctuation.
* Token filters
    * Last, each term is passed through any token filters in turn, which can change terms (for example, lowercasing Quick), remove terms (for example, stopwords such as a, and, the) or add terms (for example, synonyms like jump and leap).

Elasticsearch provides many character filters, tokenizers, and token filters out of the box. These can be combined to create custom analyzers suitable for different purposes. We discuss these in detail in [Custom Analyzers](https://www.elastic.co/guide/en/elasticsearch/guide/current/custom-analyzers.html).

See list of [built-in analyzers](https://www.elastic.co/guide/en/elasticsearch/guide/current/analysis-intro.html#_built_in_analyzers).

The `_analyze` API can be used to see the result of applying a given analyzer to some text. See sense_reference for a sample usage.

### When Analyzers are used

When we index a document, its full-text fields are analyzed into terms that are used to create the inverted index. However, when we search on a full-text field, we need to pass the query string through the same analysis process, to ensure that we are searching for terms in the same form as those that exist in the index.

Full-text queries, which we discuss later, understand how each field is defined, and so they can do the right thing:

* When you query a full-text field, the query will apply the same analyzer to the query string to produce the correct list of terms to search for.
* When you query an exact-value field, the query will not analyze the query string, but instead search for the exact value that you have specified.

### Specifying Analyzers

When Elasticsearch detects a new string field in your documents, it automatically configures it as a full-text string field and analyzes it with the standard analyzer.

You don’t always want this. Perhaps you want to apply a different analyzer that suits the language your data is in. And sometimes you want a string field to be just a string field—to index the exact value that you pass in, without any analysis, such as a string user ID or an internal status field or tag.

To achieve this, we have to configure these fields manually by specifying the mapping. See the next section for more information.


## [Mapping](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping-intro.html)

In order to be able to treat date fields as dates, numeric fields as numbers, and string fields as full-text or exact-value strings, Elasticsearch needs to know what type of data each field contains. This information is contained in the mapping.

Each document in an index has a type. Every type has its own mapping, or schema definition. A mapping defines the fields within a type, the datatype for each field, and how the field should be handled by Elasticsearch. A mapping is also used to configure metadata associated with the type.

We discuss mappings in detail in [Types and Mappings](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping.html). In this section, we’re going to look at just enough to get you started.

### Core Simple Field Types

Elasticsearch supports the following simple field types:

* String: string
* Whole number: byte, short, integer, long
* Floating-point: float, double
* Boolean: boolean
* Date: date


When you index a document that contains a new field—one previously not seen—Elasticsearch will use [dynamic mapping](https://www.elastic.co/guide/en/elasticsearch/guide/current/dynamic-mapping.html) to try to guess the field type from the basic datatypes available in JSON, using the following rules:

**JSON type     ===>     Field type**
* Boolean: true or false     ===>     boolean
* Whole number: 123     ===>     long
* Floating point: 123.45     ===>     double
* String, valid date: 2014-09-15     ===>     date
* String: foo bar     ===>     string

This means that if you index a number in quotes ("123"), it will be mapped as type string, not type long. However, if the field is already mapped as type long, then Elasticsearch will try to convert the string into a long, and throw an exception if it can’t.

### Viewing the Mapping

The mapping for the fiels for a document type can be retrieved like this:
```
GET /gb/_mapping/tweet
```

This shows us the mapping for the fields (called *properties*) that Elasticsearch generated dynamically from the documents that we indexed:
```
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

**Note:** Don't assume your mapping is correct, because having the wrong type dynamically assigned can produce strange results in your queries. Check the mapping, or set it yourself to make sure that everything behaves as expected!


### Customizing Field Mappings

While the basic field datatypes are sufficient for many cases, you will often need to customize the mapping for individual fields, especially string fields. Custom mappings allow you to do the following:

* Distinguish between full-text string fields and exact value string fields
* Use language-specific analyzers
* Optimize a field for partial matching
* Specify custom date formats
* And much more

The most important attribute of a field is the type. For fields other than string fields, you will seldom need to map anything other than type:
```
{
    "number_of_clicks": {
        "type": "integer"
    }
}
```

Fields of type string are, by default, considered to contain full text. That is, their value will be passed through an analyzer before being indexed, and a full-text query on the field will pass the query string through an analyzer before searching.

The two most important mapping attributes for string fields are index and analyzer.

#### index

The index attribute controls how the string will be indexed. It can contain one of three values:

* `analyzed`
    * First analyze the string and then index it. In other words, index this field as full text.
* `not_analyzed`
    * Index this field, so it is searchable, but index the value exactly as specified. Do not analyze it.
* `no`
    * Don’t index this field at all. This field will not be searchable.

The default value of index for a string field is analyzed. If we want to map the field as an exact value, we need to set it to not_analyzed:
```
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}
```

#### analyzer

For analyzed string fields, use the analyzer attribute to specify which analyzer to apply both at search time and at index time. By default, Elasticsearch uses the standard analyzer, but you can change this by specifying one of the built-in analyzers, such as whitespace, simple, or english:
```
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```

### Updating a Mapping

You can specify the mapping for a type when you first create an index. Alternatively, you can add the mapping for a new type (or update the mapping for an existing type) later, using the /_mapping endpoint.

Although you can add to an existing mapping, you can’t change existing field mappings. If a mapping already exists for a field, data from that field has probably been indexed. If you were to change the field mapping, the indexed data would be wrong and would not be properly searchable.

We can update a mapping to add a new field, but we can’t change an existing field from analyzed to not_analyzed.

You can create an index with a mapping specified like this:
```
PUT /gb 
{
  "mappings": {
    "tweet" : {
      "properties" : {
        "tweet" : {
          "type" :    "string",
          "analyzer": "english"
        },
        "date" : {
          "type" :   "date"
        },
        "name" : {
          "type" :   "string"
        },
        "user_id" : {
          "type" :   "long"
        }
      }
    }
  }
}
```

See sense_reference for more examples.


## [Complex Core Field Types](https://www.elastic.co/guide/en/elasticsearch/guide/current/complex-core-fields.html)

Besides the simple scalar datatypes that we have mentioned, JSON also has null values, arrays, and objects, all of which are supported by Elasticsearch.

For arrays, all entries must have the same data type.

See this page for info on multivalue fields (arrays), empty fields, multilevel objects, mapping for inner objects, and arrays of inner objects. 


# [Full-Body Search](https://www.elastic.co/guide/en/elasticsearch/guide/current/full-body-search.html)

Search lite—a query-string search—is useful for ad hoc queries from the command line. To harness the full power of search, however, you should use the request body search API, so called because most parameters are passed in the HTTP request body instead of in the query string.

Request body search—henceforth known as search—not only handles the query itself, but also allows you to return highlighted snippets from your results, aggregate analytics across all results or subsets of results, and return did-you-mean suggestions, which will help guide your users to the best results quickly.


## [Empty Search](https://www.elastic.co/guide/en/elasticsearch/guide/current/_empty_search.html)

You can perform an empty search using `GET` requests that include a body! Sometimes this isn't allowed, so the `_search` API can be used with both `GET` and `POST`. 

An empty search would look like this:
```
GET /_search
{}
```

And you can use the from and size parameters for pagination:
```
GET /_search
{
  "from": 30,
  "size": 10
}
```
OR
```
POST /_search
{
  "from": 30,
  "size": 10
}
```


## [Query DSL](https://www.elastic.co/guide/en/elasticsearch/guide/current/query-dsl-intro.html)

The query DSL is a flexible, expressive search language that Elasticsearch uses to expose most of the power of Lucene through a simple JSON interface. It is what you should be using to write your queries in production. It makes your queries more flexible, more precise, easier to read, and easier to debug.

To use the Query DSL, pass a query in the query parameter:
```
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```

The empty search is functionally equivalent to a `match_all` query clause:
```
GET /_search
{
    "query" : {
        "match_all" : {}
    }
}
```

### Structure of a query clause

The "query clause" replaces "YOUR_QUERY_HERE" above, and has this structure:
```
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```

If it references one particular field, it has this structure:
```
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

For instance, you can use a match query clause to find tweets that mention elasticsearch in the tweet field:
```
{
    "match": {
        "tweet": "elasticsearch"
    }
}
```

The full search request would look like this:
```
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

### Combining Multiple Query Clauses

Any complete query clauses can be combined underneath the parent `query` element using the `bool` clause. The nesting can be arbitrarily deep. 

At the top level under `query`, you can only have ONE CLAUSE. This clause can contain as many clauses as you want though. When using `bool`, the next level down is `must`, `should`, and `must_not`. Underneath each of these, you can use array notation `[...]`, to comma separate entire query clauses. You could, for example, have 2 `match` clauses under the same `should` clause. This would NOT be allowed directly under the `query` level. 

See sense_reference for some examples. **Note that the Elastic guide actually has malformed queries in it**. 


## [Queries and Filters](https://www.elastic.co/guide/en/elasticsearch/guide/current/_queries_and_filters.html)

Although we refer to the query DSL, in reality there are two DSLs: the query DSL and the filter DSL. Query clauses and filter clauses are similar in nature, but have slightly different purposes.

A filter asks a yes|no question of every document and is used for fields that contain exact values:

* Is the created date in the range 2013 - 2014?
* Does the status field contain the term published?
* Is the lat_lon field within 10km of a specified point?

A query is similar to a filter, but also asks the question: How well does this document match?

A typical use for a query is to find documents:

* Best matching the words full text search
* Containing the word run, but maybe also matching runs, running, jog, or sprint
* Containing the words quick, brown, and fox—the closer together they are, the more relevant the document
* Tagged with lucene, search, or java—the more tags, the more relevant the document

A query calculates how *relevant* each document is to the query, and assigns it a relevance \_score, which is later used to sort matching documents by relevance. This concept of relevance is well suited to full-text search, where there is seldom a completely “correct” answer.

### Performance Differences

The output from most filter clauses—a simple list of the documents that match the filter—is quick to calculate and easy to cache in memory, using only 1 bit per document. These cached filters can be reused efficiently for subsequent requests.

Queries have to not only find matching documents, but also calculate how relevant each document is, which typically makes queries heavier than filters. Also, query results are not cachable.

Thanks to the inverted index, a simple query that matches just a few documents may perform as well or better than a cached filter that spans millions of documents. In general, however, a cached filter will outperform a query, and will do so consistently.

The goal of filters is to reduce the number of documents that have to be examined by the query.

### When to Use Which

As a general rule, use query clauses for full-text search or for any condition that should affect the relevance score, and use filter clauses for everything else.


## [Most Important Queries and Filters](https://www.elastic.co/guide/en/elasticsearch/guide/current/_most_important_queries_and_filters.html)

Below is a list of the most important queries and filters. See the page for examples.

### Filters

* term
* terms
* range
    * gt
    * gte
    * lt
    * lte
* exists
* missing
* bool
    * must
    * must_not
    * should (at least one clause must match, like an `or` statement)

### Queries

* match_all
* match
* multi_match
* bool (slightly different from `bool` filter; see `should` parameter)
    * must
    * must_not
    * should (if these clauses match, they increase the "score"; otherwise they have no effect)

**Note:** if there are no `must` clauses, at least one `should` clause has to match for the document to be returned. If there is a `must` clause, no `should` clauses are required to match. 


## [Combining Queries with Filters](https://www.elastic.co/guide/en/elasticsearch/guide/current/_combining_queries_with_filters.html)

Parameters with `query` or `filter` in the name establish the outer *context* as a query or filter context, and expect a single argument containing either a single query or filter clause. 

Compound query clauses can wrap around other query clauses, and compound filter clauses can wrap other filter clauses. However, it is often useful to apply a filter to a query or, less frequently, to use a full-text query as a filter.

To do this, there are dedicated query clauses that wrap filter clauses, and vice versa, thus allowing us to switch from one context to another. It is important to choose the correct combination of query and filter clauses to achieve your goal in the most efficient way.


## [Validating Queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/_validating_queries.html)

Queries can become quite complex and, especially when combined with different analyzers and field mappings, can become a bit difficult to follow. The validate-query API can be used to check whether a query is valid.

You use `_validate` like this:
```
GET /gb/tweet/_validate/query
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```

You can add `?explain` to the query string to see an explanation of why the query is invalid, or how the successfull query actually worked:
```
GET /gb/tweet/_validate/query?explain 
{
...
}
```

For valid queries, this will show how the query is performed based on the `analyzer` being used. 


# [Sorting and Relevance](https://www.elastic.co/guide/en/elasticsearch/guide/current/sorting.html)

I will return!


# [Mapper Attachments Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments.html)

The mapper attachments plugin lets Elasticsearch index file attachments in common formats (such as PPT, XLS, PDF) using the Apache text extraction library [Tika](http://tika.apache.org/).

In practice, the plugin adds the attachment type when mapping properties so that documents can be populated with file attachment contents (encoded as base64).

The full reference guide on Mapping has everything you need to use mapping properly and can be found [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html). Setting up mapping properly is the KEY to making attachments work properly. 


## [Hello, world](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments-helloworld.html)

See sense_reference for example.


## [Mapper Usage](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments-usage.html)

The attachment type not only indexes the content of the doc in content sub field, but also automatically adds meta data on the attachment as well (when available).

The metadata supported are:

* date
* title
* name only available if you set \_name see above
* author
* keywords
* content_type
* content_length is the original content_length before text extraction (aka file size)
* language

They can be queried using the "dot notation", for example: my_attachment.author.


## [Querying of accessing metadata](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments-querying-metadata.html)

See sense_reference.


## [Indexed Characters](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments-indexed-characters.html)

By default, 100000 characters are extracted when indexing the content. This default value can be changed by setting the index.mapping.attachment.indexed_chars setting. It can also be provided on a per document indexed using the \_indexed_chars parameter. -1 can be set to extract all text, but note that all the text needs to be allowed to be represented in memory:
```
PUT /test/person/1
{
    "my_attachment" : {
        "_indexed_chars" : -1,
        "_content" : "... base64 encoded attachment ..."
    }
}
```


## [Metadata parsing error handling](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments-error-handling.html)

While extracting metadata content, errors could happen for example when parsing dates. Parsing errors are ignored so your document is indexed.

You can disable this feature by setting the index.mapping.attachment.ignore_errors setting to false.


## [Language Detection](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments-language-detection.html)

By default, language detection is disabled (false) as it could come with a cost. This default value can be changed by setting the index.mapping.attachment.detect_language setting. It can also be provided on a per document indexed using the \_detect_language parameter.

Note that you can force language using \_language field when sending your actual document:
```
{
    "my_attachment" : {
        "_language" : "en",
        "_content" : "... base64 encoded attachment ..."
    }
}
```


## [Highlighting Attachments](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-attachments-highlighting.html)

If you want to highlight search results from attachments, this is what you need! See sense_reference for a complete example. 

The key is to set "store" : true and "term_vector" : "with_positions_offsets". 


## [Mapper Size Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/2.2/mapper-size.html)

If you want to store and use the file size for documents, you can use the mapper-size plugin. Start on this page for more information. 

