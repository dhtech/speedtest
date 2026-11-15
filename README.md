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
- four files of zeros at `/100M`, `/1G`, `/10G` and `/100G`, served
  uncompressed and with `Cache-Control: no-store`.

On our servers the puppet module `dhspeedtest` does both: the page comes
from this repository's `master` through the application store, the files
are made on the server.

## Making the files

The files hold nothing but zeros, and they are made **sparse**: the file
has its full size, but no block of it is written, so it takes no disk
space. 100G fits on a 10G disk. A read gives zeros, which is all a speed
test needs.

Make them with `dd`, where the web server will find them, one command a
file:

```sh
cd /var/lib/dh-speedtest        # or wherever the server serves them from
dd if=/dev/zero of=100M bs=1 count=0 seek=100M
dd if=/dev/zero of=1G   bs=1 count=0 seek=1G
dd if=/dev/zero of=10G  bs=1 count=0 seek=10G
dd if=/dev/zero of=100G bs=1 count=0 seek=100G
chmod 644 100M 1G 10G 100G
```

What the options do: `count=0` copies nothing at all, and `seek=10G`
first moves 10G into the output file - so `dd` only sets the file's
size and writes no data. Each command is done at once and says
`0+0 records in, 0+0 records out`.

The sizes are binary: to `dd`, `100M` is 100 x 1024 x 1024 bytes and
`1G` is 1024 MB (it is `MB` and `GB` that would be the decimal ones).
The page counts with the same numbers.

Check that they are the right size and really take no space:

```sh
ls -l  100M 1G 10G 100G      # the sizes: 104857600, 1073741824, ...
du -h  100M 1G 10G 100G      # the space used: 0 for each
```

How the web server should hand them out, as nginx has it on our server:

```nginx
location ~ ^/(100M|1G|10G|100G)$ {
  root         /var/lib/dh-speedtest;
  types        { }
  default_type application/octet-stream;
  gzip         off;                      # zeros compress to nothing
  sendfile     on;                       # the kernel sends the file
  add_header   Cache-Control "no-store"; # nothing on the way keeps it
}
```
