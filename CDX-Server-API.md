In addition to replay capabilities, pywb also provides an extensive api for querying the capture index (CDX).

The api can be used to get information about a range of archive captures/mementos, including filtering, sorting, and pagination for bulk query. The actual archive files (WARC/ARC) files are not loaded during this query, only
the generated CDX index. pywb actually uses this same api internally to perform all index lookups in a consistent way.

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

The following is a list of available query params, as of pywb 0.9.1

### `url`

The only required parameter to the cdx server api is the url, ex:
`http://localhost:8080/coll-cdx?url=example.com`

will return a list of captures for 'example.com'


### `matchType`

The cdx server supports the following `matchType`

- `exact` -- default setting, will return captures that match the url exactly

- `prefix` -- return captures that begin with a specified path, eg: `http://example.com/path/*`

- `host` -- return captures which for a begin host (the path segment is ignored if specified)

- `domain` -- return captures for the current host and all subdomains, eg. `*.example.com`

As a shortcut, instead of specifying a separate `matchType` parameter, wildcards may be used in the url:

- `...coll-cdx?url=http://example.com/path/*` is equivalent to `...coll-cdx?url=http://example.com/path/&matchType=prefix`

- `...coll-cdx?url=*.example.com` is equivalent to `...coll-cdx?url=example.com&matchType=domain`

*Note: if you are using legacy cdx index files which are not SURT-ordered, the `domain` option will not be available. if this is the case, you can use the `wb-manager convert-cdx` option to easily convert any cdx to latest format`*

### `limit`

Setting `limit=` will limit the number of index lines returned. Limit must be set to a positive integer. If no limit is provided, all the matching lines are returned, which may be slow. (If using a ZipNum compressed cluster, the page size limit is enforced and no captures are read beyond the single page. See Pagination API for more info).

### `sort`

The `sort` param can be set as follows:

- `reverse` -- will sort the matching captures in reverse order. It is only recommended for `exact` query as reverse a large match may be very slow. (An optimized version is planned)

- `closest` -- setting this option also requires setting `closest=<ts>` where `<ts>` is a specific timestamp to sort by. This option will only work correctly for `exact` query and is useful for sorting captures based no time distance from a certain timestamp. (pywb uses this option internally for replay in order to fallback to 'next closest' capture if one fails)

Both options may be combined with `limit` to return the top N closest, or the last N results.

### `output` (JSON output)

Setting `output=json` will return each line as a proper JSON dictionary. (Default format is `text` which will return the native format of the underlying CDX index, and may not be consistent). Using `output=json` is recommended for extensive analysis and it may become the default option in a future release.

### `filter`

The `filter` param can be specified multiple times to filter by specific fields in the cdx index. Field names correspond to the fields returned in the JSON output. Filters can be specified as follows:

```...coll-cdx?url=example.com/*&filter=mime:text/html&!status:200```

Return captures from example.com/* where mime is text/html and http status is not 200.

The `!` modified before `status` indicates negation. The `~` modifier can also be used to specify a regex
instead of exact filter match. For example: `filter=~mime:text/.*` will match any CDX line where `mime` field
matches the regex `text/.*`. Negation and regex modifier may be combined, eg. `filter=!~text/.*`

