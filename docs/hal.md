#HTTP Server

Thr HTTP server for SqlStreamStore is implemented as an aspnet core middleware.

##Initialization

You are responsibile for initializing the underlying store. Please refer to the documentation of the store that you are using.

##Settings

There is an extension method named `UseSqlStreamStoreHal` located in the namespace `SqlStreamStore.HAL`. It applies to `IApplicationBuilder`. It has two parameters, one optional:
```
public void Configure(IApplicationBuilder app) => app
    .UseSqlStreamStoreHal(IStreamStore streamStore, SqlStreamStoreMiddlewareOptions options = default);

```
where `streamStore` is an implementation of `IStreamStore`, and options (optional) is an instance of `SqlStreamStoreMiddlewareOptions`. This options class has a single configuration value: `UseCanonicalUrls`. If `true`, the middleware will sort your query string parameters and issue a redirect if the query string parameters you used are not expressed in the same order. This is to aid in HTTP caching. Set to `false` if your reverse proxy already sorts your query string parameters, e.g., AWS API Gateway. Defaults to `true`.

##Concepts

`SQLStreamStore.HAL` uses `application/hal+json` to deliver responses. As this is at its heart a RESTful protocol, you may leverage all the usual HTTP techniques to scale out horizontally.

In addition, any non-safe operations (i.e., writes) you may execute on the current resource will be presented to you as `json-hyper-schema` objects as embedded resources. Also, within these `json-hyper-schema` resources, hints may be provided as to how they should be rendered. These are part of the `x-schema-form` sub-property.

##Operations

All of the samples below assume you have `SQLStreamStore.HAL` running on `localhost:5000`.

The `href` property of any link will always be relative to the url that the request was sent to. It will indicate which content type you can `Accept`. If a url has more than one representation, then a link for each `Content-Type` will be included in the response. A link to `self` should also be included with every response.

###Index

The index resource (`/`) provides the main entry point into the rest of the server. You may bookmark this url. 

```
curl -H 'accept: application/hal+json' http://localhost:5000/
```

```
HTTP/1.1 200 OK
Date: Mon, 17 Dec 2018 21:30:17 GMT
Content-Type: application/hal+json
Transfer-Encoding: chunked
Vary: Accept

{
  "provider": "InMemory",
  "versions": {
    "streamStore": "1.2.0-beta.2+build.279",
    "server": "1.0.0-rc.3.15"
  },
  "_links": {
    "streamStore:index": {
      "href": "./",
      "type": "application/hal+json",
      "title": "Index"
    },
    "self": {
      "href": "./",
      "type": "application/hal+json",
      "title": "Index"
    },
    "streamStore:find": {
      "href": "./streams/{streamId}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Find a Stream"
    },
    "streamStore:feed-browser": {
      "href": "./streams{?p,t,m}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Browse Streams"
    },
    "streamStore:feed": {
      "href": "./stream",
      "type": "application/hal+json"
    },
    "curies": {
      "href": "./docs/{rel}",
      "templated": true,
      "type": "text/markdown",
      "name": "streamStore",
      "title": "Documentation",
      "hreflang": "en"
    }
  }
}
```

###Read All Stream

The all stream resource (`/stream`) links to every event currently in the Stream Store, in the order that they were written. You may bookmark this url.

```
curl -H 'accept: application/hal+json' http://localhost:5000/stream
```

```
HTTP/1.1 200 OK
Date: Mon, 17 Dec 2018 21:40:07 GMT
Content-Type: application/hal+json
Cache-Control: max-age=0,no-cache,must-revalidate
Transfer-Encoding: chunked
ETag: "999"
Vary: Accept
SSS-HeadPosition: 999

{
  "fromPosition": 999,
  "nextPosition": 979,
  "isEnd": false,
  "_links": {
    "streamStore:index": {
      "href": "",
      "type": "application/hal+json",
      "title": "Index"
    },
    "streamStore:find": {
      "href": "streams/{streamId}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Find a Stream"
    },
    "streamStore:feed-browser": {
      "href": "streams{?p,t,m}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Browse Streams"
    },
    "first": {
      "href": "stream?d=f&m=20&p=0&e=0",
      "type": "application/hal+json"
    },
    "previous": {
      "href": "stream?d=b&m=20&p=979&e=0",
      "type": "application/hal+json"
    },
    "streamStore:feed": {
      "href": "stream?d=b&m=20&p=-1&e=0",
      "type": "application/hal+json"
    },
    "self": {
      "href": "stream?d=b&m=20&p=-1&e=0",
      "type": "application/hal+json"
    },
    "last": {
      "href": "stream?d=b&m=20&p=-1&e=0",
      "type": "application/hal+json"
    },
    "curies": {
      "href": "docs/{rel}",
      "templated": true,
      "type": "text/markdown",
      "name": "streamStore",
      "title": "Documentation",
      "hreflang": "en"
    }
  },
  "_embedded": {
    "streamStore:message": [
      {
        "messageId": "f24a1f39-8e68-4f65-8138-b8ff6d550f44",
        "createdUtc": "2018-12-17T21:20:02.7616568Z",
        "position": 999,
        "streamId": "test-8d807c28fb3d4138b067ddab08bd99fd",
        "streamVersion": 9,
        "type": "test",
        "metadata": "{}",
        "_links": {
          "streamStore:message": {
            "href": "streams/test-8d807c28fb3d4138b067ddab08bd99fd/9",
            "type": "application/hal+json",
            "title": "test-8d807c28fb3d4138b067ddab08bd99fd@9"
          },
          "self": {
            "href": "streams/test-8d807c28fb3d4138b067ddab08bd99fd/9",
            "type": "application/hal+json",
            "title": "test-8d807c28fb3d4138b067ddab08bd99fd@9"
          },
          "streamStore:feed": {
            "href": "streams/test-8d807c28fb3d4138b067ddab08bd99fd",
            "type": "application/hal+json",
            "title": "test-8d807c28fb3d4138b067ddab08bd99fd"
          },
          "curies": {
            "href": "docs/{rel}",
            "templated": true,
            "type": "text/markdown",
            "name": "streamStore",
            "title": "Documentation",
            "hreflang": "en"
          }
        }
      }
    ]
  }
}
```

The messages on this page will be placed in an array as an embedded resource with the rel `streamStore:message`. Only one message is shown for brevity.

You will notice some standard link relations present in the response: `next`, `prev`, `first`, and `last`. These behave exactly the same way as they do with atom: The `first` link will take you to the beginning (i.e., position 0). The `last` will take you to the end.

Lastly, the `streamStore:feed` relation of the _embedded_ message refers to the stream that message belongs to.

###Read Stream

The stream resource (`/streams/{streamId}`) links to every event in a single stream, in the order that they were written. You may bookmark this url.

```
curl -i -L -H 'accept: application/hal+json' http://localhost:5000/streams/test-8d807c28fb3d4138b067ddab08bd99fd
```

```
HTTP/1.1 308 Permanent Redirect
Date: Mon, 17 Dec 2018 21:51:45 GMT
Content-Type: application/hal+json
Transfer-Encoding: chunked
Location: ../streams/test-8d807c28fb3d4138b067ddab08bd99fd?d=b&m=20&p=-1&e=0
Vary: Accept

HTTP/1.1 200 OK
Date: Mon, 17 Dec 2018 21:51:45 GMT
Content-Type: application/hal+json
Transfer-Encoding: chunked
ETag: "9"
Vary: Accept

{
  "lastStreamVersion": 9,
  "lastStreamPosition": 999,
  "fromStreamVersion": -1,
  "nextStreamVersion": -1,
  "isEnd": true,
  "_links": {
    "streamStore:index": {
      "href": "../",
      "type": "application/hal+json",
      "title": "Index"
    },
    "streamStore:find": {
      "href": "../streams/{streamId}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Find a Stream"
    },
    "streamStore:feed-browser": {
      "href": "../streams{?p,t,m}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Browse Streams"
    },
    "first": {
      "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd?d=f&m=20&p=0&e=0",
      "type": "application/hal+json"
    },
    "streamStore:feed": {
      "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd?d=b&m=20&p=-1&e=0",
      "type": "application/hal+json",
      "title": "test-8d807c28fb3d4138b067ddab08bd99fd"
    },
    "self": {
      "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd?d=b&m=20&p=-1&e=0",
      "type": "application/hal+json",
      "title": "test-8d807c28fb3d4138b067ddab08bd99fd"
    },
    "last": {
      "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd?d=b&m=20&p=-1&e=0",
      "type": "application/hal+json"
    },
    "streamStore:metadata": {
      "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd/metadata",
      "type": "application/hal+json"
    },
    "curies": {
      "href": "../docs/{rel}",
      "templated": true,
      "type": "text/markdown",
      "name": "streamStore",
      "title": "Documentation",
      "hreflang": "en"
    }
  },
  "_embedded": {
    "streamStore:append": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "Append to Stream",
      "type": "object",
      "required": [
        "messageId",
        "type"
      ],
      "properties": {
        "messageId": {
          "type": "string",
          "pattern": "^[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}$",
          "x-schema-form": {
            "key": "messageId",
            "type": "uuid"
          }
        },
        "type": {
          "type": "string"
        },
        "jsonData": {
          "type": "object",
          "x-schema-form": {
            "key": "jsonData",
            "type": "textarea",
            "rows": 30
          }
        },
        "jsonMetadata": {
          "type": "string",
          "x-schema-form": {
            "key": "jsonMetadata",
            "type": "textarea",
            "rows": 30
          }
        }
      }
    },
    "streamStore:delete-stream": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "Delete Stream",
      "type": "object"
    },
    "streamStore:message": [
      {
        "messageId": "f24a1f39-8e68-4f65-8138-b8ff6d550f44",
        "createdUtc": "2018-12-17T21:20:02.7616568Z",
        "position": 999,
        "streamId": "test-8d807c28fb3d4138b067ddab08bd99fd",
        "streamVersion": 9,
        "type": "test",
        "metadata": "{}",
        "_links": {
          "streamStore:message": {
            "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd/9",
            "type": "application/hal+json",
            "title": "test-8d807c28fb3d4138b067ddab08bd99fd@9"
          },
          "self": {
            "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd/9",
            "type": "application/hal+json",
            "title": "test-8d807c28fb3d4138b067ddab08bd99fd@9"
          },
          "streamStore:feed": {
            "href": "../streams/test-8d807c28fb3d4138b067ddab08bd99fd",
            "type": "application/hal+json",
            "title": "test-8d807c28fb3d4138b067ddab08bd99fd"
          },
          "curies": {
            "href": "../docs/{rel}",
            "templated": true,
            "type": "text/markdown",
            "name": "streamStore",
            "title": "Documentation",
            "hreflang": "en"
          }
        }
      }
    ]
  }
}
```

Please note the use of `-L` in curl - follow redirects. Although the stream url (`/streams/{streamId}`) is bookmarkable, the url it redirects to _is not_. For ease of use, please make sure your client is configured to automatically follow redirects.

The resource returned from this url behaves almost exactly like `/stream`. As above, only one message is shown for brevity. Additionally, embedded resources that describe the allowed operations on the stream will be presented to you: `streamStore:append`, and `streamStore:delete-stream`.

###Read Single Message

The single message resource (`/streams/{streamId}/{streamVersion}`) contains information about a single message. You may bookmark this url.

```
curl -i -H 'accept: application/hal+json' http://localhost:5000/streams/test-8d807c28fb3d4138b067ddab08bd99fd/0
```

```
HTTP/1.1 200 OK
Date: Mon, 17 Dec 2018 22:07:00 GMT
Content-Type: application/hal+json
Cache-Control: max-age=31536000
Transfer-Encoding: chunked
ETag: "0"
Vary: Accept

{
  "messageId": "3d4953db-bdf2-427c-bc3d-3d3cd5c9cce9",
  "createdUtc": "2018-12-17T21:20:02.7616535Z",
  "position": 990,
  "streamId": "test-8d807c28fb3d4138b067ddab08bd99fd",
  "streamVersion": 0,
  "type": "test",
  "payload": "{ \"foo\": \"b922a8e3-6880-4bfe-b3ea-aedd93673823\", \"baz\": {  }, \"qux\": [ 0, 0, 1, 0, 3, 1, 4, 2, 4, 8 ] }",
  "metadata": "{}",
  "_links": {
    "streamStore:index": {
      "href": "../../",
      "type": "application/hal+json",
      "title": "Index"
    },
    "streamStore:find": {
      "href": "../../streams/{streamId}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Find a Stream"
    },
    "streamStore:feed-browser": {
      "href": "../../streams{?p,t,m}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Browse Streams"
    },
    "first": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd/0",
      "type": "application/hal+json"
    },
    "next": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd/1",
      "type": "application/hal+json"
    },
    "last": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd/-1",
      "type": "application/hal+json"
    },
    "streamStore:feed": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd?d=b&m=20&p=-1&e=0",
      "type": "application/hal+json",
      "title": "test-8d807c28fb3d4138b067ddab08bd99fd"
    },
    "streamStore:message": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd/0",
      "type": "application/hal+json",
      "title": "test-8d807c28fb3d4138b067ddab08bd99fd@0"
    },
    "self": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd/0",
      "type": "application/hal+json",
      "title": "test-8d807c28fb3d4138b067ddab08bd99fd@0"
    },
    "curies": {
      "href": "../../docs/{rel}",
      "templated": true,
      "type": "text/markdown",
      "name": "streamStore",
      "title": "Documentation",
      "hreflang": "en"
    }
  },
  "_embedded": {
    "streamStore:delete-message": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "Delete Stream Message",
      "type": "object"
    }
  }
}
```

Notice here that the message _does not_ come back as an embedded resource, but rather as the resource itself. As in `SQLStreamstore`, the `payload` and `metadata` are stored as strings. _editor's note: this may change before release!_

###Read Stream Metadata

The stream metadata resource (`/streams/{streamId}/metadata`) contains information about a stream's metadata - well known properties include `maxCount`, which is the maximum number of messages allowed in the stream, and `maxAge`, the maximum age of a message in seconds. Any additional metadata that was written to the stream will also be inlcluded here.

Unlike other urls, this url is subject to change and is therefore _not_ bookmarkable.

```
curl -i -H 'accept: application/hal+json' http://localhost:5000/streams/test-8d807c28fb3d4138b067ddab08bd99fd/metadata
```

```
{
  "streamId": "test-8d807c28fb3d4138b067ddab08bd99fd",
  "metadataStreamVersion": -1,
  "_links": {
    "streamStore:index": {
      "href": "../../",
      "type": "application/hal+json",
      "title": "Index"
    },
    "streamStore:find": {
      "href": "../../streams/{streamId}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Find a Stream"
    },
    "streamStore:feed-browser": {
      "href": "../../streams{?p,t,m}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Browse Streams"
    },
    "streamStore:metadata": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd/metadata",
      "type": "application/hal+json"
    },
    "self": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd/metadata",
      "type": "application/hal+json"
    },
    "streamStore:feed": {
      "href": "../../streams/test-8d807c28fb3d4138b067ddab08bd99fd",
      "type": "application/hal+json",
      "title": "test-8d807c28fb3d4138b067ddab08bd99fd"
    },
    "curies": {
      "href": "../../docs/{rel}",
      "templated": true,
      "type": "text/markdown",
      "name": "streamStore",
      "title": "Documentation",
      "hreflang": "en"
    }
  },
  "_embedded": {
    "streamStore:metadata": {
      "title": "Set Stream Metadata",
      "type": "object",
      "$schema": "http://json-schema.org/draft-07/schema#",
      "properties": {
        "maxCount": {
          "type": "integer",
          "minimum": 1
        },
        "maxAge": {
          "type": "integer",
          "minimum": 1
        },
        "metadataJson": {
          "type": "object",
          "x-schema-form": {
            "key": "metadataJson",
            "type": "textarea",
            "rows": 30
          }
        }
      }
    }
  }
}
```

Only one operation is allowed on this resource: `streamStore:metadata`, which allows you to change the metadata associated with its parent stream.

###Append to Stream

There are two slightly different ways to append to a stream: multiple messages or a single message.

_Editor's note: this is not final and is therefore subject to change!_

With a single message:
```
curl -i -H 'sss-expected-version: -3' -H 'content-type: application+json' -d '{"messageId": "390903e2-6e2a-4e3e-ba94-27038a720fce", "type": "-", "jsonData": { }}'  http://localhost:5000/streams/new-stream
```

```
HTTP/1.1 200 OK
Date: Mon, 17 Dec 2018 22:35:50 GMT
Content-Type: application/hal+json
Transfer-Encoding: chunked
Vary: Accept

{
  "currentVersion": 0,
  "currentPosition": 1000,
  "_links": {
    "streamStore:index": {
      "href": "../",
      "type": "application/hal+json",
      "title": "Index"
    },
    "streamStore:find": {
      "href": "../streams/{streamId}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Find a Stream"
    },
    "streamStore:feed-browser": {
      "href": "../streams{?p,t,m}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Browse Streams"
    },
    "streamStore:feed": {
      "href": "../streams/new-stream",
      "type": "application/hal+json"
    },
    "self": {
      "href": "../streams/new-stream",
      "type": "application/hal+json"
    },
    "curies": {
      "href": "../docs/{rel}",
      "templated": true,
      "type": "text/markdown",
      "name": "streamStore",
      "title": "Documentation",
      "hreflang": "en"
    }
  }
}
```

With multiple messages: 

```
curl -i -H 'sss-expected-version: -3' -H 'content-type: application+json' -d '[{"messageId": "5dce5675-a698-4a02-afc1-48ad88427f9b", "type": "-", "jsonData": { }}, {"messageId": "6da80336-671b-4b07-b166-cebe410b8178", "type": "-", "jsonData": { }}]'  http://localhost:5000/streams/new-stream-multiple
```

```
HTTP/1.1 200 OK
Date: Mon, 17 Dec 2018 22:35:50 GMT
Content-Type: application/hal+json
Transfer-Encoding: chunked
Vary: Accept

{
  "currentVersion": 1,
  "currentPosition": 1005,
  "_links": {
    "streamStore:index": {
      "href": "../",
      "type": "application/hal+json",
      "title": "Index"
    },
    "streamStore:find": {
      "href": "../streams/{streamId}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Find a Stream"
    },
    "streamStore:feed-browser": {
      "href": "../streams{?p,t,m}",
      "templated": true,
      "type": "application/hal+json",
      "title": "Browse Streams"
    },
    "streamStore:feed": {
      "href": "../streams/new-stream-multiple",
      "type": "application/hal+json"
    },
    "self": {
      "href": "../streams/new-stream-multiple",
      "type": "application/hal+json"
    },
    "curies": {
      "href": "../docs/{rel}",
      "templated": true,
      "type": "text/markdown",
      "name": "streamStore",
      "title": "Documentation",
      "hreflang": "en"
    }
  }
}
```

The difference between the two types of requests is that in the case of multiple messages, the message body is an array.

The `messageId` property _must_ be a `Guid`. Additionally, `type` and `jsonData` are required.

If the `SSS-ExpectedVersion` header is omitted, then `ExpectedVersion.Any` is assumed.

###Delete Stream

To delete a stream, simply send a `DELETE` request to the stream.

```
curl -i -X DELETE  http://localhost:5000/streams/new-stream
```

```
HTTP/1.1 204 No Content
Date: Mon, 17 Dec 2018 22:44:37 GMT
Vary: Accept
```

###Delete Stream Message

To delete a stream message, simply send a `DELETE` request to the stream message.

```
curl -i -X DELETE http://localhost:5000/streams/test-8d807c28fb3d4138b067ddab08bd99fd/0
```

```
HTTP/1.1 204 No Content
Date: Mon, 17 Dec 2018 22:46:10 GMT
Vary: Accept
```

Keep in mind that this is a dangerous operation, so great care must be used when using it. For example, if you have an aggregate stream with 5 messages in it, and you delete message 2, your aggregate may not functional properly upon re-hydration. Additionally, it may break HTTP caching for this stream _and_ the all stream.

###Set Stream Metadata

_TBD_