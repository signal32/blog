+++
title = "Custom vector maps using OpenMapTiles and TileServer-GL"
date = 2022-02-01
+++

In this post we're going to setup and display our own vector maps using custom styling and completely open source software. 
For simplicity we won't integrate this into any framework, but the fundamentals here will be applicable to most frameworks.

As always, the full code is available on [GitHub](https://github.com/signal32).

## Building Tiles
First we need to source our data and process it into tiles that can be served quickly and efficiently.
[OpenMapTiles](https://openmaptiles.org/) provides both an open source schema for these tiles and tools to build them from OSM data.

Start by cloning OpenMapTiles:
 ```sh
 $ git clone https://github.com/openmaptiles/openmaptiles.git
 ```

Open `.env` and look for the zoom parameters below. For this demo I will increase the `MAX_ZOOM` to 14, so more detail can be seen later after styling the maps. 
```py
# Which zooms to generate with make generate-tiles-pg
MIN_ZOOM=0
MAX_ZOOM=7
```
Now we can use the convenient `quickstart` utility to download and build our tiles to `data/tiles.mbtiles`. This may take some time so be patient and go grab a coffee.
```sh
$ ./quickstart.sh scotland
```
## Serving Tiles
Next we're going to serve these tiles to our web clients using [TileServer-GL](https://github.com/maptiler/tileserver-gl) from 
[MapBox](https://www.mapbox.com/) who produce many mapping tool that are arguable easier to setup, however these are generally closed source, cloud based or require a license to use.

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


## Creating a Custom Theme


## Conclusion

