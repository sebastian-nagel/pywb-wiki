With release 0.7.0, pywb features extensive support for video (and audio) replay from archives and and via the live proxy. Using pywb-webrecorder allows for live proxy recording of video/audio which can then be played back from any WARC file.

## HTML5

To replay HTML5 video (and audio), pywb rewrites the `src=` attribute of `<video>`, `<audio>` and `<source>` tags. Often, the video/audio is added dynamically so the rewrite happens both statically
and dynamically (via the wombat.js client rewriting library).

To fully support seek in HTML5 video/audio, pywb also supports HTTP/1.1 Range requests.

### Range Request Cache

pywb now supports serving HTTP/1.1 range requests for any archived page, not just videos.
pywb uses an experimental range cache, where it will extract the contents of the WARC record into a temporary file, and serve range requests from the uncompressed file. (This is necessary as most WARC records are compressed and do not allow seek).
At this time, the file is cached until the application exits. When using uWSGI, the cache is shared
amongst multiple workers, otherwise exists per worker.

It should be noted that this cache is mostly for testing. In production, running with nginx is recommended as it can provide this cache on its own)

## Flash

Full flash replay is difficult as that requires intercepting url requests from different flash players
and redirecting them to the archive.

Instead, pywb attempts to replace any Flash based video player with an HTML5 Player, or, if that fails, with a different Flash player, FlowPlayer 3.

A new client side video replay library called, `vidrw.js`, detects `<object>` and `<embed>` tags and attempts to replace the custom player with a known HTML or FlowPlayer replacement.

Determining the actual video stream from a Flash object can be difficult. Luckily, the [youtube-dl](http://rg3.github.io/youtube-dl/) library solves a lot of these problems and is used by pywb internally to determine the actual video.

### Video Info and Youtube-dl

When using the live rewrite mode, pywb will use youtube-dl to determine if a particular <embed> or <object> tag represents a video. youtube-dl is used not to actually download the video, but to obtain a JSON info file representing the available formats.

Since this check often happens from the client (from the `vidrw.js` library), a special modifier, `vi_`, , has been added to query youtube-dl directly:

`http://localhost:8080/pywb/vi_/<url>` results in a call to: `youtube-dl -J <url>`
The returned JSON is returned directly with the content type `application/vnd.youtube-dl_formats+json`
and allows the client to choose from multiple formats. The `vidrw.js` library includes logic to check each available format and pick the best one that is supported by the browser natively (HTML5 replay).
If no supported format could be found, pywb then loads an included version of FlowPlayer and attempts to play back the video with FlowPlayer.

## Recording with live proxy and pywb-webrecorder

When running with pywb-webrecorder, the live rewrite with proxy mode is used, and the proxy
is assumed to be 'video aware'. Both HTML5 requests and video info requests will go through the proxy,
and so extra logic exists to ensure that videos are recorded as best as possible: (Currently, warcprox is assumed to be the proxy but in theory and proxy can work).

* The video info files are sent to the proxy (eg. warcprox) via a special request where they are written as WARC metadata records.

* Multiple range requests to same url are not sent to the recording proxy. The range request is removed before being sent to the recording proxy.

* Video info is automatically created (if any) for a url the first time a range request is used.

