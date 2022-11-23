# Basemap QuickStart Guide
 ## Intro
 The purpose of this repository is to test tileserver-gl functionality and should be able to be used to get the server up and running in minimal time. Important items included in the repository are:
 

 - Data
 - Styles
 - Config.json
 - Dockerfile

These will be discussed later after the setup.

## Setup
*Required:* Docker Desktop must be installed to use the *docker* command in the terminal

To quickly get the server running, you will want to open a command prompt and navigate to the folder where you have copied the repository. Once there, run the following command for your terminal / OS.

Windows Command Prompt:
`docker run --rm -it -v %cd%:/data -p 8080:8080 maptiler/tileserver-gl`

Powershell:
`docker run --rm -it -v $(pwd):/data -p 8080:8080 maptiler/tileserver-gl`

Linux:
`docker run --rm -it -v $(pwd):/data -p 8080:8080 maptiler/tileserver-gl`

This will start up the server and you should be able to navigate to [http://localhost:8080](localhost:8080) and see the Tileserver homepage with the styled map and the two data sources.

## Data
The initial data for this test was imported into a PostGIS database from OSM. Since the origin of the data is not really relevant to this test, I will focus on the conversion from PostGIS to MBTiles. This took two steps:

 1. Exporting data from PostGIS to GeoJSON
 2. Converting GeoJSON to MBTiles

### Exporting to GeoJSON
To export the data to GeoJSON, the *ogr2ogr* cli tool was used. By first navigating the the directory where we wanted the data stored, we were then able to use the following command in the terminal:

    ogr2ogr -f GeoJSON planet_osm_roads.geojson PG:"host='localhost' dbname='osm_test' user='postgres' password='****' port=5432" planet_osm_roads

The first part tells that the output should be GeoJSON and then also includes the output file name. Then, the next part tells where to find the data, in this case it's a PG database connection. Then the last input is the Table to be exported.
**For future testing**, we will probably want to test exporting multiple layers together. 

### Converting to MBTiles
To convert to MBTiles, the *tippecanoe* cli tool was used. Since this is a bit more complicated, the tool was run from an Ubuntu terminal using WSL2 where tippecanoe was installed. This may be able to be run from a Docker container instead.

For the command, when run from the directory where the data is stored:

    tippecanoe -z15 -s EPSG:3857 -o planet_osm_roads.mbtiles --drop-densest-as-needed --force planet_osm_roads_geojson

For the zoom, you can also use *-zg* instead of *-z15* and it will determine the number of zoom levels needed.


## Styles
Currently, there is only one style and it's very rough as it was more of a proof of concept to see how everything works. The style was created using a Maputnik Docker container.  This was run by using the following command in a terminal:

    docker run -it --rm -p 8888:8888 maputnik/editor
 
 You can then navigate to [http://localhost:8888](http://localhost:8888) to access the editor. You may want to clear most of the layers from the layer tab or you may be able re-source them later on.

Once ready, you will need to add the data being served from the tileserver as an XYZ Vector Tile. The steps to do this are:

 1. Click **Data Sources** at the top of the screen
 2. Scroll down to **Add New Source**
 3. Change the Source ID to the name you would like to give your source
 4. Change the Source Type to **Vector (XYZ Urls)**
 5. Change the Tile Url to http://localhost:8080/data/planet_osm_roads/{z}/{x}/{y}.pbf

The *planet_osm_roads* is the data name from the tileserver. If you would like to use a different layer or data source, you will need to change the url to match.

Once you have added the Source, you will then need to add the layer. To do this:

 1. Click **Add Layer** at the top of the layer list
 2. In the popup, change the ID to the name for your layer
 3. Change the type to an appropriate type: In this case we will use **Line** for the roads
 4. For the **Source**, if you click on the field, it will show a dropdown and you can select your added source
 5. For **Source Layer** you will need to enter the name of the layer from the source: In this case, the layer's name is *planet_osm_roads*
 
 The layer should now be in the layer list and you can style and duplicate the layer as needed. 
 
 For some example styling, I suggest visiting the [Maputnik Editor](https://maputnik.github.io/editor/#0.4/0/0). If you find a style you would like to copy or use as a base, you can simply copy the JSON, specifically the **paint** field, and copy it into your layer's JSON.

You can also read more about the style specifications in the [Mapbox Documentation](https://docs.mapbox.com/mapbox-gl-js/style-spec/layers/).

## Config.json
For the config, everything is mostly setup, however; if new data or styles are added, they will need to be added to the config.
More information can be found in the [Documentation](https://tileserver.readthedocs.io/en/latest/config.html).


## Docker Image Setup
Once the project is more mature, it may be possible to convert it into a Docker Image which can be easily shared / updated. With the Dockerfile in the folder, you can run the command:

    docker build -t docker_image_name:1.0.0
 
 Where the docker_image_name would be the name that is decided for the image and the 1.0.0 is the version of the image. When updating, you can then increment the version. This will cause anyone trying to use image to access the latest build (assuming they don't request a specific version). *This may require a paid docker account though to host the image.*

Once the image has been built, it can be run with the command:

    docker run --name container_name -p 8080:8080 docker_image_name:1.0.0

If hosted, you may be able to replace the *1.0.0* tag with the *latest* tag
