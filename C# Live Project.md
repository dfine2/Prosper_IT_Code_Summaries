# C# Live Project
## Introduction
After learning the basics C# and ASP.NET with the Tech Academy, I had the opportunity to work with a team of other students to design a Management Portal for Erectors, Inc., a real construction company.  Using Entity Framework Code First, we built a fully operational MVC application to manage employees, work shifts, jobs, company news, and other important operations.  The two-week sprint began relatively late into the application's development, and I spent most of the first week adding functionality and finishing touches to a nearly deliverable-ready project. During the second week, the team and I began on a fresh version of the project, giving me hands-on experience with Code First and the early stages of MVC development.
## Week One
### Directions to the Job Site
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

The final result looks like this, though ideally the user would be closer to the job site than I was.

![Leaflet Routing](https://github.com/dfine2/code_summaries/blob/master/img/directions(small).PNG?raw=true)

## Data Filter Row
The Management Portal's dashboard contained a Razor partial view to display and edit a table of site users.  My job was to add a filter row to this table so that users could limit the display to only rows matching desired criteria. I began by adding a new row to the table to hold the filter fields.
```razor
<tr>
			<td>
				@Html.TextBox("UsernameFilter", null, new { @list = "Usernames", @class = "filterBox", @onkeyup = "filter()"})
				<datalist id="Usernames">
					@foreach (var Item in Model)
					{
						<option>@Item.UserName</option>
					}
				</datalist>
			</td>
			<td>
				@Html.TextBox("FNameFilter", null, new { @list = "FNames", @class="filterBox", @onkeyup = "filter()"})
				<datalist id="FNames">
					@foreach (var Item in Model)
					{
						<option>@Item.FName</option>
					}
				</datalist>
			</td>
			<td>
				@Html.TextBox("LNameFilter", null, new { @list = "LNames", @class = "filterBox", @onkeyup ="filter()"})
				<datalist id="LNames">
					@foreach (var Item in Model)
					{
						<option>@Item.LName</option>
					}
				</datalist>
			</td>
			<td>
				@Html.TextBox("Category", null, new { @list = "Categories", @class = "filterBox", @onkeyup="filter()" })
				<datalist id="Categories">
					@foreach (var Item in Model)
					{
						<option>@Item.WorkType</option>
					}
				</datalist>
			</td>
			<td>
				@Html.TextBox("Role", null, new { @list = "Roles", @class = "filterBox", @onkeyup="filter()" })
				<datalist id="Roles">
					@foreach (var Item in Model)
					{
						<option>@Item.UserRole</option>
					}
				</datalist>
			</td>

		</tr>
```
This code both created a row of empty text fields and bound them to properties of the model class(User). I then wrote the filter() JavaScript function, which I bound to the onkeyup event of each text field.

```JavaScript
function filter() {
		var table = document.getElementById("userList").getElementsByTagName("tr");
		//Get filter values in a list
		var filterRow = table[1].getElementsByTagName("td");
		var elements = [];
		for (var i = 0; i < filterRow.length; i++) {
			element = filterRow[i].firstElementChild;
			elements.push(element.value);
		}
		//Iterate over rows
		for (var i = 2; i < table.length; i++) {
			var hits = []  //A list of booleans, if any are false, hide the row.
			var data = table[i].getElementsByTagName("td");
			for (var x = 0; x < elements.length; x++) {
				if (data[x].getElementsByTagName("select").length > 0) {  //Filtering for drop-down list categories
					var list = data[x].innerText.split("\n");
					var content = list[0];
				}
				else {
					var content = data[x].innerText;
				}
				var input = elements[x].toLowerCase();
				var hit = content.toLowerCase().includes(input);
				hits.push(hit);
				}
				if (hits.includes(false)) {
					$(table[i]).hide();
				}
				else {
					$(table[i]).show();
				}
			}
		} 
```
The filter() function maintains a list of the values in the filter row and checks them against the values in the corresponding columns. If any column does not include the text in its filter field, then that row is hidden.

![User Filter](https://github.com/dfine2/code_summaries/blob/master/img/usernamefilter.PNG?raw=true)

## Week Two

