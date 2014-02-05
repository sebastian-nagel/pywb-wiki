Here's a brief overview of the various components of `pywb` to date.


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



