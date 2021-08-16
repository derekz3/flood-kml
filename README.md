# flood-kml

Unblockers: 

* Google Maps Platform > Web > Maps Javascript API > [KML Layers](https://developers.google.com/maps/documentation/javascript/examples/layer-kml) > Google Maps JavaScript API Sample Application > [sample-layer-kml branch](https://github.com/googlemaps/js-samples/tree/sample-layer-kml) > [cta.kml file](https://github.com/googlearchive/js-v2-samples/blob/gh-pages/ggeoxml/cta.kml)

BLocKers:

* [Hosting KML on Public Webserver](https://stackoverflow.com/questions/28212176/hosting-kml-files-on-public-webserver)
    * Lots of solutions are offered, but I chose to mimic what I found in the sample application associated with the Maps JavaScript API documentation by the layers we want to display in a github repo (although I didn't archive the repo). [*This is our kml storage repo*](https://github.com/derekz3/flood-kml).   

* [Google Maps KML Compatibility](https://geopointe.force.com/help/s/article/My-KML-Layer-has-data-but-is-not-appearing-on-the-Map-What-s-Wrong#:~:text=Information&text=There%20are%20a%20couple%20of,not%20appearing%20on%20the%20Map%3A&text=Fortunately%2C%20there%20is%20a%20workaround,be%20compatible%20with%20Google%20Maps.)  
    * Google Maps has certain capabilities when it comes to processing (and being able to display) KMLs. In order to ensure that Google Maps is capable of rendering our KML, we need to ensure that all the tags that we use belong to [Google's list of supported elements](https://developers.google.com/maps/documentation/javascript/kmllayer#supported-elements%E2%80%8B), i.e. remove any unsupported elements. An easy workaround is to upload the file to [Google My Maps](https://mymaps.google.com/), then export it out as a KML. The exported KML will be compatible with Google Maps.  
    * Well, not quite. We still have to make a couple edits.  
    
    Namely, the first two lines of the KML need to be:  
    
    ```
        <?xml version="1.0" encoding="UTF-8"?>
        <kml xmlns="http://earth.google.com/kml/2.1">
    ```

    in order for the file to be recognized as compatible with Google Maps.  

* Formatting the KML

    * Next, we need some proper styling, so the polygons don't look awful.  

    For that, the `<Style>` block should look as follows:  

    ```
        <Style id="poly-0288D1-1-255-normal">
        <LineStyle>
            <color>5f1478ff</color>
            <width>2</width>
        </LineStyle>
        <PolyStyle>
            <color>5f1478ff</color>
            <fill>1</fill>
            <outline>0</outline>
        </PolyStyle>
        </Style>
        <Style id="poly-0288D1-1-255-highlight">
        <LineStyle>
            <color>5f1478ff</color>
            <width>2</width>
        </LineStyle>
        <PolyStyle>
            <color>5f1478ff</color>
            <fill>1</fill>
            <outline>0</outline>
        </PolyStyle>
        </Style>
    ```  
    
    * `<fill>`: Boolean value. Specifies whether to fill the polygon.  
    * `<outline>`: Boolean value. Specifies whether to outline the polygon. Polygon outlines use the current LineStyle.  
    * `<color>` and `<width>` of line self-explanatory.

    * Reference these sites for coloring:  
        * [A simple color KML / RGB color converter](http://netdelight.be/kml/index.php)  
        * [ColorHexa (Color encyclopedia : Information and conversion.)](https://www.colorhexa.com/ff7814)  
        * [KML Color](http://www.zonums.com/gmaps/kml_color/)


* [Size/Complexity Restrictions](https://developers.google.com/maps/documentation/javascript/kmllayer#restrictions), [[*TLDR*]](https://stackoverflow.com/questions/42149489/kml-layer-not-displaying-on-map)  
    * The Google Maps JavaScript API has limitations to the size and complexity of loaded KML files. Most notably for us, the maximum fetched file size of a raw KML or compressed KMZ is only 3MB (the 100 year flood plain alone is over 30MB), the maximum uncompressed KML file size is 10MB, and there is a limit on the number of KML Layers that can be displayed on a single Google Map. the number of layers you can add will vary by application; on average, you should be able to load between 10 and 20 layers without hitting the limit. If you still hit the limit, use a URL shortener to shorten the KML URLs. Alternatively, create a single KML file consisting of NetworkLinks to the individual KML URLs.  
        * For these reasons, I've decided to split the `100yr.shp` file (vector layer) by the `SOURCE_CIT` attribute, yielding 51 smaller sized layers for now.  
        * Turns out, 90% of the polygons are accounted for by 15 of these layers.  
    * For best performance, it is recommended that we:  
        * Generate the files before they will be needed, and serve them statically (if it takes a long time for your server to transmit the KML file, the KmlLayer may not display).  
        * Avoid using query parameters to limit the viewport of layers (instead, you can limit the map viewport with the bounds_changed event. Users will only be sent features that can be displayed automatically. If there is a large amount of data in your geospatial data server, consider using [data layers](https://developers.google.com/maps/documentation/javascript/datalayer) instead).  
        * Use multiple KmlLayers for each group of features that you wish to permit users to toggle, rather than a single KmlLayer with different query parameters.  
        * Use compressed KMZ files to reduce file size, Reduce the precision of all points to an appropriate precision.  
        * Merge and simplify the geometry of similar features, such as polygons and polylines.  
        * Remove any unused elements or image resources.  
