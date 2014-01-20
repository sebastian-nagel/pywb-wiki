*(Note: this is an evolving document as PyWb is under major development)*


The WARC file format supports records of type `warc/revisit` where the actual http content (payload) is stored in a different WARC record. These records indicate that duplicate content (but not http headers) was captured earlier, and therefore is not stored again in the WARC.

(ARC files do not support revisits and always follow Case 1 below)

This can save a lot of space when crawling, but adds a bit of complexity for replay.
When replaying such as records, it is necessary to load both the revisit record and the original payload record.

pywb currently relies on [wayback-cdx-server][1] semantics for resolving revisit records.

Joining CDX Lines Optimization
------------------------------

Often times (but not always, see below on [Different Url Revisit Records](.#different-url-revisit-records)), the original record is an earlier capture of the same url.

Since the cdx server reads the urls in sorted order by timestamp, it is easy to track and 'resolve' the record against the previous capture by joining cdx lines with matching hash digest.
This prevents pywb from having to do a 2nd index lookup on the same url.

For example, given a cdx stream that has the following two entries:

```
com,example)/ 20130730210139 http://www.example.com/ text/html 200 B2LTWWPUOYAH7UIPQ7ZUPQ4VMBSVC36A - - 830 15152354 live-20130730183509913-02223-20130730214633985/live-20130730205901684-02244.arc.gz
...
com,example)/ 20140116020757 http://example.com/ warc/revisit - B2LTWWPUOYAH7UIPQ7ZUPQ4VMBSVC36A - - 524 271204751 top_domains-00500-20140115-204608/top_domains-00500-20140116015012-00006.warc.gz
```

to replay the capture at 20140116020757, the warc record at 20130730210139 will also need to be loaded.

The `resolveRevisits=true` flags allows the cdx server to join the two lines, as follows:

```
com,example)/ 20140116020757 http://example.com/ text/html 200 B2LTWWPUOYAH7UIPQ7ZUPQ4VMBSVC36A - - 524 271204751 top_domains-00500-20140115-204608/top_domains-00500-20140116015012-00006.warc.gz 524 271204751 top_domains-00500-20140115-204608/top_domains-00500-20140116015012-00006.warc.gz
```

The 3 entries `(orig.offset, orig.length, orig.filename)` of the original are added to the regular 11-field CDX lines, to produce a merged 14 field line.

How PyWb Resolves Records
-------------------------

pywb can consume the 14-field records as explained above, and performs the following checks.
It is designed to support the widest variety of warc revisit patterns.
(This can be seen in [replay.py][2])


**Case 1:** The `orig.filename` is absent. Regular non-revisit record, the http headers and payload are replaying from the same WARC record.

**Case 2:** The `orig.filename` is present by the regular filename is a `-`. This indicates an older, obsolete form of revisit, where the filename is omitted in the CDX. For such records, it is necessary to replay the WARC headers and payload from the `orig.filename`

**Case 3:** The `filename` and `orig.filename` are present. The revisit record has been 'resolved' (that is, an original capture has been found for the revisit digest) as per the above join. Load both warc records.
Replay http headers from the `filename` warc and payload from `orig.filename`

**Case 4:** Special case of 3. The warc/revisit record has no http headers (`Content-Length: 0`), replay http headers and payload from the original record. While the headers should have been included in the revisit record by the crawler, there are instances of empty revisit records.

Different Url Revisit Records
-----------------------------

So far, the assumption has been that the original capture is of the same url as the revisit.
However, with latest WARC standard, it is possible to have an original capture with different url and matching digest. For example a capture of `http://example.com` could be the same as that from `http://example.iana.org/`

With the latest WARC spec, it is possible to refer to a different url and timestamp for the original payload.
There are only two options: either there is a payload under a different url or the original is missing altogether.

**Case 5:** If the mimetype in the cdx is `warc/revisit`, then the join has not found a matching original with same url. It is necessary to look for `WARC-Refers-To-Target-URI` in the WARC record to see if it refers to the original. If it doesn't, not much more can be done as the payload is missing.

**Case 6:** if the `WARC-Refers-To-Target-URI` is present, also load `WARC-Refers-To-Date` and attempts to perform a 2nd index lookup, filtering on the revisit digest to ensure the different lookup is an exact match. If successful, the payload record will be this other lookup and the headers will come from revisit record.


[1]: https://github.com/internetarchive/wayback/tree/master/wayback-cdx-server
[2]: https://github.com/ikreymer/pywb/blob/master/pywb/replay.py