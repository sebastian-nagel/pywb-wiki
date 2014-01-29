*(Note: this is an evolving document as PyWb is under major development)*


The WARC file format supports records of type `warc/revisit` where the actual http content (payload) is stored in a different WARC record. These records indicate that duplicate content (but not http headers) was captured earlier, and therefore is not stored again in the WARC.

(ARC files do not support revisits and always follow Case 1 below)

This can save a lot of space when crawling, but adds a bit of complexity for replay.
When replaying such as records, it is necessary to load both the revisit record and the original payload record.

### Joining CDX Lines Optimization


Often times (but not always, see below on [Different Url Revisit Records](PyWb-Record-Lookup-and-Revisits#different-url-revisit-records)), the original record is an earlier capture of the same url.

Since the cdx server reads the urls in sorted order by timestamp, it is easy to track and 'resolve' the record against the previous capture by joining cdx lines with matching hash digest.
This prevents pywb from having to do a 2nd index lookup on the same url.

For example, given a cdx stream that has the following two entries:

```
com,example)/ 20130730210139 http://www.example.com/ text/html 200 B2LTWWPUOYAH7UIPQ7ZUPQ4VMBSVC36A - - 830 15152354 live-20130730183509913-02223-20130730214633985/live-20130730205901684-02244.arc.gz
...
com,example)/ 20140116020757 http://example.com/ warc/revisit - B2LTWWPUOYAH7UIPQ7ZUPQ4VMBSVC36A - - 524 271204751 top_domains-00500-20140115-204608/top_domains-00500-20140116015012-00006.warc.gz
```

to replay the capture at 20140116020757, the warc record at 20130730210139 will also need to be loaded.

The **resolve_revisits=true** flags allows the cdx server to join the two lines, as follows:

```
com,example)/ 20130730210139 http://www.example.com/ text/html 200 B2LTWWPUOYAH7UIPQ7ZUPQ4VMBSVC36A - - 830 15152354 live-20130730183509913-02223-20130730214633985/live-20130730205901684-02244.arc.gz - - -
...
com,example)/ 20140116020757 http://example.com/ text/html 200 B2LTWWPUOYAH7UIPQ7ZUPQ4VMBSVC36A - - 524 271204751 top_domains-00500-20140115-204608/top_domains-00500-20140116015012-00006.warc.gz 830 15152354 live-20130730183509913-02223-20130730214633985/live-20130730205901684-02244.arc.gz
```

The 3 entries extra fields, orig.offset, orig.length, orig.filename, of the original record are added to the regular 11-field CDX lines, to produce a final 14 field line.

In the case of non-revisit records, the last 3 fields are just blank, eg. '- - -'

Even though ARC files can not have revisit records themselves, it is possible for an ARC record to be referenced as the original record.

The implementation of **resolve_revisits=True** and other cdx operations can be found in [cdxserve.py][1]

## How pywb Resolves Records

pywb can consume the 14-field records as explained above, and performs the following checks.
It is designed to support the widest variety of warc revisit patterns.

(The below cases correspond to the logic listed in [replay.py][2])

**Case 2:**  Regular (ARC or WARC) record. The original record fields are an empty '-'. The http headers and payload are replayed from the same record.

**Case 3:** The original record fields are present (not a empty '-'), so this is a revisit record and a match for the original has been found with the same digest. The headers are replayed from the current filename (11-th) field, while the payload is replayed from the record specified by the original filename (14-th) field


### Different Url Revisit Records

So far, the assumption has been that the original capture is of the same url as the revisit.
However, with latest WARC standard, it is possible to have an original capture with different url and matching digest. For example a capture of `http://example.com` could be the same as that from `http://example.iana.org/`

With the latest WARC spec, it is possible to refer to a different url and timestamp for the original payload.

There are only two options: either there is a payload under a different url or the original is missing altogether.

**Case 1:** If the mimetype in the cdx is `warc/revisit`, then the join has not found a matching original with same url. It is necessary to look for `WARC-Refers-To-Target-URI` in the WARC record to see if it refers to the original. if the `WARC-Refers-To-Target-URI` is present, also load `WARC-Refers-To-Date` and attempts to perform a 2nd index lookup, filtering on the revisit digest to ensure the different lookup is an exact match. If successful, the payload record will be this other lookup and the headers will come from revisit record.

If `WARC-Refers-To-Target-URI` is missing, than the original payload is missing and the content can not be replayed.

### Other Special Cases 

Some other special cases involving revisits are as follows:

* It is possible that the `filename` field is blank, in which case the headers are served from same record as payload (This is a very old form of revisit)

* It is possible that a revisit record is empty (`Content-Length: 0`), and does not contain http headers. In such cases, the http headers from the payload record are used instead.


If either the payload or headers records could not be resolved, the replay will fail.

[1]: ../blob/master/pywb/cdxserve.py
[2]: ../blob/master/pywb/replay.py