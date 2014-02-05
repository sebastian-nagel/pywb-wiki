The following is a brief overview of different components of pywb to date.

### Config Loading/Bootstrap

pywb contains a pluggable module for initializes the WSGI app.
**pywb_init.py** contains the default configuration which initializes pywb from a YAML file.
By changing environment var `PYWB_CONFIG_MODULE`, its possible to switch to a different module.

The init modules `pywb_config()` is called on startup to create a routing object for pywb.

### WSGI Wrapper

**wbapp.py** contains the entry point for the WSGI application. The application callback is created using the config passed from `pywb_config()`. The application callback passes the WSGI request env to the router and receives a *WbResponse* which is then returned to WSGI caller.

Any exceptions from the app are caught and rendered via an error page, if available, or via default text error message.

### Routing

pywb currently supports *ArchivalRequestRouter* which contains a serious of *Route* objects that map incoming request to a handler, eg. `Route('pywb', handler)` will route requests from /pywb to handler.
The router also parses the WSGI env and creates an appropriate WbRequest that is sent to the handler.
Included in the WbRequest is a standard url rewriter (*UrlRewriter*) and wayback url format *WbUrl* to use.

These settings can be modified to support alternative url replay models or different routing.

A proxy-based router will soon be added which will route HTTP proxy request to pywb and replay content without url rewriting.

### Index Reading

Before data can be read from an archive, it is necessary to look up the source in an index.
The index contains a mapping from url to archived content file and offset/length.

pywb supports the CDX index format, and contains a *LocalCDXServer* component for searching and filtering cdx files.

This component is more general than requirements of pywb and will probably be moved to a subpackage to allow standalone use as a WSGI app.

pywb can also connect via HTTP to a *RemoteCDXServer* which returns a stream of filtered cdx response for a given query.

This allows for pywb to run in a WSGI container without a local index, provided the index is accessible via a cdx server.

### ARC/WARC Reading

pywb supports reading compressed and uncompressed .warc and .arc files, from either local file system (FileReader) or via HTTP 1.1 Range requests (HttpReader).
pywb reads and parses warc and arc headers, and maintains a stream of the record for further processing or streaming back via WSGI.

#### Archive File Resolvers

After obtaining filename, offset, length info from the cdx index, pywb will attempt to locate the specified archive file using a resolver. The resolver converts the relative filename from the index to the absolute filename where pywb may find the warc/arc. The simplest (default) resolve is the `PrefixResolver` where a local directory or http path is prepended to the filename. Other resolvers include a `PathIndexResolver` where the file is looked up in a local index, and `RedisResolver` where the filename is looked up as a key in a Redis key-store cache.

### Views/UI

pywb supports several views that render responses to the user.
For non-archival content, Jinja2 template system is used to render html for query page.
The template .html ca be configured in the YAML config or disabled altogether.

For archival content, pywb supports either transparent replay which streams archival content unaltered, or a pluggable *RewritingReplayView* which supports url rewriting.

### Url Rewriting

However, the main reason for pywb existence is to facilitate proper rewriting of archived content so that it may be rendered from a different url then it was originally captured.

As such, pywb comes with a standard *HeaderRewriter*, *HTMLRewriter*, *CSSRewriter*, *JSRewriter* and *XMLRewriter*.

The html rewriter parses html and replaces specific instances of urls with their rewritten versions, delegating to the CSS/JS/XML rewriters to handle domain specific content. Content is rewritten based on the `Content-Type` stored in the archived record.

The CSS/JS/XML rewriters are currently regex based.

#### HTML Inserts

The *RewritingReplayView* also supports a custom html template which is inserted into the `<head>` element of a page. The insert may generated a banner or trigger other client side rewriting.

#### Other processing

The html rewriter also rewriters headers via the `HeaderRewriter` to ensure redirects are properly handled.
Gzipped and chunked content is normalized, and `chardet` library is used to determine charset before parsing.

#### Customization/Domain Specific Rewriting

To properly render difficult/dynamic sites, rewriting may need to be customized on a domain specific level.
The HTML, CSS, JS, XML can be replaced in a custom rewriting view.

---

If successful, the rewritten response is returned via a `WbResponse` object back up the chain to the WSGI callback. Any errors are reported to the user.
