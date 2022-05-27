+++
title = "Custom vector maps with OpenMapTiles and TileServer-GL"
date = 2022-02-01
[extra]
image = "/images/posts/map_screenshot.png"
background_image = "/images/posts/map_screenshot.png"
+++

In this post we're going to setup and display our own vector maps with custom styling using free, open source software. 
This solution is ideal for experimentation and prototyping but may not scale well in production.
For simplicity we won't integrate this into any framework, but the fundamentals here will be applicable to most frameworks.

As always, the full code is available on [GitHub](https://github.com/signal32).

## Building Tiles
First we need to source our data and process it into tiles that can be served efficiently.
[OpenMapTiles](https://openmaptiles.org/) provides both an open source schema for these tiles and tools to build them from OSM data. I ran into difficulties running it on Windows, however on Ubuntu the process was quite straightforward.



Start by cloning OpenMapTiles:
 ```sh
 $ git clone https://github.com/openmaptiles/openmaptiles.git
 ```

Open `.env` and look for the zoom parameters below. For this demo I will increase the `MAX_ZOOM` to 14, so more detail can be seen when styling the maps later. This will generate files of exponentially larger size so you may wish to use the default setting first before applying this step.
```py
# Which zooms to generate with make generate-tiles-pg
MIN_ZOOM=0
MAX_ZOOM=14
```
Now we can use the convenient `quickstart` utility to download and build our tiles to `data/tiles.mbtiles`.
```sh
$ ./quickstart.sh scotland
```
This will take some time, especially if your zoom level is hight, so be patient and go grab a coffee. ☕
## Serving Tiles
Next, we're going to serve these tiles to our web clients using [TileServer-GL](https://github.com/maptiler/tileserver-gl) from 
[MapBox](https://www.mapbox.com/) who produce many mapping tool that are arguably easier to setup, however these are generally closed source, cloud based or require a license to use.

We're going to be using Docker to simplify the next few stages so go ahead and create a `docker-compose.yml` file in your project source directory.
```yaml
# docker-compose.yml
version: '3'
services:
  tileServer:
    image: maptiler/tileserver-gl:${TILESERVER_VERSION:-latest}
    restart: on-failure
    command: 
        --verbose 
        --config /config/tileserver/tileserver.json 
        --mbtiles /tile_data/tiles/tiles.mbtiles 
        --port 8090
    volumes:
      - ./config:/config
      - ${TILESERVER_TILE_DIR:-./data/tileserver}:/tile_data
    ports:
      - "${TILESERVER_PORT:-8080}:8080"
```
I like to keep configuration variables in an adjacent `.env` file like so.
```py
# .env
TILESERVER_VERSION=latest
TILESERVER_TILE_DIR=./tiles
TILESERVER_PORT=8080
```

## Displaying Tiles
For this we'll use [MapLibre-GL-JS](https://github.com/maplibre/maplibre-gl-js) an open source fork of Mapbox GL JS.
Rather than fuss about setting up a web framework we'll keep things concise by linking it into a plain old html file from a CDN.
 ```html
    <script src="https://unpkg.com/maplibre-gl@1.15.2/dist/maplibre-gl.js"></script>
    <link href="https://unpkg.com/maplibre-gl@1.15.2/dist/maplibre-gl.css" rel="stylesheet"/>
 ```

 **Note:** I'm working on finishing this post soon while using it as an example to assist styling the website in general.


## Creating a Custom Theme


## Conclusion
I used this approach to display maps in [Exlorer](https://explore.hamishweir.uk/) (working title).