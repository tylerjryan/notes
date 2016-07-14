# Authentication

This guide describes how to authenticate with a secure Elasticsearch cluster.


## [Javascript Client](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/auth-reference.html)

You can configure the client to use SSL for connecting to your elasticsearch cluster, including certificate verification and http auth.

### Basic Auth

You can include the credentials directly as a property of the host config:
```
var client = new elasticsearch.Client({
  host: [
    {
      host: 'es1.internal.org',
      auth: 'user:password'
    }
  ]
});
```

### HTTPS/SSL (Recommended)

It is recommended that you use SSL instead of storing credentials in clear text in your client configuration. 

Without any additional configuration you can specify https:// host urls, but the certificates used to sign these requests will not verified (rejectUnauthorized: false). To turn on certificate verification you must specify an ssl: object either in the top level config or in each host config object and set rejectUnauthorized: true. The ssl config object can contain many of the same configuration options that tls.connect() accepts. For convenience these options are also listed in the configuration reference.

Specify a certifact authority that should be used to verify server certificates on all nodes:
```
var client = new elasticsearch.Client({
  hosts: [
    'https://box1.internal.org',
    'https://box2.internal.org',
    'https://box3.internal.org'
  ],
  ssl: {
    ca: fs.readFileSync('./cacert.pem'),
    rejectUnauthorized: true
  }
});
```

