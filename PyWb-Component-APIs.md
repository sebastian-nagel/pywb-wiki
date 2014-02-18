PyWb consists of several distinct subpackages, each of which may be useful on their own.

The following is an overview of the APIs expose by each package.

#### pywb.cdx

The CDX provides access to reading the cdx index from a variety of sources via a CDX server.
The CDX Server supports the following api:

`CDXServer.load_cdx(**params)`

The params dictionary contains a parameters for the cdx query
(currently a subset of CDX API listed here:
https://github.com/internetarchive/wayback/tree/master/wayback-cdx-server)

The function returns an iterator of cdx objects, or plain text cdx stream.
The [cdx object][1] is a dictionary containing available cdx fields.


#### pywb.warc



[1]: ../blob/master/pywb/cdx/cdxobject.py