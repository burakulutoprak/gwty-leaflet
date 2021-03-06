## Overview

gwty-leaflet is wrapper for the famous maps javascript library [Leaflet](http://leafletjs.com/). gwty-leaflet is based on JsInterop and allows using Leaflet from your GWT application exactly the same way as from a javascript script with a slight advantage: static typing. It was partially generated automatically from Leaflet docs.


##Dependency 

The latest stable version of gwty-leaflet is 0.4. The source code of 0.4 version has been moved to the 0.4 branch. The current version is 0.5-SNAPSHOT and is undergoing developments to reflect the changes in the 1.0 version of leaflet: 

```xml
                     <dependency>
                        <groupId>com.gwidgets</groupId>
                        <artifactId>gwty-leaflet</artifactId>
                        <version>0.5-SNAPSHOT</version>
                     </dependency>
```

If you are using a snaphost version, then you need to include the snapshot repository: 

```xml
<repositories>
		<repository>
			<id>snapshots</id>
			<url>https://oss.sonatype.org/content/repositories/snapshots/</url>
			<releases>
				<enabled>false</enabled>
			</releases>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>
```


Also, do not forget to include gwty-leaflet in your .gwt.xml module definition file: 

```xml
 <inherits name='com.gwidgets.api.GwtyLeaflet' />
                  
```

##Leaflet version:

gwty-leaflet 0.5 is compatible with leaflet 1.0. 

##Leaflet javascript files:

As in any JsInterop wrapper, you need to refer to the Javascript files from your .html app file. You can either download the Js files from [Leaflet website](http://leafletjs.com/download.html), or refer to them directly using their cdn (there are performance implications off course!).

```html
<link rel="stylesheet" href="https://npmcdn.com/leaflet@0.7.7/dist/leaflet.css" />
<script src="https://npmcdn.com/leaflet@0.7.7/dist/leaflet.js"></script>
```

Starting from the 0.5 version, a new feature has been added to allow dynamically from the code adding leaflet resource files instead of including them manually in the .html: 

```java
LeafletResources.whenReady(
        e -> 
                 {
      MapOptions options = new MapOptions.Builder(L.latLng(52.51, 13.40), 12, 12).dragging(false).build();
    final Map map = L.map("map", options);
    GWT.log(options.getZoom()+"");
    L.tileLayer(MAP_URL, null).addTo(map);
     return null;
    });
```


##Initializing objects

All objects initializations are done through the L class. gwty-leaflet provides all factory methods of Leaflet. For example:

```java
//equivalent to new Map(...)
Map map = L.map("map", new MapOptions.Builder().build());

//equivalent to new Circle(...)
Circle circle = L.circle(L.latLng(51.508, 11), 200, options);

```
For more information about Leaflet objects creational patterns, you can refer to [Leaflet's official documentation](http://leafletjs.com/reference.html).  

## Options

As specified by Leaflet documentation, the creation of some objects requires a set of options. gwty-leaflet provides all the options with their respective default values as [Objects](https://github.com/gwidgets/gwty-leaflet/tree/master/src/main/java/com/gwidgets/api/leaflet/options) annotated with @JsType. As of version 0.4, options builders were introduced to help in the creation of option Objects and enforce fields validations. Several options have required fields, and using builders help the developer distinguish between required and optional fields. 

Before version 0.4: 
```java
              PathOptions options = new PathOptions();
                options.fillColor = "#fff";
                options.opacity = 1;

                //...

                L.circle(L.latLng(51.508, 11), 200, options).addTo(map);
```

After version 0.4: 

```java
              PathOptions options = new PathOptions.Builder()
                                     .fillColor("#fff")
                                     .opacity(1)
                                      .build();

                //...

                L.circle(L.latLng(51.508, 11), 200, options).addTo(map);
```

For more informations about the available options for each objects, and their utility. You can refer to the original leaflet documentation. 

## Example

To create a map in a div with an id="map", we can do something like: 
```java
                
                L.map("map", new MapOptions.Builder().build()).setView(L.latLng(51.505, -0.09), 12, new ZoomPanOptions.Builder().build());

```


## Events

Events are available only in some objects. Events can be handled throught the following methods: clearAllEventListeners(), on(String type, Function fn), once(String type, Function fn), off(String type, Function fn), fire(String type). 

For defining actions, events needs to be supplied with an abstract callback function that needs to be implemented by the developer. The below example will dipslay a pop up on each map click: 

```java
map.on(EventTypes.Map.CLICK, new Function(){
                        @Override
                        public JavaScriptObject call(JavaScriptObject event) {

                                MouseEvent msEvent = event.cast();
                                
                                map.openPopup("hello", msEvent.getLatlng(), new PopupOptions.Builder().build());
                                return null;
                        }
                        
                });
```

Event Objects that the event object can be cast to are (depending on the usage): 

- DragEndEvent
- ErrorEvent
- GeoJSONEvent
- LayersControlEvent
- LayerEvent
- LocationEvent
- MouseEvent
- PopupEvent
- ResizeEvent
- TileErrorEvent
- TileEvent

All events objects extend [JavaScriptObject](http://www.gwtproject.org/javadoc/latest/com/google/gwt/core/client/JavaScriptObject.html), so they can be converted from/to a JavaScriptObject using cast() method.

Events are executed following the order of registration. 

Events are explained in details in Leaflet's documentation. 

## Events types constants

There is a long list of available events for some objects, and the developer may not know what events are available for the object they are using. The [EventType](https://github.com/gwidgets/gwty-leaflet/blob/master/src/main/java/com/gwidgets/api/leaflet/events/EventTypes.java) class contains a list of subclasses which contains the available events types constants. The event type can be accessed in static a way like: EventType.{object name}.{event type constant}. For example, to register the loading event on a TileLayer : 

```java
tile.on(EventTypes.TileLayer.LOADING, new Function(){
                        @Override
                        public JavaScriptObject call(JavaScriptObject event) {

                               //cast raw event object
                               TileEvent tileEvent = event.cast();
                               //Do something
                               
                                return null;
                        }
                        
                });
```

Here is a list of the events that can be registred for objects that can handle events:

<div>
<table id="archetypes">
<thead style="font-weight:800;"> 
 <th>Object</th>
 <th>Available Events</th>
</thead>
<tbody>
  <tr>
    <td>
    Map
    </td>
    <td>
     click, dblclick, mousedown, mouseup, mouseover, mouseout, mousemove, contextmenu, focus, blur, preclick, load, unload, viewreset, movestart, move, moveend, dragstart, drag, dragend, zoomstart, zoomend, zoomlevelschange, resize, autopanstart, layeradd, layerremove, baselayerchange, overlayadd, overlayremove, locationfound, locationerror, popupopen, popupclose  
    </td>
  </tr>
  <tr>
    <td>
    Marker
    </td>
    <td>
    click, dblclick, mousedown, mouseover, mouseout, contextmenu, move, dragstart, drag, dragend, add, remove, popupopen, popupclose
   </td>
  </tr>
  <tr>
    <td>
    TileLayer
    </td>
    <td>
    loading, load, tileloadstart, tileload, tileunload, tileerror
    </td>
  </tr>
  <tr>
    <td>
      Path
    </td>
    <td>
       click, dblclick, mousedown, mouseover, mouseout, contextmenu, add, remove, popupopen, popupclose
    </td>
  </tr>
  <tr>
    <td>
      FeatureGroup
    </td>
    <td>
       click, dblclick, mouseover, mouseout, mousemove, contextmenu, layeradd, layerremove
    </td>
  </tr>
  <tr>
    <td>
      ControlLayers
    </td>
    <td>
       baselayerchange, overlayadd, overlayremove
    </td>
  </tr>
  <tr>
    <td>
      PosAnimation
    </td>
    <td>
       start, step, end
    </td>
  </tr>
</tbody>

</table>
</div>


## GWT version:

As of the current version, gwty-leaflet is compiled using GWT 2.8.0. 


