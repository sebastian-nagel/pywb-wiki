#### Table of Contents

* [Intro and Usage](#intro-and-usage)
* [Supported Sites](#supported-sites)
* [How It Works](#how-it-works)
* [Testing Video](#testing-video)
   - [WebRecorder.io](#webrecorderio)
   - [pywb-webrecorder](#pywb-webrecorder)
   - [pywb Live Proxy](#pywb-live-proxy)
* [Choosing Player Type](#choosing-player-type)
* [Implementation Details](#implementation-details)



# Intro and Usage

With release 0.7.0, pywb features extensive support for video (and audio) replay from web archive files as well via live rewrite proxy. Combining the live proxy support with [pywb-webrecorder](github.com/ikreymer/pywb-webrecorder) allows for live recording of video/audio which can then be played back from the archive.

At this time, the replay is designed to work well with video and audio recorded via live proxy. Due to the complexity of video archiving, additional preparation may be needed to replay video and audio recorded via other methods. Video recording and replay work best in Chrome, followed by Firefox, tested on Linux and OS X. Other browsers have not been tested at this time, though browsers supporting HTML 5 should work.

### Supported Sites

In order to support a wide variety of sites, pywb uses several methods to attempt to properly replay video.

* Any plain HTML5 video should work as the HTML `src=` attributes are rewritten similar to other HTML content.

* For flash based videos, any site [supported by youtube-dl](http://rg3.github.io/youtube-dl/supportedsites.html) should work. pywb uses the excellent [youtube-dl](http://rg3.github.io/youtube-dl/) project to detect and download videos that are not plain HTML.

## How it Works

* In general case, pywb will attempt to replace any flash based with equivalent HTML5 replay.

* If that fails (or HTML5 is not supported), the flash version of [FlowPlayer](https://flowplayer.org/) is used to attempt to play back the video stream.

* For certain common video sites, an attempt is made to use the original HTML5 player. Currently this applies only to YouTube, but other common video sites may be added in the future. If the original player playback fails, either the HTML5 or FlowPlayer option is tried.

## Testing Video

There are several ways to play around with the video recording and replay features.

#### WebRecorder.io

The easiest way to test out the new features is to use the WebRecorder.io service which is based on pywb.

For instance, you may record a video by visiting:
https://webrecorder.io/record/https://www.youtube.com/watch?v=_MgzgwOfNOE

and then play it back by visiting:
https://webrecorder.io/replay/https://www.youtube.com/watch?v=_MgzgwOfNOE

or simply preview without recording via:
https://webrecorder.io/preview/https://www.youtube.com/watch?v=_MgzgwOfNOE

#### pywb-webrecorder

If you'd like to record a video locally, you may run the [pywb-webrecorder](github.com/ikreymer/pywb-webrecorder) application to record a video via live proxy. Refer to [pywb-webrecorder](github.com/ikreymer/pywb-webrecorder) documentation for more info on setting up this app.

Using the default settings, you can point your browser to:
http://localhost:8080/record/https://www.youtube.com/watch?v=_MgzgwOfNOE

Then point, your browser to:
http://localhost:8080/replay/https://www.youtube.com/watch?v=_MgzgwOfNOE

to play back the video. Note: recording the video may take some time. If the video is still recording,
you will see `Still Downloading...` messages in the log.

#### pywb Live Proxy

To see the video rewrite directly without any other tools, you can run the `live-rewrite-server` provided with pywb. Then, point a browser to `http://localhost:8090/rewrite/https://www.youtube.com/watch?v=_MgzgwOfNOE` (This sample app simply proxies the live web through the pywb rewriting system).
This may be useful for testing if a video will work, as the other tools are built on top of the same system.

## Choosing Player Type

*Note: The below options are still experimental and subject to change as of pywb 0.7.2*

By adding the following anchor *#_pywbvid=type*, it is possible to explicitly select which player will be used by the client side video library. These are most useful for recording, although it is often possible to record with html player and then replay with the flash player.

When not specified, the players are tried in this order of preference: first the original HTML5 player (currently available for YouTube only), then the native browser HTML5, and then FlowPlayer Flash, if all else has failed.


#### `_pywbvid=orig`

Prefer the original player. Currently available for YouTube only. This will attempt to use the YouTube HTML5 player (also default option), and even replace the YouTube flash player with YT HTML 5 player (not the default).

Ex: https://webrecorder.io/preview/https://www.youtube.com/watch?v=_MgzgwOfNOE#_pywbvid=orig

#### `_pywbvid=html`

This will force the browser's HTML5 player. This is the default for all sites except youtube at the moment.

Ex: https://webrecorder.io/preview/https://www.youtube.com/watch?v=_MgzgwOfNOE#_pywbvid=html

#### `_pywbvid=flash`

This option will prefer the FlowPlayer flash player to be used instead of the original or browser HTML5 player. By default, the Flash player is used only when HTML5 is not available, however this option will force this particular player.

Ex: https://webrecorder.io/preview/https://www.youtube.com/watch?v=_MgzgwOfNOE#_pywbvid=flash


This options are still experimental and may not always work. When recording with `_pywbvid=flash` or `_pywbvid=html`, it is usually not possible to play back with the original player (`_pywbvid=orig`) as that path is not recorded. However, the video will still play back with the best available player.
These options are for advanced users and generally not necessary.


# Implementation Details

The following sections explain in more detail how the video replay system works. They will be expanded as needed.

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

When live proxying/recording, the youtube-dl info is created live.

When replaying from the archive, the youtube-dl info is looked up in the WARC file.
When using pywb-webrecorder, this happens automatically.
For legacy videos/videos not recorded via pywb-webrecorder/warcprox, it may be necessary to generate
the video info file to map the video container to the actual video. This is beyond the scope of this first release.


## Recording with live proxy / pywb-webrecorder / webrecorder.io

When running with pywb-webrecorder, the live rewrite with proxy mode is used, and the proxy
is assumed to be 'video aware'. Both HTML5 requests and video info requests will go through the proxy,
and so extra logic exists to ensure that videos are recorded as best as possible: (Currently, warcprox is assumed to be the proxy but in theory and proxy can work).

* The video info files are sent to the proxy (eg. warcprox) via a special request where they are written as WARC metadata records.

* Multiple range requests to same url are not sent to the recording proxy. The range request is removed before being sent to the recording proxy.

* Video info is automatically created (if any) for a url the first time a range request is used.

* Multiple `range=` argument requests treated as range requests (see below).

#### Youtube and DASH Support (0.7.2 update)

The YouTube HTML5 player uses DASH (http://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) by default, which uses HTTP requests with a `range=` query argument instead of HTTP 1.1 Range header.

pywb 0.7.0 initially included special case support for DASH, where it looks for the `range=` argument in addition to the Range header and treats both the same way. (Multiple `range=` arguments are not sent to the proxy, for instance).

Although DASH recording and replay is still supported, replay was not consistent from browser to browser as video stream  (due to the nature of the protocol).

With version 0.7.2, pywb ensures that the DASH replay is actually disabled in the YouTube player to ensure maximum compatibility. DASH support may be added as an option in future versions.