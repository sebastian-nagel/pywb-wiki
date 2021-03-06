PyWb consists of several distinct subpackages, each of which may be useful on their own.

The following is an overview of the APIs expose by each package.

#### pywb.cdx

This package provides access to reading the cdx index from a variety of sources via a CDX server.
The CDX Server supports the following api:

`CDXServer.load_cdx(self, **params)`

The params dictionary contains a parameters for the cdx query
(currently a subset of CDX API listed here:
https://github.com/internetarchive/wayback/tree/master/wayback-cdx-server)

The function returns an iterator of cdx objects, or plain text cdx stream.
The [cdx object][1] is a dictionary containing available cdx fields.


#### pywb.warc

This package provides facilities for loading archived content from ARC and WARC files.
First, It provides a low level api for loading one record at a time, given a full URI and an offset and length.

`ArcWarcRecordLoader.load(self, uri, offset, length)`

The URI must be a full local, file:// or http:// identifier.
The load() functions returns a WARC/ARC record header header and a read stream to continue reading.

This package also provides a higher level API for interpreting a cdx line and returning the archived HTTP content.

[resolvingloader.py][2] provides the following interface:

`ResolvingLoader.resolve_headers_and_payload(self, cdx, failed_files)`

This function takes a cdx object from the cdx server (and an optional `failed_files` list to store failed attempts) and returns an http headers object + content stream pair, resolving any revisit records, as explained in [PyWb-Record-Lookup-and-Revisits](PyWb-Record-Lookup-and-Revisits).

ResolvingLoader is initialized with an instance of `CDXServer` to allow for additional cdx lookups necessary for revisit lookup.


#### pywb.rewrite

The rewrite package deals with content rewriting necessary to replay archival content.
It provides rewriters for HTML, CSS, JS, XML and http headers based on custom rules and a page specific url rewriter. The url rewriter is used to convert urls to be relative to a given archived page.

Currently, it provides the following main interface via [rewrite_content.py][3]

`RewriteContent.rewrite_content(self, urlrewriter, headers, stream, head_insert_str = None):`

The rewrite_content() function rewrites the http headers and consumes the input stream.
An optional string to be inserted into the `<head>` for html content may be specified.

The function returns a pair of rewritten headers and an iterator of rewritten content.
The headers and iterator are ready to be sent as part of a WSGI response.


[1]: ../blob/master/pywb/cdx/cdxobject.py
[2]: ../blob/master/pywb/warc/resolvingloader.py
[3]: ../blob/master/pywb/rewrite/rewrite_content.py