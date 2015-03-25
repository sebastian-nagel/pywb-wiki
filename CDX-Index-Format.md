The main index format used by pywb is a plain text format, originating with the Internet Archive. Several extensions have been made to the format, including the latest CDXJ for pywb 0.9.0. The ZipNum Cluster format is a compressed version of cdx, with an extra secondary index. It is possible to support other index formats, CDX remains the default format used in pywb.

## CDX File Format

An index for a web archive (WARC or ARC) file is often referred to as a CDX file, probably from Capture/Crawl inDeX (CDX). A CDX file is typically a sorted plain-text file (optionally gzip-compressed) format, with each line representing info about a single capture in an archive. The CDX contains multiple fields, typically the url and where to find the archived contents of that url. Unfortunately, no standardized format for CDX files exists, and there have been many formats, usually with varying number of space-seperated fields. Here is an old reference for CDX File (from Internet Archive). In practice, CDX files typically contain a subset of the possible fields.

While there are no required fields, in practice, the following 6 fields are needed to identify a record: url search key, url timestamp, original url, archive file, archive offset, archive length. The search key is often the url transformed and 'canonicalized' in a way to make it easier for lexigraphic seaching. A common transformation is to reverse subdomains example.com -> com,example,)/ to allow for searching by domain, then subdomains.

The indexing job uses the flexible pywb cdx-indexer to create indexs of a certain format. However, the other jobs are compatible with any existing CDX format as well. Other indexing tools can be used also but require seperate integration.

## ZipNum Sharded CDX

A CDX file is generally accessed by doing a simple binary search through the file. This scales well to very large (multi-gigabyte) CDX files. However, for very large archives (many terabytes or petabytes), binary search across a single file has its limits.

A more scalable alternative to a single CDX file is gzip compressed chunked cluster, with a binary searchable index. In this format, sometimes called the ZipNum or Ziplines cluster (for some X number of cdx lines zipped together), all actual CDX lines are gzipped compressed an concatenated together. To allow for random access, the lines are gzipped in groups of X lines (often 3000, but can be anything). This allows for the full index to be spread over N number of gzipped files, but has the overhead of requiring N lines to be read for each lookup. Generally, this overhead is negligible when looking up large indexes, and non-existent when doing a range query across many CDX lines.

The index can be split into an arbitrary number of shards, each containing a certain range of the url space. This allows the index to be created in parallel using MapReduce with a reduce task per shard. For each shard, there is an index file and a secondary index file. At the end, the secondary index is concatenated to form the final, binary searchable index. 

The [webarchive-indexing](https://github.com/ikreymer/webarchive-indexing) project provides tools for creating such an index, both locally and via MapReduce.

### Single-Shard Index

A ZipNum index need not have multiple shards, and provides advantages even for smaller datasets. 
For example, in addition to less disk space from using compressed index, using the ZipNum index allows for the [Pagination API]() to be available when using the cdx server for bulk querying.