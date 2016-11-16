## Distributed Archive Schema (Draft)

*Note: this implementation is a work in progress*

To enable distributed web archives spread across different services and platforms, a future release of pywb will support a flexible configuration that allows for connecting multiple archives and loading archived content (mementos) from the best available source. This configuration intentionally treats local and remote sources the same way, allowing for local CDX files to be read alongside queries to 'remote' Memento API and CDX Server endpoints.

To start, the following index sources will be supported.
Each source will have a unique prefix in the spec.

* Local CDXJ files
* Redis CDXJ interface (sorted list)
* Memento Timegate API
* CDX Server API


The following example demonstrates several access example configurations/schema.

### Local Archive

Simplest case of loading CDX from local directories,
WARCs from a local store and a back-up remote store.

```
collections:
    local:
        index: file:///local/index
        content:
             - file:///local/warcs
             - http://remote/path/to/warcs
```

### Memento-based and CDX Based Archive

Access to memento API and CDX server API based archives can be specified as follows
in its full form. The `memento` and `cdx` types indicate the type of access.

Since content is loaded directly from the specified replay url, the `content: $live` is used to indicate
the use of a live loader (as opposed to loading from a WARC file).

```
collections:
    aqpt:
       index:
          -
            type: memento
            timegate_url: http://arquivo.pt/wayback/{url}
            timemap_url: http://arquivo.pt/wayback/timemap/link/{url}
            replay_url: http://arquivo.pt/wayback/{timestamp}id_/{url}

       content: $live

    rhiz:
       index:
          -
            type: cdx
            api_url: http://webenact.rhizome.org/all-cdx?url={url}
            replay_url: http://webenact.rhizome.org/all/{timestamp}id_/{url}

       content: $live

```

The above schema is quite verbose and could be shortened to the equivalent:

```
collections:
    aqpt: memento+http://arquivo.pt/wayback/
    rhiz: cdx+http://webenact.rhizome.org/all-cdx
```

The `memento+` and `cdx+` schemes are now used to indicate the type of source and the source can be defined more clearly.
Given a prefix, it is often possible to determine the timegate, timemap and replay urls for memento sources.
For cdx sources, some assumptions are made, such as if the source ends in `-cdx` (followng pywb cdx conventions), the replay url can be obtained by dropping the `-cdx`. For more specificity, the replay collection can be specified for the cdx source:

```
collections:
    ia: cdx:web+http://web.archive.org/cdx
```

would be equivalent to:

```
collections:
    ia:
       index:
          - 
            type: cdx
            api_url: http://web.archive.org/cdx?url={url}
            replay_url: http://web.archive.org/web/{timestamp}id_/{url}
```

(The `content: $live` loader can be assumed to be implicit and need not be specified)

### Aggregate Loaders

The main benefit of such a definition would be to allow more complex loaders to be defined:

```
collections:
   many:
       index:
           ia: cdx:web+http://web.archive.org/cdx
           aqpt: memento+http://arquivo.pt/wayback/
           rhiz: cdx+http://webenact.rhizome.org/all-cdx
           local: ./local/index
       
       content:
           - $live
           - ./local/warcs

       index_timeout: 5.0

```

The above schema would specify that urls should be looked up in 4 sources, 2 cdx-server based, 1 memento based,
and 1 local source. Each source will have a timeout of 3.0 secs so that if a source does not return then, it is ignored.
When loading content, it may be loaded from either the live web or from a local WARCs directory.

### Sequence Fallback

The above demonstrates how to load from multiple archives simultaneously. In some cases, it may be beneficial to first check a local source, and then fall back to a remote source, if the local source does not have the requested memento.


```
collections:
   many:
       sequence:
         - 
           index: ./local/index
           content:
                  ./local/warcs
         -
           index:
               ia: cdx:web+http://web.archive.org/cdx
               aqpt: memento+http://arquivo.pt/wayback/
               rhiz: cdx+http://webenact.rhizome.org/all-cdx
               local: ./local/index
       
           index_timeout: 5.0

```
