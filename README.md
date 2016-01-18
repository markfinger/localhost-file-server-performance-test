Localhost HTTP file server performance tests
============================================

This is a collection of scripts that seek to test the performance characteristics
of a browsers loading files from localhost. In particular, I sought to resolve
if a browser will load hundreds of files faster from either a single server or from
multiple servers.


Background
----------

The HTTP1 spec state that browsers should only use a limited set of parallel connections,
this creates a situation where assets will often be served from multiple hostnames to
circumvent the browser's limitations.

In practice, most browsers tend to ignore the specification, and will actually pull
in multiple assets at any time. The number of assets fetched in parallel tends to differ
between browsers, but each browser has been steadily increasing their number over the
years.

To resolve some fundamental issues in HTTP1 that make it difficult to fetch assets in
parallel and improving performance, the HTTP2 spec attempts to reduce the overhead
on opening multiple parallel connections.


Methodology
-----------

While browsers provide some limited performance indicators, we are testing it from a
user's perspective. That manifests as: measure the difference between timestamps taken
at the start of a HTML document's execution (an inline script element in the head) and
at the start of the window's `load` event.

To stress test the browser, the document links to hundreds of assets, each served from
either the origin server or from other localhost servers that run in separate processes
and listen on different ports.

The number of files to be tested is 100, 300, and 500, composed of blocking (non-async)
script elements.

The following combinations of servers were tested:
- 0 file servers (the origin serves assets)
- 1 file server (the origin does not serve assets)
- 3 file servers (the origin does not serve assets)
- 8 file servers (the origin does not serve assets)
- 15 file servers (the origin does not serve assets)


Conclusions
-----------

### HTTP1

There are no immediate measurable gains by splitting your service of files across localhost
ports.

As long as your origin server is non-blocking, you should probably serve the files from
the same server. This assumes that you are not performing CPU-intensive work in the server,
if so, you may experience event loop blocks which degrade the response times.

The results are contained in `data.json` and `data_second_run.json`. While some results
show performance gains when you use 3 file servers (besides the origin server), these
results cannot be replicated consistently and are likely to be reflections of system load.

The tests were run against `Chrome/47.0.2526.111` and `Firefox/41.0` on an OSX machine.


### HTTP2

Similarly to HTTP1, the results were inconsistent and ultimately inconclusive.

Anecdotally, Chrome seems to improve its performance as more servers are added. From
a very limited set of data, 8 seems to be the sweet spot.

Firefox's performance seems to degrade as you start serving files from other localhost
ports.

The results are contained in `data_http2.json`.

The tests were run against `Chrome/47.0.2526.111` and `Firefox/41.0` on an OSX machine.
