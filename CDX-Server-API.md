In addition to replay capabilities, pywb also provides an extensive api for querying the capture index (CDX).

The api can be used to get information about a range of archive captures/mementos, including filtering, sorting, and pagination for bulk query. The actual archive files (WARC/ARC) files are not loaded during this query, only
the generated CDX index.

For example, the following query might return the first 10 results from host http://example.com/* where
the mime type is text/html:

`http://localhost:8080/coll-cdx?url=http://example.com/*&page=1&filter=mime:text/html&limit=10`

## Usage and Configuration

The `cdx-server` command line application starts pywb in cdx server only mode (web archive replay functionality is not loaded, only the index). It can be used the same way as the `wayback` command line application, including the auto-configuration init.

The cdx server functionality can also be enabled when running full-replay with `wayback` by setting:

```
enable_cdx_api: true
```

### API Endpoint

When `enable_cdx_api` is set to true, a cdx server endpoint is created for each collection and accessible by adding `-cdx` to the regular collection path, ex:

- `http://localhost:8080/pywb/<timestamp>/<url>` - web archive replay

- `http://localhost:8080/pywb-cdx` - cdx server endpoint

- `http://localhost:8080/other_collection/<timestamp>/<url>` - web archive replay

- `http://localhost:8080/other_collection-cdx` - cdx server endpoint


The suffix can additionally be customized through the `enable_cdx_api` setting

`enable_cdx_api: -index`

This will make the endpoint be `http://localhost:8080/pywb-index` instead


# API Reference
