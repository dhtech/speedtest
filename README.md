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

Make them where the web server will find them, one command a file:

```sh
cd /var/lib/dh-speedtest        # or wherever the server serves them from
truncate -s 100M 100M
truncate -s 1G   1G
truncate -s 10G  10G
truncate -s 100G 100G
chmod 644 100M 1G 10G 100G
```

The sizes are binary: `100M` is 100 x 1024 x 1024 bytes, `1G` is 1024 MB.
The page counts with the same numbers.

Check that they are the right size and really take no space:

```sh
ls -l  100M 1G 10G 100G      # the sizes: 104857600, 1073741824, ...
du -h  100M 1G 10G 100G      # the space used: 0 for each
```

Things to know:

- Do not make them with `fallocate` or by writing zeros
  (`dd if=/dev/zero of=10G bs=1M count=10240`): both really fill the
  disk. If `truncate` is missing, this is the same thing with `dd`:
  `dd if=/dev/zero of=10G bs=1 count=0 seek=10G`.
- A plain copy can turn a sparse file into a full one. Copy with
  `cp --sparse=always` or `rsync -S`, or simply make the files again on
  the new machine - that is quicker than copying them.
- Another size is one more `truncate` and one more tile in
  `index.html`: the link (`href="/50G"`) and its size in bytes
  (`data-bytes`).

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
