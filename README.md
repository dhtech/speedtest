# DreamHack Speedtest

The page of the event's speed test server (`speedtest1.event.dreamhack.se`).

It is one file, `index.html`: plain HTML5, its own CSS and a small
script, no libraries, no fonts, nothing to build.

## What it does

Four tiles: 100M, 1G, 10G, 100G. Picking one makes the browser fetch a
file of that size from the server and shows how fast it arrives: the
speed over the last second, the average, the fastest second, how much
has come and how long it took. Nothing is kept in memory, so 100G works.

Without scripts the tiles are ordinary links to the files, as the first
version of this page had them. The page also gives the `curl` and `wget`
lines for a terminal: a browser cannot count as fast as a fast link
delivers.

## What the server must provide

- this directory as the web root;
- four files of zeros at `/100M`, `/1G`, `/10G` and `/100G` (binary
  sizes; sparse files take no disk: `truncate -s 10G 10G`), served
  uncompressed and with `Cache-Control: no-store`.

On our servers the puppet module `dhspeedtest` does both: the page comes
from this repository's `master` through the application store, the files
are made on the server.

## Trying a change

Serve a directory that holds `index.html` and a couple of the files
(`truncate -s 100M 100M`) with any web server, for example
`python3 -m http.server`, and open it.

## History

The first version (2012 or so, `dreamhackcrew/speedtest`) was the same
four links, drawn with jQuery and a font-replacement script. Rewritten
2026-11-15 without them.
