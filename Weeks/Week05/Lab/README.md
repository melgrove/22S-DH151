# Week 5 Lab

<img src="images/jhcovid.png">

Welcome to week 5! As we gear ourselves to the midterm presentations, the purpose of this lab is to demonstrate the development of a project from start to finish. This lab will create a web map that shows the global growth of the covid 19 pandemic from start (being around January 2020) to present day. The data is brought in via a real-time feed from the [Johns Hopkins University Covid19 Data Repository](https://github.com/CSSEGISandData/COVID-19) on GitHub.

## Getting started

### Starter files

- starter files available [here](starter)

Or, you can create the files as indicated here:

### `Week5/index.html`

Notice the updated title and data source. Papaparse is also included in the head using a [cdn](https://cdnjs.com/libraries/PapaParse) link.

```html
<!DOCTYPE html>
<html>
<head>
	<title>World Covid Map</title>
	<meta charset="utf-8" />

	<!-- style sheets -->
	<link rel="stylesheet" href="css/style.css">

	<!-- leaflet -->
	<link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
	<script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>

	<!-- jquery -->
	<script src="https://code.jquery.com/jquery-3.6.0.min.js" integrity="sha256-/xUj+3OJU5yExlq6GSYGSHk7tPXikynS7ogEvDej/m4=" crossorigin="anonymous"></script>

	<!-- papaparse for csv data -->
	<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>

</head>
<body>

	<div class="header">
		Covid Confirmed Cases by Country
	</div>
	<div class="sidebar">
		
	</div>
	<div class="content">
		<div id="map"></div>
	</div>
	<div class="footer">
		Data source: <a href="https://github.com/CSSEGISandData/COVID-19">COVID-19 Data Repository by the Center for Systems Science and Engineering (CSSE) at Johns Hopkins University</a>
	</div>

	<script src="js/map.js"></script>

</body>
</html>
```

### `Week5/js/map.js`
Notice the updated global variable list, and the `readCSV()` function from the previous lab.

```js
// Global variables
let map;
let lat = 0;
let lon = 0;
let zl = 3;
let path = '';
let markers = L.featureGroup();

// initialize
$( document ).ready(function() {
    createMap(lat,lon,zl);
	readCSV(path);
});

// create the map
function createMap(lat,lon,zl){
	map = L.map('map').setView([lat,lon], zl);

	L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
		attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
	}).addTo(map);
}

// function to read csv data
function readCSV(path){
	Papa.parse(path, {
		header: true,
		download: true,
		complete: function(csvdata) {
			console.log(csvdata);
			
		}
	});
}
```

### `Week5/css/style.css`
Notice the adjusted grid-termplate values.

```css
body,html {
	margin:0;
	height:100%;
	width:100%;
}

#map {
	height: 100%;
}

body {
	display: grid;
	grid-template-rows: 60px 1fr 40px;
	grid-template-columns: 300px 1fr;
	grid-template-areas: 
	"header header"
	"sidebar content"
	"footer footer";
}

.header {
	grid-area: header;
	padding:10px;
	background-color: #333;
	color: white;
	font-size: 2em;
}

.sidebar {
	color: white;
	grid-area: sidebar;
	padding:10px;
	background-color: #555;
	overflow: auto;
}

.content {
	grid-area: content;
}

.footer {
	grid-area: footer;
	padding:10px;
	background-color: rgb(175, 175, 175);
}
```

Confirm that you have your "starter" code ready to go by activating live server and seeing the starter map on your browser window. 

<img src="images/starter.png">

## Mapping Covid CSV data

Take a look at the csv data as hosted by John's Hopkins:

- [csv data on github](https://github.com/CSSEGISandData/COVID-19/blob/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv)

Rather than downloading the data to use, we will rely on a direct feed from the github repository. To do so, click on the "raw" button to access the csv data in raw format.

- [raw csv data link](https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv)

We can use the URL link, rather than a relative link to a local file, to bring in the data. Consider why this is useful, and in what situations this may not be a good idea.

### Updating the path variable

Update the global variable for `path` to indicate the path to the covid csv data.

```js
let path = 'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv';
```

### Store the data as a global variable

Global variables are variables that are accessible anywhere in your code. If you create a variable inside a function, it will only be accessible within that function. In order to "globalize" our data, let's first create a global variable as a placeholder for it.

```js
let csvdata;
```

Next, within the function to read the csv data, assign the data to the `csvdata` global variable. Additionally, create the call to the `mapCSV()` function.

```js
// function to read csv data
function readCSV(path){
	Papa.parse(path, {
		header: true,
		download: true,
		complete: function(data) {
			console.log(data);
			// put the data in a global variable
			csvdata = data;

			// call the mapCSV function to map the data
			mapCSV();
		}
	});
}
```

### Inspecting the browser console

Save the `map.js` file, and view the console output in your browser window.

<kbd><img src="images/console.png"></kbd>

Inspect on how the data is structured. What does each row represent? Are there locational attributes? How is the data for each day represented in the data?


### :octocat: **Challenge Exercise:** 
Begin to construct the function to map the data. Add the function for `mapCSV(csvdata)` as shown below, and fill in the blanks to map the countries.

*NOTE: The data from Johns Hopkins may include null or empty values, which in javascript shows up as `undefined`. In order to check whether a variable exists (such as the `item.Lat` field), add an `if` statement prior to creating a marker. The `if` statement is provided below.*

```js
function mapCSV(){

	// loop through every row in the csv data
	csvdata.data.forEach(function(item,index){
		// check to make sure the Latitude column exists
		if(item.Lat != undefined){

			// Lat exists, so create a circleMarker for each country


			// add the circleMarker to the featuregroup

		} // end if
	})

	// add the featuregroup to the map


	// fit the circleMarkers to the map view

}
```

<img src="images/covidcircles.png">

Answer:
```js
function mapCSV(){

	// loop through every row in the csv data
	csvdata.data.forEach(function(item,index){
		// check to make sure the Latitude column exists
		if(item.Lat != undefined){

			// Lat exists, so create a circleMarker for each country
            let marker = L.circleMarker([item.Lat,item.Long])

			// add the circleMarker to the featuregroup
            markers.addLayer(marker)

		} // end if
	})

	// add the featuregroup to the map
    markers.addTo(map)


	// fit the circleMarkers to the map view
    map.fitBounds(markers.getBounds())
}
```


### Inspecting the metadata

Nice map! We now have a visual representation of every country in the covid data on the map. Let's investigate on how we are going to map the cases per country.

Papaparse stores metadata of the csv file in its object under the variable `meta`. 

Return to your browsers live view, and open the developer's console. At the prompt, type in the following commands:

What are the fields?
```js
csvdata.meta.fields
```

How many fields are there?

```js
csvdata.meta.fields.length
```

To get the last fieldname, i.e. the last date in the data:

```js
ccsvdata.meta.fields[csvdata.meta.fields.length-1]
```
Notice the `-1` at the end. Why is this necessary? 

Now that we know how to get the last date from the csv headers, we can go back to our javascript code, and assign a variable for it.

First, create a global variable for `lastdate`:

```js
let lastdate;
```

Next, assign the last date to the global variable. At the same time, create the call to a new function `mapCSV(lastdate)` that will create the map with the data for the last date.

```js
// function to read csv data
function readCSV(){
	Papa.parse(path, {
		header: true,
		download: true,
		complete: function(data) {
			console.log(data);
			// put the data in a global variable
			csvdata = data;

			// get the last date and put it in a global variable
			lastdate = ccsvdata.meta.fields[csvdata.meta.fields.length-1];

			// map the data for the given date
			mapCSV(lastdate);
		}
	});
}
```

### Varying circle size by value

Consider that the circle size can be differentiated to visualize the number of cases per country. 

However, converting case counts by country to circles on a computer screen is challenging. Consider that at the time of this writing (4/25/2022), the single highest case count by country is in the United States at 80,984,914 cases (it was 31,929,351 a year ago). Imagine if we decided to create circles by case counts, the United States would have a radius of 80 million pixels! For this reason, we need to scale our the case counts to a pixel range that makes sense.

A simple solution would be divide the case counts by a large enough number, that the highest total would equal to roughly 100 pixels, so divide by 800,000.

Additionally, the `mapCSV()` function no longer needs to be fed data because `csvdata` is now a global variable that we can access. Instead, have `mapCSV()` require a date in order to generate a map of differential sized circles for *any given date*.

```js
// map function now requires a date
function mapCSV(date){

	// clear layers in case you are calling this function more than once
	markers.clearLayers();

	// loop through each entry
	csvdata.data.forEach(function(item,index){
		if(item.Lat != undefined){
			// circle options
			let circleOptions = {
				radius: item[date]/800000,　// divide by high number to get usable circle sizes
				weight: 1,
				color: 'white',
				fillColor: 'red',
				fillOpacity: 0.5
			}
			let marker = L.circleMarker([item.Lat,item.Long],circleOptions)
			.on('mouseover',function(){
				this.bindPopup(`${item['Country/Region']}<br>Total confirmed cases as of ${date}: ${item[date]}`).openPopup()
			}) // show data on hover
			markers.addLayer(marker)	
		}   
	});

	markers.addTo(map)
	map.fitBounds(markers.getBounds())

}
```
<img src="images/covidmap.png">

However, imagine if the case counts continue to grow (knock on wood that it does not). What will happen to our map? Eventually, the size of the circles may overwhelm the map (again, knock on wood that this does not happen).

Instead, why not create a function that *ensures* that no matter what the case count, the largest circle radius will *always* be 100 pixels?

In the revised `mapCSV` function below, instead of a straight division of 800,000, the radius option calls a function `getRadiusSize(item[date])`. Update your `mapCSV` function as shown below, and then create the new function `radiusSize(date)` that returns the correct radius size, scaled so that the maximum

```js
function mapCSV(date){

	// clear layers in case you are calling this function more than once
	markers.clearLayers();

	// loop through each entry
	csvdata.data.forEach(function(item,index){
		if(item.Lat != undefined){
			// circle options
			let circleOptions = {
				radius: getRadiusSize(item[date]),　// call a function to determine radius size
				weight: 1,
				color: 'white',
				fillColor: 'red',
				fillOpacity: 0.5
			}
			let marker = L.circleMarker([item.Lat,item.Long],circleOptions)
			.on('mouseover',function(){
				this.bindPopup(`${item['Country/Region']}<br>Total confirmed cases as of ${date}: ${item[date]}`).openPopup()
			}) // show data on hover
			markers.addLayer(marker)	
		}   
	});

	markers.addTo(map)
	map.fitBounds(markers.getBounds())

}
```
### :octocat: **Challenge Exercise 2:** 

Complete the new function to get radius size:

```js
function getRadiusSize(value){

	// create empty array to store data
	let values = [];

	// add case counts for most recent date to the array


	// get the max case count for most recent date


	// per pixel if 100 pixel is the max range


	// return the pixel size for given value

}
```

*Cheatsheet*: Take a peek at the answer [here](completed/js/map.js).

### Sidebar 

The final part of this lab is to populate the sidebar. Create yet another function `createSidebarButtons()` that populates the sidebar with buttons or text links that update the map in meaningful ways.

Add the call to the function in the `readCSV` function. Your function should look like this:

```js
// function to read csv data
function readCSV(){
	Papa.parse(path, {
		header: true,
		download: true,
		complete: function(data) {
			console.log(data);
			// put the data in a global variable
			csvdata = data;

			// get the last date
			lastdate = csvdata.meta.fields[csvdata.meta.fields.length-1];
			
			// map the data
			mapCSV(lastdate);

			// create sidebar buttons
			createSidebarButtons();

		}
	});
}
```

Example sidebar function:

```js
function createSidebarButtons(){

	// put all available dates into an array
	// using slice to remove first 4 columns which are not dates
	let dates = csvdata.meta.fields.slice(4)

	// loop through each date and create a hover-able button
	dates.forEach(function(item,index){
		$('.sidebar').append(`<div onmouseover="mapCSV('${item}')" class="sidebar-item" title="${item}">${item}</div>`)
	})
}
```

And add the stylesheet class for `sidebar-item` to the `style.css` file:

```css

.sidebar-item {	/* border: 1px solid gray; */
	padding: 2px;	/* background-color: #333; */
	float:left;
	cursor:pointer;
	color: rgb(255, 79, 79);
	font-size: 1em;
}
.sidebar-item:hover {
	background-color: rgb(255, 0, 0);
}
```
<img src="images/covidfinal.png">

You can see the final version [here](https://yohman.github.io/21S-DH151/Weeks/Week05/Lab/completed) 

### :octocat: **Challenge Exercise:** 

Add a display of the date that is being hovered over somewhere on the page. You may choose to add it in the header, sidebar, or on top of the map itself. 

Hint:
- create a `<div>` with a class or id that will hold the date display
- reference the `div` using jquery and add the date to it as users hover over the red circles (you can add this to the `mapCSV` function)

# Other ways of importing data

## From an existing API

<img src="images/artapi.png">

Open data portals provide endpoints to their data, typically in json format. For smaller datasets, you can directly access their data if you have the endpoint URL. Consider the endpoint for the [Public Art dataset](https://controllerdata.lacity.org/Audits-and-Reports/Public-Art/ejf8-ekfc/data) from the LA Controller's data portal. The URL endpoint is:

```
https://controllerdata.lacity.org/resource/ejf8-ekfc.json
```

Once you have the endpoint to a json file, you can import the data into your javascript project with the following `fetch()` command:

```js

let url = "https://controllerdata.lacity.org/resource/ejf8-ekfc.json"

function getJSON(url){
fetch(url)
	.then(response => {
		return response.json();
	})
	.then(data =>{
		console.log(data)
	}
}
```

Inspect the console:

<kbd><img src="images/artjson.png"></kbd>

Note that every API endpoint will look different. For this public art data, the endpoint itself is an array of objects. Therefore, you can loop through it as-is:

```js
function mapJSON(data){
	// loop through each row in the json data
	data.forEach(function(item,index){
		// create a marker
		let marker = L.circleMarker([item.latitude,item.longitude])

		// add marker to featuregroup
		markers.addLayer(marker);
	})

	// add featuregroup to map
	markers.addTo(map);

	// zoom out to the extent of featuregroup
	map.fitBounds(markers.getBounds());
}
```

## Google Sheets

Check the [following google sheet](https://docs.google.com/spreadsheets/d/1wk6ylUHjbJDatNBeTZCFtpISVVwgxy9QEGplVVuh5_M/edit?usp=sharing) of the Dunitz data as an example.

If your data is in a Google Sheet, you can import it directly (and in real time) to your javascript project.

1. Open a google sheet document
1. Go to File, Publish
1. In the first drop down, choose the sheet you want to publish
1. In the second drop down, choose `Comma-separated values (.csv)`
1. Publish the sheet to the web

	<img src="images/gpublish.png">

1. Copy the URL

	<img src="images/gpublish2.png">

1. You can use the URL as a csv file feed using the same Papaparse code:


```js
let path = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQtXb-BG5Ee-AB8S8xjgEsLuEIoUGyvgtqrVsojnYkFePHy-VICUMkp9R16FHuPTv0uaRwHM29wbRxx/pub?gid=1347161303&single=true&output=csv"

// function to read csv data
function readCSV(path){
	Papa.parse(path, {
		header: true,
		download: true,
		complete: function(csvdata) {
			console.log(csvdata);
			
		}
	});
}
```

The console reveals the data in the same format as a saved csv file. Happy mapping!

<img src="images/gpublish3.png">


