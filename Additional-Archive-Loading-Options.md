## Remote HTTP Archive Paths

In addition to locally stored WARC/ARC files, pywb supports loading archives over HTTP/HTTPS.
(The HTTP server must support HTTP/1.1 Range requests to server portions of a WARC/ARC files)

The simplest way is to specify an HTTP/HTTPS prefix:

`archive_paths: http://example.com/warcs/`

Then, if the collection index (CDX) contains a reference to `mywarc1.warc.gz`, pywb will attempt to load

`http://example.com/warcs/mywarc1.warc.gz`

### S3 Paths

pywb can load archives from Amazon S3 as long as they are publicly accessible. S3 Auth is not yet supported.

To load WARCs from S3, you can specify the HTTP equivalent for your bucket.

For instance, to load from `s3://my_sample_bucket/warcs/`, the entry could be:

`archive_paths: https://my_sample_bucket.s3.amazonaws.com/`

This should work as long as the bucket is publicly accessible.


## Path Index File

For more complex use cases, a special 'path index' file may be used to specify a path per warc

`archive_paths: ./path-index.txt`

Where path-index might contain:

```
mywarc1.warc.gz<TAB>http://example.com/warcs/mywarc1.warc.gz<TAB>http://backup.example.com/warcs/mywarc1.warc.gz
mywarc2.warc.gz<TAB>http://example.com/warcs2/mywarc2.warc.gz
...
```

The path index file:
 * is TAB delimited
 * is Sorted by WARC filename (matching exactly the entries in the CDX)
 * May contain one more full paths to the WARC to be resolved. If there are multiple entries, they will be tried in succession until one succeeds.
 * May contain a mix of any other paths, eg. both http and local file paths may be used.




