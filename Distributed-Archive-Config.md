# Distributed Archive Schema (Draft)

*Note: this implementation is a work in progress*

To enable distributed web archives spread across different services and platforms, a future release of pywb will support a flexible configuration that allows for connecting multiple archives and loading archived resources (mementos) from the best available source. This configuration intentionally treats local and remote sources the same way, allowing for local CDX files to be read alongside queries to 'remote' Memento API and CDX Server endpoints. The spec will cover both index lookup and resource loading. The spec allows for declaring collections, where each collection can consist of multiple index and resource declarations.

## Index Spec

To start, the following index sources will be supported.
Each source will have a unique prefix in the spec.

* Local CDXJ files
* Redis CDXJ interface (sorted list)
* Memento Timegate API
* CDX Server API

The following example demonstrates several access example configurations/schema.
When possible, a full schema is used along with a 'shorthand' version that simplifies configuration
and follows certain conventions. A collection is declared to consist of one more index and resource declarations.

### Local Archive

Simplest case of loading CDX from local directories or files,
WARCs from either a local store and a back-up remote store could be specified as follows
with a full schema:

```
collections:
    local:
        index:
            -
              type: file
              path: file:///local/index/

        resource:
              warc:
                 - file:///local/warcs
                 - http://remote/path/to/warcs
```

A more simplified schema of the above can be written as:

```
collections:
    local:
        index: /local/index
        resource:
             - /local/warcs
             - http://remote/path/to/warcs
```

### Memento API Archive

A memento-API based archive can be specified as follows in the full schema:

```
collections:
    aqpt:
       index:
          -
            type: memento
            timegate_url: http://arquivo.pt/wayback/{url}
            timemap_url: http://arquivo.pt/wayback/timemap/link/{url}
            replay_url: http://arquivo.pt/wayback/{timestamp}id_/{url}

       resource: $live
```

In addition to the timegate and timemap urls, a replay url is specified for accessing the 'raw' unaltered memento.
The schema urls can include special variables `{url}` and `{timestamp}` which are interpolated to retrieve the final urls to lookup.

Since the resource is loaded directly from the specified replay url, the `resource: $live` is used to indicate
the use of a live loader (as opposed to loading from a WARC file) in the full schema.

If the above conventions are observed, it is possible to derive the above schema from a simplified config:

```
collections:
    aqpt: memento+http://arquivo.pt/wayback/
```

The special `memento+` prefix is used to indicate a memento source, and the timegate, timemap and replay urls
are derived from the prefix. The `resource` live loader is assumed as well.

### CDX API Archive

Loading from a CDX API based archive can be specified in a similar way:

```
collections:
    rhiz:
       index:
          -
            type: cdx
            api_url: http://webenact.rhizome.org/all-cdx?url={url}&closest={timestamp}
            replay_url: http://webenact.rhizome.org/all/{timestamp}id_/{url}

       resource: $live

```

The above full schema could be specified to a shorter config:

```
collections:
    rhiz: cdx+http://webenact.rhizome.org/all-cdx
```

#### Replay Prefix

Since the CDX API does not alone provide a way to determine the replay url, some assumptions could be made.
In this example, if the cdx url ends with `-cdx` (followng pywb cdx conventions), the replay url can be obtained by dropping the `-cdx`.

When the replay url is a different, such as:

```
collections:
    ia:
       index:
          -
            type: cdx
            api_url: http://web.archive.org/cdx?url={url}&closest={timestamp}
            replay_url: http://web.archive.org/web/{timestamp}id_/{url}

       resource: $live

```

The replay url prefix could be specified as a space-delimited prefix:

```
collections:
    ia: cdx+http://web.archive.org/cdx /web/
```

### Live Index

It is also possible to specify a 'live' collection, where the lookup is simply the live web. This can be done as follows:

```
collections:
   live:
      index:
        -
          type: live
```

or simply:

```
collections:
    live: $live
```

#### Replay from WARC/ARC

The above assumes resource is loaded directly from the live web, not from a WARC/ARC file.
It would also be possible to configure a remote CDX API index, where the resource is loaded from a WARC.

```
collections:
   remote-cdx:
      index: cdx+http://path/to/cdx/server
      resource: http://path/to/warc/storage

```

In such an example, the WARC loader is used instead of the live web loader. (See Loaders section for more details).


### Aggregate Loaders

The main benefit of such a definition would be to allow more complex loaders to be defined:

```
collections:
   many:
       index:
           ia: cdx+http://web.archive.org/cdx /web
           aqpt: memento+http://arquivo.pt/wayback/
           rhiz: cdx+http://webenact.rhizome.org/all-cdx
           local: ./local/index
       
       resource:
           - $live
           - ./local/warcs

       index_timeout: 5.0

```

The above schema would specify that urls should be looked up in 4 sources, 2 cdx-server based, 1 memento based,
and 1 local source. Each source will have a timeout of 3.0 secs so that if a source does not return then, it is ignored.
When loading the resource, it may be loaded from either the live web or from a local WARCs directory.

### Sequential Loading

In some cases, it may be beneficial to first check a local source, and then fall back to one or more remote sources, if the local source is not available or does not contain the requested memento url. There could be a final fallback to loading a live resource. This can be done with a setup as follows:

```
collections:
   seq_fallback:
       sequence:
         - 
           index: ./local/index/
           resource:
               ./local/warcs
         -
           index:
               ia: cdx+http://web.archive.org/cdx /web
               aqpt: memento+http://arquivo.pt/wayback/
               rhiz: cdx+http://webenact.rhizome.org/all-cdx
       
           index_timeout: 3.0

         - index: live

```

In this example, there is first a local CDX lookup for files located in `./local/index`.
If the lookup fails, 3 remote sources (2 cdx, 1 memento) are queried. If they return a successful match within 3.0 seconds, the lookup succeeds. If this second lookup fails, the live lookup is performed (which always succeeds).

With the `sequence` setup, it should be possible to specify as many lookups as needed, and set timeout values for each step of the way.

### Index API

The API for looking up the index is the familiar CDX Server API, which produces a CDXJ/JSON output. The output contains a line for each matched result. The data in the result, however, differs based on the type of query that was found.

For example, the following query might result in the following response (timestamps shorted to illustrate the format) after querying local and remote sources. A collection named `many` is configured (as shown above) to query several archives:

```
=> GET /many/index?url=http://example.com&output=json&closest=2016
<=
{... "timestamp": "2016", "live_url": "http://webenact.rhizome.org/all/2016id_/http://example.com/", "source": "rhiz", "source_type": "cdx"},
{... "timestamp": "2015", "live_url": "http://web.archive.org/web/2015id_/http://example.com/", "source": "ia", "source_type" "cdx"},
{... "timestamp": "2014", "filename": "warc-name.gz", "offset": "0", "length": "1000", "source": "local", "source_type": "file"},
{... "timestamp": "2013", "live_url": "http://arquivo.pt/wayback/2013id_/http://example.com/", "source": "aqpt", "source_type": "memento"}
```

In this example, 4 archives were queried and produced one match for each url, sorted closest to 2016 (again, timestamps shortened for purposes of this example).

The response lines contains a `source` field to indicate the name of the source as declared in the schema, and a `source_type`, indicating the type of source (optional?). In addition, the JSON response may contain many other fields that are available.

Two response formats are possible based on the type of source.
If the response JSON contains a "live_url" field, this indicates that the requested resource can be loaded directly from this url. This is the case for CDX server and Memento based sources.

For the local response, a field named "filename" field along with "offset" and "length" are included (based on standard CDX spec). This type of response does not have an absolute path to the WARC file and requires a `resource` loader to be configured with one or more filename resolvers.

## Resource Loader

The `resource` section of each collection declaration defines what sort of resource loaders are used when using the Resource API (It is not needed for the Index API).

If omitted, `resource: $live` is assumed, which activates the live web loader. This loader is sufficient for loading any results that have a full live web url, eg. where the `live_url` field is provided.

However, it is not sufficient for loading WARC resources from CDX, as these do not contain absolute paths.
For that, a resource declaration should include paths for resolving the WARC paths:

```
       resource:
           - $live
           - ./local/warcs
           - http://remote/warc/path/
```

Given the responses above, any response line with `live_url` will be handled by the `$live` loader.
Any response line that contains a `filename`/`offset`/`length` combo will be handled by the WARC loader. eg.
`warc-name.gz` will be looked up in `./local/warcs/warc-name.gz`, then in `http://remote/warc/path/warc-name.gz`.

To remove ambiguity, the `resource` section could be defined with special `warc-prefix+` and `warc-index+` prefixes
to indicate the the urls represent either path prefixes, or indexes to lookup filename to absolute path.

```
       resource:
           - $live

           # warc prefix to find exact WARC path

           - warc-prefix+file:///local/warcs
           - warc-prefix+http://remote/warc/path/

           # text-based filename lookup
           - warc-index+file:///warc/path-index.txt

           # redis-based filename lookup
           - warc-index+redis://redis/hash_key
```

If omitted, the `warc-prefix` or `warc-index` is determined automatically based on the url. This will allow for adding other lookup methods and non-WARC lookup in the future.

## Resource API

The Resource API uses the Index API and the Resource Loader to return a valid WARC record of the requested resource.

As such, the Resource API is identical to the Index API and differs only in the response, which is either a valid WARC record (200) or a 404 Not Found.

If the index API query is:
```
=> GET /many/index?url=http://example.com&output=json&closest=2016
<=
{... "timestamp": "2016", "live_url": "http://webenact.rhizome.org/all/2016id_/http://example.com/", "source": "rhiz", "source_type": "cdx"},
{... "timestamp": "2015", "live_url": "http://web.archive.org/web/2015id_/http://example.com/", "source": "ia", "source_type" "cdx"},
{... "timestamp": "2014", "filename": "warc-name.gz", "offset": "0", "length": "1000", "source": "local", "source_type": "file"},
{... "timestamp": "2013", "live_url": "http://arquivo.pt/wayback/2013id_/http://example.com/", "source": "aqpt", "source_type": "memento"}
```

the resource API query would be:
```
=> GET /many/resource?url=http://example.com&output=json&closest=2016
...
<= HTTP/1.0
Content-Type: application/warc-record
Archive-Source-Coll: rhiz
Memento-Datetime: ...


...
WARC/1.0
WARC-Date: 2016
WARC-Target-URI: http://example.com/
Link: <http://example.com>; rel="original"
...
```

The response format includes the WARC record as the response body.
The headers for the response include additional metadata, such as `Archive-Source-Coll` which indicates the source of the archive that this response is coming from, and is just the `source` field in the index response line. This allows the user to know exactly which archive returned this response.

The headers also include standard URL-M memento headers to indicate that this WARC response is a memento (without having to parse the WARC record even). This should allow the Resource API endpoint to also serve as a valid memento endpoint (TODO).

### Resource Loading Logic

Each response line from the Index API is attempted, until there is a successful resource load. If the resource is a live web resource, it is converted on-the-fly to a WARC response.
If it is already a WARC response, it is returned as is.
(If it is an ARC record, it is converted to a WARC record on the fly as well)

This logic can be explained with roughly the following pseudo-code:

```
for each response from Index API
   try
     if liveweb loader:
       load from live web
       convert to WARC response
       return WARC response

     if WARC loader:
       resolve absolute WARC path
       return WARC response
   catch:
      continue to next line

return 404 Not Found, error info

```

### Live Web Resources

This approach can also be used to load live web resources and wrap them in a WARC record.
For example, if a collection is defined as a live web collection:

```
collections:
   live: $live
```

The the resource API request to:
```
=> GET /live/resource?url=http://example.com/
<= application/warc-record
```

results in a live request to `http://example.com/` and the response from the server is wrapped in a WARC response record and returned.

### Other HTTP Methods

The resource API is not limited GET methods, but can support any other method as well.
For example,

```
=> POST /live/resource?url=http://httpbin.org/post`
...
<post data>

<= application/warc-record
```

should perform a POST to the live resource. POST data can also be looked up in a local CDX index which supports POST request resolution.

### Resource API via POST

However, there are limitations on supporting different http methods in this way. It would not be possible to use a HEAD request and retrieve a WARC record of the response. Additionally, HTTP headers sent to the Resource API may not be the ones that should be sent to the archival or live server.

For this reason, the Resource API also supports an all-POST interface. In this mode, the HTTP request that is sent to the remote server (or used for lookup) is the POST data.

For example, requesting http://example.com/ might be done with:
```
=> POST /live/resource-post?url=http://example.com/
...
GET /
User-Agent: ...
...
<= application/warc-record
```

It should even be possible to record a HEAD request and other requests in this way:

```
=> POST /live/resource-post?url=http://example.com/
...
HEAD /
User-Agent: ...
...
<= application/warc-record
```

The response is always a valid WARC record, which could then be used for replay, recorded, etc...



