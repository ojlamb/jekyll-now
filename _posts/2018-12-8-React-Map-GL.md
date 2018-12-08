---
layout: post
title: Working with react-map-gl
---

As many of my friends and colleagues know, I really really enjoy working with [Mapbox]('mapbox.com'). There are a number of reasons why, but namely the API is very intuitive and well documented. In my humble opinion they are the best web mapping platform out there. Ability to handle large datasets and data-driven styling are

When my recent project required some geospatial data visualizations, it was an easy decision to go with Mapbox. Like many new apps on the web these days our front ui was built with [React](https://reactjs.org/) and [Redux](https://redux.js.org/). To optimize for delivery we made the decision to work with a react Mapbox component library. There are a few out there, but we landed on [react-map-gl](https://uber.github.io/react-map-gl), an open source library developed by Uber.


*"react-map-gl makes using Mapbox GL JS in React applications easy...react-map-gl is a suite of React components for Mapbox GL JS, a WebGL-powered vector and raster tile mapping library. In addition to exposing MapboxGL functionality to React apps, react-map-gl also integrates seamlessly with deck.gl."*

Here I want to share my experience with working with react-map-gl, some code to help you get started and some high level thoughts about the library.

#### react-map-gl
react-map-gl makes its pretty simple to create a web map.

```
import ReactMapGL from 'react-map-gl';

<ReactMapGL
  width={400}
  height={400}
  latitude={37.7577}
  longitude={-122.4376}
  mapStyle={MAP_STYLE}
  mapboxApiAccessToken={MAPBOX_TOKEN}
  zoom={8}
  onViewportChange={(viewport) => {
    const {width, height, latitude, longitude, zoom} = viewport;
    // Optionally call `setState` and use the state to update the map.
  }}
/>
```

 It also includes components that you might expect, like navigation, markers, popups, etc.

For the purposes of this article, I won't bore you with examples that you can see [here](https://uber.github.io/react-map-gl/#/examples). I will, however, dig into some more nuanced examples that I was able to work through over the past few months.

##### How do you display a geojson feature in react-map-gl?
Displaying a geojson feature is not entirely straight forward. It is not as simple as using the [addLayer](https://www.mapbox.com/mapbox-gl-js/example/geojson-line/) function. Developers have to download a style from [Mapbox Studio](https://www.mapbox.com/mapbox-studio/) and from there add and style geojson.

***Cool so how do i?***

I would recommend creating a map directory that contains your styles and a function library that enables you to work with the map style easier.

1. Save your style into the dir
`app/map/myStyle.json`

2. Create a function library that will enable you to work with the map easier. `app/map/map.js`
Here is what mine looks like:

```
import { fromJS } from 'immutable';
import MAP_STYLE from './myStyle.json';

export const defaultMapStyle = fromJS(MAP_STYLE);

// For more information on data-driven styles, see https://www.mapbox.com/help/gl-dds-ref/
export const fillLayer = (source, interactive) => fromJS({
  id: source,
  source,
  type: 'fill',
  interactive
});

export const circleLayer = (source, interactive) => fromJS({
  id: source,
  source,
  type: 'circle',
  interactive
});

export const lineLayer = (source, interactive) => fromJS({
  id: source,
  source,
  type: 'line',
  interactive
});

export const setLayerStyle = (layer, properties) => layer.set('paint', fromJS(properties));

export const generateMapStyle = (style, id, data, layer) => style
  .setIn(['sources', id], fromJS({
    type: 'geojson',
    data
  }))
  .set('layers', style.get('layers').push(layer));
```
Lets unpack this a little - We import our mapstyle, and create an immutable map.

We also create functions that help us create our favorite geo layers - points, lines and polygons. These functions take two arguements, the name of the layer (string) `source` and if the layer is interactive or not (boolean) `interactive.` This essentially creates a layer object that can be added to `myStyle` and ultimately the map.

The next function `setLayerStyle`, does exactly that, adds style properties to a layer. The function takes two arguements, a `layer` ( returned object created from one of the functions above) and a `properties` object. The properties must follow the [mapbox style specification](https://www.mapbox.com/mapbox-gl-js/style-spec/). You can also leverage data-driven styling and expressions here. This is some pretty powerful stuff!

Next function in the library is generateMapStyle. Its kind of a doozy but it does a lot. Its what combines or style, data and layer and adds it to the map. Heres an example of how it all comes together.

```
import MapGL from 'react-map-gl';
import { DefaultMapStyle, setLayerStyle, circleLayer, generateMapStyle } from 'common/map';
export class MyMap extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      mapData: null,
      mapStyle: myStyle,
      viewport: {
        width: 600,
        height: 600,
        latitude: -96,
        longitude: 37.8,
        zoom: 8,
      }
    };
  }

  setMapStyle() {
    const myLayer = setLayerStyle(circleLayer('myLayer', true), this.getPaintProperties());
    const mapData = <GEOJSON DATA>
    const mapStyle = generateMapStyle(DefaultMapStyle, 'myLayer', mapData, myLayer);
    this.setState({mapStyle, mapData});
  }

  getpaintProperties = () => ({
    circleColor: "ff0000",
    circleRadius: 2,
  })


  render() {
    const { mapStyle} = this.state;
    return (
      <MapGL
        mapStyle={mapStyle}
        mapboxApiAccessToken=<MAPBOX_TOKEN>
        {...this.state.viewport}
        onLoad={() => this.getMapLayers()}
        onViewportChange={this.updateViewport}
      >
    )
  }
}
```

A couple of things to note here, is the onLoad function. When the map loads for the first time, it will kick off the getMapLayers function, generate the new style and on the next render it the map will consume the new mapStyle with you custom geoJson. Sweet!
