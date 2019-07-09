# C# Live Project
## Introduction
After learning the basics C# and ASP.NET with the Tech Academy, I had the opportunity to work with a team of other students to design an Management Portal for Erectors, Inc., a real construction company.  Using Entity Framework Code First, we built a fully operational MVC application to manage employees, work shifts, jobs, company news, and other important operations.  The two-week sprint began relatively late into the application's development, and I spent most of the first week adding functionality and finishing touches to a nearly deliverable-ready project. During the second week, the team and I began on a fresh version of the project, giving me hands-on experience with Code First and the early stages of MVC development.
## Week One
### Directions to the Worksite
One of the many functions of the Management Portal was to display a list of scheduled construction jobs for various customers.  While the app already provided addresses and maps for each of these jobs, it did not provide travel instructions. My task was to provide the user step by step navigation from their current location to the job site. First, I wrote a JavaScript function to get the user's current location.

```javascript
	//Get user's current location to provide directions to the job site
	function getLocation() {
		if (navigator.geolocation) {
			navigator.geolocation.getCurrentPosition(initMap);  //returns the user's current position as a parameter to the pre-existing initMap function.
		} else {
			alert("The browser you are using does not support Geolocation.")
		}
	}
```
I had getLocation() feed a parameter into the pre-existing function responsible for creating the a Leaflet map (initMap()) I then wrote code within the initMap() function to render the Leaflet Router GUI, taking the user's current position as the starting point and the address of the job site as the destination. For context, I've provided the entire initMap(), although my primary contribution was the Routing Machine section.

```javascript

    // Render the map with proper location and a destination marker, takes the user's current position as a paramter from getLocation
	function initMap(position) {
		// Set the address from the database and intialize the starting map view/zoom level
		var address = '@Model.StreetAddress @Model.State @Model.Zipcode';  //C# Razor code referencing a "Job" model object
		var map = L.map('map').setView([0, 0], 14);
		L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
			attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
		}).addTo(map);

		//// Convert the address into coordinates and map it with a marker
		geocoder = new L.Control.Geocoder.Nominatim();
		geocoder.geocode(address, function (results) {
			latLng = new L.LatLng(results[0].center.lat, results[0].center.lng);
			marker = new L.Marker(latLng);
			map.setView(latLng);
			marker.addTo(map);
			//Get directions with Leaflet Routing Machine (This section is my primary contribution to the initMap() function)
			L.Routing.control({
				waypoints: [
					L.latLng(position.coords.latitude, position.coords.longitude),
					latLng
				],
				routeWhileDragging: true,
				geocoder: L.Control.Geocoder.nominatim()
			}).addTo(map);
		});
		L.Control.geocoder().addTo(map); // Add a search bar for the user to manually find a location
```


