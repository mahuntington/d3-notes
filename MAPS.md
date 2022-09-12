# Creating a map

The topics that we will cover in this chapter include:

1. Creating a map
1. Define GeoJSON
1. Use a projection
1. Generate a `<path>` using a projection and the GeoJSON data
1. Create a rectangular overlay based on coordinates entered into a form
1. Draw a rectangle on the map and display its map coordinates

In this section we'll generate `<path>` elements from GeoJSON data that will draw a map of the world

## Define GeoJSON

GeoJSON is just JSON data that has specific properties that are assigned specific data types.  Here's an example:

```javascript
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [125.6, 10.1]
    },
    "properties": {
        "name": "Dinagat Islands"
    }
}
```

In this example, we have one `Feature` who's geometry is a `Point` with the coordinates `[125.6, 10.1]`.  It has "Dinagat Islands" as its name.  Each `Feature` follows this general structure:

```javascript
{
    "type": STRING,
    "geometry": {
        "type": STRING,
        "coordinates": ARRAY
    },
    "properties": OBJECT
}
```

We can also have a `Feature Collection` which is many `Features` grouped together in a `features` array:

```javascript
{
    "type": "FeatureCollection",
    "features": [
        {
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [102.0, 0.5]
            },
            "properties": {
                "prop0": "value0"
            }
        },
        {
            "type": "Feature",
            "geometry": {
                "type": "LineString",
                "coordinates": [
                    [102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]
                ]
            },
            "properties": {
                "prop0": "value0",
                "prop1": 0.0
            }
        },
        {
            "type": "Feature",
            "geometry": {
                "type": "Polygon",
                "coordinates": [
                    [
                        [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
                        [100.0, 1.0], [100.0, 0.0]
                    ]
                ]
            },
            "properties": {
                "prop0": "value0",
                "prop1": { "this": "that" }
            }
        }
    ]
}
```

This basically follows the form:

```javascript
{
    "type": "FeatureCollection",
    "features": ARRAY
}
```

The `features` property is an array of `feature` objects which we've defined previously.

### Set up the HTML

Let's set up a basic D3 page:

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
    <meta charset="utf-8">
    <title></title>
    <script src="https://d3js.org/d3.v7.min.js" charset="utf-8"></script>
    <script src="https://cdn.rawgit.com/mahuntington/mapping-demo/master/map_data.js" charset="utf-8"></script>
</head>
<body>
    <svg></svg>
    <script src="app.js" charset="utf-8"></script>
</body>
</html>
```

The only thing different from the setup that we've used in previous chapters is this line:

```html
<script src="https://cdn.rawgit.com/mahuntington/mapping-demo/master/map_data.js" charset="utf-8"></script>
```

This just loads an external javascript file which sets our GeoJSON data to a variable.  Here's what the beginning of it looks like:

```javascript
var map_json = {
    type: "FeatureCollection",
    features: [
        {
            type: "Feature",
            id: "AFG",
            properties: {
                name: "Afghanistan"
            },
            geometry: {
                type: "Polygon",
                coordinates: [
                    //lots of coordinates
                ]
            }
        }
        // lots of other countries
    ]
}
```

Note that the `map_json` variable is just a JavaScript object that adheres to the GeoJSON structure (it adds an `id` property which is optional).  This is very important.  If the object didn't adhere to the GeoJSON structure, D3 would not work as it should.

In production, you would probably make an AJAX call to get this data, or at the very least, create your own geoJSON file similar to the one being hosted on rawgit.com.  The setup above was created to make learning easier by decreasing the complexity associated with AJAX.

## Use a projection

Now let's start our `app.js` file:

```javascript
const width = 960;
const height = 490;

d3.select('svg')
    .attr('width', width)
    .attr('height', height);
```

At the bottom of `app.js` let's add:

```javascript
const worldProjection = d3.geoEquirectangular();
```

This generates a projection, which governs how we're going to display a round world on a flat screen.  There's lots of different types of projections we can use: https://github.com/d3/d3-geo/blob/master/README.md#azimuthal-projections

The line above tells D3 to create an equirectangular projection (https://github.com/d3/d3-geo/blob/master/README.md#geoEquirectangular)

## Generate a `<path>` using a projection and the GeoJSON data

Now that we have our projection, we're going to generate `<path>` elements for each data element in the `map_json.features` array.  Then we set the fill of each element to `#099`.  Add this at the end of app.js:

```javascript
d3.select('svg').selectAll('path')
    .data(map_json.features)
    .enter()
    .append('path')
    .attr('fill', '#099');
```

Here's what it should look like at the moment if we open index.html in Chrome and view the elements tab in the developer tools:

![](https://i.imgur.com/ljSlk4s.png)

We created the `path` elements, but they each need a `d` attribute which will determine how they're going to drawn (i.e. their shape).

We want something like:

```javascript
d3.selectAll('path').attr('d', (datum, index)=>{
    //somehow use datum to generate the value for the 'd' attributes
});
```

Writing the kind of code described in the comment above would be very difficult.  Luckily, D3 can generate that entire function for us.  All we need to do is specify the projection that we created earlier.  At the bottom of `app.js` add the following:

```javascript
const dAttributeFunction = d3.geoPath()
    .projection(worldProjection);

d3.selectAll('path').attr('d', dAttributeFunction);
```

`geoPath()` generates the function that we'll use for the `d` attribute, and `projection(worldProjection)` tells it to use the `worldProjection` var created earlier so that the `path` elements appear as an equirectangular projection like this (This is helpful because we can use different projections to view a round world on a flat screen in different ways):

![](https://i.imgur.com/hX7hOoB.png)

## Create a rectangular overlay based on coordinates entered into a form

```html
<form>
    Top Left:
    <input type="text" placeholder="latitude"/>
    <input type="text" placeholder="longitude"/>
    <br/>
    Bottom Right:
    <input type="text" placeholder="latitude"/>
    <input type="text" placeholder="longitude"/>

    <input type="submit"/>
</form>
```

```js
d3.select('form').on('submit', (event) => { //set up a submit handler on the form
	event.preventDefault(); // stop the form from submitting

	d3.selectAll('rect').remove(); //remove any previously drawn rectangles

	const lat1 = d3.select('input:first-child').property('value'); //get first latitude input
	const lng1 = d3.select('input:nth-child(2)').property('value'); //get first longitude input
	const location1 = worldProjection([lat1, lng1]); //convert these into pixel coordinates
	const x1 = location1[0]; //reassign for ease of reading
	const y1 = location1[1];

	const lat2 = d3.select('input:nth-child(4)').property('value'); //get second latitude input (the <br/> is :nth-child(3)
	const lng2 = d3.select('input:nth-child(5)').property('value'); //get second longitude input
	const location2 = worldProjection([lat2, lng2]); //convert these into pixel coordinates
	const x2 = location2[0]; //reassign for eas of reading
	const y2 = location2[1];

	d3.select('svg').append('rect') // append a rectangle to the svg
		.attr('fill', '#000') //make it black
		.attr('x', x1) //assign top left x location
		.attr('y', y1) //assign top left y location
		.attr('width', x2-x1) //compute width from bottom right x coordinate
		.attr('height', y2-y1) //compute height from bottom right y coordinate
});
```

## Draw a rectangle on the map and display its map coordinates

## Conclusion

In this section we've covered how to use D3 to create a projection and render GeoJSON data as a map, and we've learned about using different projects to visualize the world.  This can be helpful when displaying populations or perhaps average rainfall of various regions.  Congratulations! You've made it to the end of this book. Now go off and create amazing visualizations.
