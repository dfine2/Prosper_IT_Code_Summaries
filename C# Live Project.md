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

### Data Filter Row
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

### Chat Model
Now working on an entirely new iteration of the Management Portal, one of the team's early tasks was to create a way for users to send and receive chat messages. My task was to lay the groundwork for this feature by building a Chat C# class with Entity Framework Code First.
```C#
namespace ManagementPortal.Models
{
    public class ChatMessage
    {
        [Display(Name = "Id")]
        public Guid ChatMessageId { get; set; }
        [Display(Name = "Sender")]
        public string Sender { get; set; }
        [Display(Name = "Date")]
        public DateTime Date { get; set; }
        [Display(Name = "Message")]
        public string Message { get; set; }
    }
}
```

I then scaffolded an Entity Framework Controller to manage CRUD operations and views for the new class, so that a user's chats would be stored in the apps SQL database.

### Company News
The team had recently added a CompanyNews C# model and controller to allow users to read and post company-wide notices. I was charged with both creating seeding the database with dummy events, and with adding a partial view to display on the dashboard. I seeded the CompanyNews database by adding following code to the project's Configuration.cs file.
```C#
 var NewsItems = new List<CompanyNews>
            {
                new CompanyNews
                {
                    DateStamp = Convert.ToString(new DateTime(2018, 12, 11).Ticks),
                    Title = "Company Party",
                    NewsItem = "The Company New Year's Eve party will be held at Jack's house this year. Contact Jack for directions.See you there!",
                    ExpirationDate = new DateTime(2018, 1, 3)
                },
                new CompanyNews
                {
                    DateStamp = Convert.ToString(new DateTime(2019, 3, 9).Ticks),
                    Title = "Kittens for Adoption",
                    NewsItem = "Three beautiful black kittens for adoption. Just 3 weeks old. Contact Jill for more information",
                    ExpirationDate = new DateTime(2019, 1, 3)
                },
                new CompanyNews
                {
                    DateStamp = Convert.ToString(new DateTime(2019, 6, 26).Ticks),
                    Title = "New Team Member",
                    NewsItem = "Joel joins us today as a new team member. Be sure to welcome him!",
                    ExpirationDate = new DateTime(2019, 7, 22)
                },
                new CompanyNews
                {
                    DateStamp = Convert.ToString(new DateTime(2019, 7, 2).Ticks),
                    Title = "Fourth of July",
                    NewsItem = "The office will be closed from Thursday, July 4 to Friday, July 5 for the holiday.",
                    ExpirationDate = new DateTime(2019, 7, 6)
                },
                  new CompanyNews
                {
                    DateStamp = Convert.ToString(new DateTime(2019, 9, 5).Ticks),
                    Title = "Clothing Drive",
                    NewsItem = "We are collecting donations for our annual clothing drive.Please contact Joe for more information.",
                    ExpirationDate = new DateTime(2019, 10, 2)
                }
            };
            NewsItems.ForEach(x => context.CompanyNews.AddOrUpdate(n => n.DateStamp, x));
            context.SaveChanges();
        }
```
I proceeded to write a partial view for the dashboard, using the entries in the CompanyNews database table as my model.

```razor
	@using ManagementPortal.Models
	@model IEnumberable<CompanyNews>
	<div class="container" id="CompanyNewsPartial">
		<div class="row" id="NewsPartialHeader">
			<h3 class="text-center">Company News</h3>
			<a href="#createModal" data-toggle="modal">Create New</a>
		</div>
		<div class="row">
			@foreach(var story in Model)
			{
				<div class="col">
					<div class="card newsCard">
						<div class="card-body">
							<h5 class="card-title">@story.Title</h5>
							<hr/>
							<p class="card-subtitle">@story.DateStamp</p>
							<br/>
							<p class="card-text">@story.NewsItem</p>
						</div>
					</div>
				</div>
			}
		</div>
	</div>
```
Coupled with some CSS, this code produced the following result:
![New View](https://github.com/dfine2/code_summaries/blob/master/img/newsview.PNG?raw=true)

