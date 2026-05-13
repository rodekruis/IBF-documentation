# Mapping Best Practices

This is a collection of the research done into mapping frontends and backends for NRW. It shows our best practices for mapping and reasons for choosing them.

## Terms:
- Static image: An image loaded like on any normal webpage (i.e. from public/images/my_cool_data.png)
- Tile image: A big image sits on a GIS tile server. We send a request for a square chunk of it at a specific zoom level. The server then creates the chunk at the right resolution, and we download each chunk that way, for each zoom level.

## Why use static images instead of tiled raster images loaded from a map server

- Smoother interaction (pan/zoom) for the user. Tiled images load while the user is trying to use the map, causing a jerky, slow experience.
- The longer initial load of the static image is often not very long (less than a second for most data we use)
- With tiles, we need to keep reloading the image for every pan and zoom.
- Tiles cause a lot of concurrent requests, especially with more than one visible map layer. This can exhaust the connections of the browser. This means if one tiled layer is slow, because these request pile up quickly, it can block all layers from loading, which makes UX degrade even more.
- After only a few seconds of zoom/pan, the downloaded image tile data size is more than 5MB. Our largest static images are expected to be around 500kb to 2MB.
- A large GeoTiff (i.e. 50MB) often converts well to a lossless static image (png) at around 500kb-6MB. More compression is possible when compressing the range of the values.
- GeoServer needs to process the layers to serve tiles. This takes a lot of memory and time to process. We can’t cache all layer tiles. Using an image pyramid can help speed up tile requests, but the interaction is still not as smooth (panning/zooming/loading) as using a static image.
- Tiles would make sense if we had a) a lot of users loading the same layer, and b) massive images that compressed to 20+ MB png.
- If the files get larger (above 15MB in PNG format), rather than going to a tile server, it would be a much better user experience to either a) add a loading spinner, or b) make our own low/high resolution PNG images, and possibly cut the PNG into two or more images (basically we would tile it ourselves).

## Map Projection

- Base maps (such as Open Street Maps and MapTiler) are in EPSG 3857
- If the overlay data is in a different projection (EPSG 4326), OpenLayers can reproject it correctly. However, there will be a slight delay for it to display whenever a user moves the map. Using EPSG 3857 means no render delay.

## Vector Simplification

Vector line simplification is done by removing points from the line and gives these benefits:

- Smaller download size for the client
- Cleaner rendering when zoomed out

There are two ways to do this:

1. Query with the ST_Simplify function (used in NRW)

This is supported by some backends (i.e. pg_featureserv), but not all (i.e. GeoServer does not). For pg_featureserv, you need to whitelist the ST_Simplify function in the server settings, and then it is ready to use, such as with `…/items?&transform=simplify,0.01` on the query url.

2. Preprocessing the data

This is the only option for GeoServer. A common way to do this is to use Geopandas geometry.simplify(), which creates a new simplified copy of the data. If you need multiple detail levels, consider placing them in the same DB table, but with a zoom level or LOD value to query.

## Optimizations useful for mapping front ends, including OpenLayers

- Query with extents to prevent querying too much
- Set crossOrigin: 'anonymous' for layers where possible. This is needed also if you use custom shaders.
- You can set a max zoom level for MVT and Vector layers to reduce the data size.
- You can override zoom settings by writing the URL yourself, even for WMS. See here: https://openlayers.org/en/latest/examples/mapbox-vector-tiles-advanced.html
- Limit details by zoom level to prevent them being loaded. 
- If you load data GeoJson directly or via WFS, you only need to load it once for all zoom levels.
- MVT is good for dense quantities of vector data. However these reload for zoom changes.
- Generally use WFS or direct download for something sparse like country borders, while MVT is good for roads/buildings.
- Use CQL queries to not load everything.


## Raster Data

- Flatten data range in images for better PNG compression. PNGs can have over 16 million colors, but for map visuals, you might only need 10-20 visual color differences. If you flatten the range of colors (change color in steps rather than a curve), this can greatly shrink the PNG size.
- Data PNGs can have outliers that can mess up colorization or creation of legends. Use log scales, or find another way to remove high point outliers (such as capping max range at top 10th percentile).
- To get smooth map panning of raster images in OpenLayers (also with other map frontends), you must do each of these:

1. Use no post-process shaders
2. Use the base map projection (EPSG:3857) for the image. If it's in EPSG:4326, you can preprocess it into the correct projection.

## MapTiler

- When loading vector data from MapTiler, you can set max zoom very low (like 1 or 2) and you will always load a low-point vector line, even if the map user zooms in close.

## When to use the different image loading types

### MVT

 - Large vector data that is difficult to split into smaller, regional chunks
 - dense vector data

### WFS

 - Vector data that can be queried
 - Less dense vector data (like borders)

### WMTS

 - Images that are still large after PNG compression

### WMS

 - This is like WMTS with no caching, and is not recommended to use.

### Static images

 - Images that compress well to PNG (has large areas of same color)
 - These are passed to the front end like standard images with a meta data file to say how to display it.

## GeoServer

GeoServer is not being used in NRW, but if we ever need a raster tile server again, this is a good one to test against. Here are notes on issues with GeoServer.

- If GeoServer starts but always returns a 404, there is a chance one of the imported data settings files is bad. Sometimes the DB password is stored as a string in the postgis_db/datastore.xml file. If this fails to import, the whole server will only return 404. See the `passwd` key. You can also check the logs for missing or incorrect password errors, or try backing up and clearing the data directory to see if that fixes it.
- Every tile takes a connection to get the data, and if GeoServer loads slowly, it will exhaust all browser connections, causing every other request on the page to slow down. The more requests to GeoServer (more layers) make this worse, and will even cause the base map to have a long delay in loading.
- If GeoServer is unresponsive, and even the docker container is unresponsive, it may have run out of memory.
- GeoServer uses a lots more memory for the local docker container than it says. I think this has to do with emulation (since it runs natively on Linux). If it says it needs 2GB, it may actually need 8GB.
- To get past CORS errors when running locally, I found I needed the GeoServer control-flow extension installed.
- If GeoServer returns 429 errors, it means the connection is getting throttled. See the control-flow extension docs to see how to increase the throttle.
