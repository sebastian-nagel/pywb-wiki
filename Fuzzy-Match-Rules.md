From early versions, pywb has supported a powerful mechanism of 'fuzzy matching' to allow for replay of difficult or dynamically changing content. Fuzzy matching is appropriate whenever a page is requested with a dynamically changing url, for example, a timestamp or some other client-generated parameter which may change.

Fuzzy matching works by doing a prefix search on a modified version of the url (by default, the url without the query) and applying a regex on the specified url and the url in the cdx index.
The match, replace and filter applied to the cdx can all be customized allowing for very flexible 'fuzziness' of the match.


### Query Param Match Rule

Fuzzy matching rules are specified in [rules.yaml](../blob/master/config.yaml) and are entered per SURT prefix.

A simple example might be as follows:

```yaml
   - url_prefix: 'com,example)/ajax/complexquery'

     fuzzy_lookup: '([?&]id=[^&]+)'
```
(Note: when running with non-SURT cdx indexes, the rules are automatically converted to non-SURT form and should generally work unless there are subdomain wildcards in the url_prefix)

Suppose a capture exists for:
```
com,example)/ajax/complexquery?foo=bar&id=123&another=param
```

but at replay time, there is a request for:
```
com,example)/ajax/complexquery?_extra=param&fooz=bar2&some=value&id=123
```

An exact match is tried first, and if it isn't found, the fuzzy match system is activated.
A rule match is made for prefix `com,example)/ajax/complexquery` and the fuzzy lookup query is applied to both the lookup url and each url starting with the prefix (a `matchType=prefix` query is used on the cdx server). The result is equivalent to the following test:

```python
>>> re.search('([?&]id=[^&]+)', 'com,example)/ajax/complexquery?foo=bar&id=123&another=param').groups()
('&id=123',)

>>> re.search('([?&]id=[^&]+)', 'com,example)/ajax/complexquery?_extra=param&fooz=bar2&some=value&id=123').groups()
('&id=123',)
```

Since the groups match, this url is considered a match and the first matching cdx line is returned.
Note that the url prefix `com,example)/ajax/complexquery?` need not be specified as part of the regex, since its already guaranteed to match.


### Matching Multiple Query Params

Multiple capture groups can be used to match against multiple individual params, for example, if we wanted to match both `id` and `foo`, the fuzzy match rule could be:
```yaml
   fuzzy_lookup: '([?&]foo=[^&]+).*([?&]id=[^&]+)'
```

Note: the params are canonicalized in alphabetical order, so should be specified this way in the regex.

### Simplified Query Match Rules (from 0.6.1)

Most of the fuzzy matching involves query params, pywb 0.6.1 introduces a simplified notation, using a list of query params:

```yaml
   fuzzy_lookup:
      - foo
      - id
```

This syntax will convert to a fuzzy match regex matching the foo and id params. (The conversion will also ensure the params are alpha sorted and regex escaped)

### Multiple rules

There can only be a single `fuzzy_lookup` entry per `url_prefix`, however there is no limit on `url_prefix` entries. Note that the params are applied in order, so more specific rules should appear first:

```yaml
   - url_prefix: 'com,example)/path/foo/bar.html'

   ...

   - url_prefix: 'com,example)/path/'
   ...

   # catch-all
   - url_prefix: ''

```

### Query Match with Regex (from 0.7.0)

Sometimes having just a static `url_prefix` or multiple prefixes and the query args is not quite sufficient. It is useful to be able to specify a custom regex for the non-query part of the url.
This can now be done as follows:

```yaml

   url_prefix: 'com,example,'

   fuzzy_lookup:
       match:
           regex: 'com,example,.*\)/path/endpoint'
           args:
               - foo
               - id
```

This is equivalent to the following:

```yaml
   fuzzy_lookup: 'com,example,.*\)/path/endpoint([?&]foo=[^&]+).*([?&]id=[^&]+)'
```

### Other Complex Use Cases

Currently, the most common form of:

```yaml
   fuzzy_lookup: <regex>
```

is actually equivalent to the longer:

```yaml
   fuzzy_lookup:
     match: <regex>
     filter: '~urlkey:{0}'
     replace: '?'
```

The `filter` param is used to specify how to cdx server filter param, with `~urlkey:{0}` indicating a prefix match.

The `replace` param indicates the last token to replace for prefix search. Setting it to `?` indicates that the query string on will be truncated.

One example which uses this extended syntax is the the following catch-all rule for removing `_=timestamp` or `uncache=timestamp` params:

```yaml
    - url_prefix: ''
      fuzzy_lookup:
        match: '(.*)[&?](?:_|uncache)=[\d]+[&]?'
        filter: '=urlkey:{0}'
        replace: '?'
```

The extended form is used to change the filter to `=urlkey:{0}` from `~urlkey:{0}` to indicate an exact instead of a prefix match.
This notation is undergoing review and may change in a future version.
Any such changes will be documented here and in the changelist.