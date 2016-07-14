# [Elasticsearch Javascript API Guide](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference-2-2.html#api-search-2-2)

This guide describes the usage of the Elastic Javascript API.

If you are looking to get up and running with a complete basic example, see [Getting Started](#getting-started).

## [API Conventions](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-conventions.html)

The API has generic parameters that are accepted by all API methods. By "methods", we mean anything on [this page](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference-2-2.html). These generic parameters include:
* method - (String) The HTTP method to use for this request. All of the API methods have their own default.
* body - (String, Anything) The body to send along with this request. If the body is a string it will be passed along as is, otherwise it is passed to the serializer and converted to either JSON or a newline seperated list of JSON objects based on the API method.
* ignore - (Number, Number[]) HTTP status codes which should not be considered errors for this request.
* filterPath - (String|String[]) Starting in elasticsearch 1.6 the filterPath parameter can be passed to any API to filter it’s reponse values. See the elasticsearch response filtering docs for more information.

### Overriding per request

These config values can be overriden per request:
* requestTimeout - [more info](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html#config-request-timeout)
* maxRetries - [more info](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html#config-max-retries)


### Callbacks or Promises

When a callback is passed to any of the API methods, it will be called with (error, response, status). If you prefer to use promises, don’t pass a callback and a promise will be returned. The promise will either be resolved with the response body, or rejected with the error that occured (including any 300+ response for non "exists" methods).

Both styles of calling the API will return an object (either a promise or just a plain object) which has an abort() method. Calling that abort method ends the HTTP request, but it will not end the work Elasticsearch is doing.


## [Configuration](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html)

First, a connection must be established to an Elasticsearch cluster through a client. 

The `Client` constructor accepts a single object as its argument. In the Config options section all of the available options/keys are listed.
```
var elasticsearch = require('elasticsearch');
var client = new elasticsearch.Client({
  ... config options ...
});
```

A full list of config options can be found [here](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html#config-options).


## [API 2.2](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference-2-2.html)

The entire list of API methods can be found on this page. It is a great reference guide that includes examples. 

Note that you can use the `body` parameter on these methods in exactly the same way that it is used in Sense. This should allow for us to copy/paste JSONs included with requests, and they should work. Each method in the API has its own default HTTP method, so we shouldn't need to worry about this unless we need to override it for some reason. 

Almost all of the methods on the client accept two arguments:

* params - an optional object/hash of parameters More info here.
* callback - an optional function that will be called with the final result of the method. When omitted, a promise is returned. More info here.

Therefore, an example call will be structured like this:
```
client.exampleMethod(
  // first "argument" = params
  {
    param1: 30000,
    param2: "sample param",

    // undocumented params are appended to the query string
    hello: "elasticsearch", // ==> "/?hello=elasticsearch"
    extra: "more stuff"
  }, 
  // second argument = callback (optional function)
  callbackFunction (error, response, status) {
    if (error) {
      // Handle the error
      console.error('elasticsearch cluster is down!');
    } else {
      // Do something with the JSON response!
      console.log('All is well');
    }
  }
);
```


## Getting Started

This section provides a complete basic example that is used to get an app up and running. This app simply has a search bar that sends a query to Elasticsearch and expects highlighted search results to come back. It is loosely based on the [Quick Start Guide](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/quick-start.html#_creating_a_client), but has modifications for our particular application. 

### Creating a client

Create a client that talks to our Elasticsearch cluster hosted on AWS. We want this to use the latest version of the API, so we set `apiVersion` to "2.2".
```
var elasticsearch = require('elasticsearch');
var client = new elasticsearch.Client({
  apiVersion: '2.2', 
  host: 'ec2-54-183-145-73.us-west-1.compute.amazonaws.com:32780',
  log: 'trace'
})m
```

We want to write a function that can be called from the user-interface to query Elasticsearch for documents that match the query input by the user. 

Let's start by making sure that we can talk to our cluster by using the `client.ping` method:
```
client.ping({
  requestTimeout: 30000,

  // undocumented params are appended to the query string
  hello: "elasticsearch"
}, function (error) {
  if (error) {
    console.error('elasticsearch cluster is down!');
  } else {
    console.log('All is well');
  }
});
```

Now we write a search function that is called with a single parameter, `user_query`, and then uses the API `search` method of `client` to query Elasticsearch. Our content is upload as files using the "mapper-attachment". 
```
function search(user_query) {
  client.search(
    {
      index : 'documents',
      body : {
        query : {
          match : {
            _all: user_query
          }
        },
        highlight : {
          fields : {
            "file.content" : {}
          }
        }
      }
    }, function (error, response, status) {
    if (error) {
      // handle error
      throw error;
    } else {
      // handle response
      console.log("search successful");
    }
  });
}
```

We can rewrite the same function using a concept called "promises". Promises are described in more detail [here](https://github.com/cujojs/when/blob/master/docs/api.md#promise), but they are defined as:
> a proxy for a value that isn't available yet allowing you to interact with it as if it is.

You can apply the `promise.then(onFulfilledFunction)` to define the behavior that should be taken once the promise is "fulfilled". In this case, the `onFulfilledFunction` will define what we do once we get the response back. This function will look something like this:
```
function (response) {...}
```

Promises are used with an Elasticsearch API method like this:
```
client.search({
  param: "value",
  body : {...}
}).then(function (response) {
    // do something with the response
    ...
}, function (error) {
  console.trace(error.message);
});
```

Notice how the `search` method ends early (before getting its second argument), and the `promise.then` method is called. This is possible because `client.search()` returns a `promise`.  With this setup, the error handling function at the end of the above code is a second argument of `promise.then` instead of a second argument of `client.search`. 

So let's see how we can rewrite our search call using promises. It looks a bit cleaner and seems easier to understand:
```
function search(user_query) {
  client.search(
  {
    index : 'documents',
    body : {
      query : {
        match : {
          _all: user_query
        }
      },
      highlight : {
        fields : {
          "file.content" : {}
        }
      }
    }
  }).then(function (response) {
    // Handle response
    console.log("search successful");
  }, function (error) {
    console.trace(error.message);
  });
}
```


## Misc.

---

### Content-Type Request Header, JSON

When using `curl` from the command line, if you send a JSON using the `-d '{"json":"here"}'` argument, the web application receiving the request won't necessarily know how to handle the data packet. By default, it seems that the data packet is treated a single string. If however, you include a header that specifies the content type of the data to be a JSON, then the application can interpret it as such.

Here is there is the header to specify content type as a JSON:
> Content-Type: application/json

When using this with `curl`, use it like this:
> curl --header "Content-Type: application/json"
OR
> curl -H "Content-Type: application/json"

---
