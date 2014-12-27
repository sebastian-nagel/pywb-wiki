With release 0.7.0, pywb features extensive support for video (and audio) replay from archives and and via the live proxy.

## HTML5

To replay HTML5 video (and audio), pywb rewrites the `src=` attribute of `<video>`, `<audio>` and `<source>` tags. Often, the video/audio is added dynamically so the rewrite happens both statically
and dynamically (via the wombat.js client rewriting library).

To fully support seek in HTML5 video/audio, pywb also supports [[Range Requests|range-requests].


### Range Requests

To allow seeking in HTML5 video/audio replay also requires support for HTTP/1.1 range requests.
pywb now supports range queries which are enabled by default. When a range request is made, pywb
will attempt to cache the full file locally, then serve the desired range.
At this time, the file is cached until the application exits. For uWSGI based application, the uWSGI
cache is used

Range request support


## Video Detection

A new client side script, `vidrw.js`, has been added which detects `<object>`, `<embed>` tags in a page
on load. A few special case handlers, most extensively for YouTube are also added.


When an object or embed tag is detected, the source is ran 