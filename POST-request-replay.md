Proper replay of HTTP POST requests has been problematic in wayback and pywb. While initially primarily used for user form submissions, more and more sites, especially social media sites, image slideshows, etc.. use POST as part of the normal operation.

The main issue with POST requests is in the lookup phase -- the POST body is usually lost during the CDX indexing process (which uses the request URI only as the index key), making it impossible to match a POST request to a proper response.

Starting with version 0.4.5, pywb improved POST request replay for all *application/x-www-form-urlencoded* POST requests will now be possible. The main approach is to match adjacent WARC *response* (or *revisit*) and *request* records and for any POST request, extract the POST data and include it as part of the key in the CDX index.

This ``-p`` option in ``cdx-indexer`` will enable this functionality.

In this way, the POST data will be treated no differently than a GET query.

For example, a POST request:

```
WARC/1.0
WARC-Type: request
WARC-Target-Uri: http://example.com/path/post?a=b&goo=baz
...

POST /path/post?a=b&goo=baz HTTP/1.0
...
Content-Type: application/x-www-form-urlencoded
...
foo=bar

```

will be indexed with the key ``http://example.com/path/post?a=b&goo=baz`` + ``foo=bar``, which may be canonicalized to: ``com,example)/path/a=b&foo=bar&goo=baz`` as the CDX key.
(The original url field in the cdx be unchanged).

This will allow pywb to handle incoming POST requests, read the POST body and search for the url + post data as a single key.

Fuzzy matching rules can then be applied on the final url, including POST data, and may often be needed to map POST requests to actual archived data.

pywb will also not issue redirects for POST requests, as 302 will lose the POST data, and 307 (the correct status to use) will result in a modal dialog in certain browsers (eg. Firefox).

(This change applies only to application/x-www-form-urlencoded POST data -- other POST data, or other http verbs, are not currently supported in this way).