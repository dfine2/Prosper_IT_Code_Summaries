# C# Live Project
## Introduction
After learning the basics C# and ASP.NET with the Tech Academy, I had the opportunity to work with a team of other students to design a Management Portal for Erectors, Inc., a real construction company.  Using Entity Framework Code First, we built a fully operational MVC application to manage employees, work shifts, jobs, company news, and other important operations.  The two-week sprint began relatively late into the application's development, and I spent most of the first week adding functionality and finishing touches to a nearly deliverable-ready project. During the second week, the team and I began on a fresh version of the project, giving me hands-on experience with Code First and the early stages of MVC development. After a brief interruption for some Python work, I returned to this project for another two-week sprint.

## First Sprint

### Week One
#### Directions to the Job Site
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

#### Data Filter Row
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

### Week Two

#### Chat Model
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

#### Company News
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

<!-- Create Pop-up -->

<div class="modal fade" role="dialog" id="createModal">
	<div class="modal-dialog modal-dialog-centered">
		<div class="modal-content">
			<div class="modal-header">
				<button type="button" class="close" data-dismiss="modal">%times;</button>
				<h4 class="modal-title">Create an News Story</h4>
			</div>
			<div class="modal-body">
				@Html.Action("_CreateCompanyNews","Home")
			</div>
		</div>
	</div>
</div>

```
Coupled with some CSS, this code produced the following result:
![New View](https://github.com/dfine2/code_summaries/blob/master/img/newsview.PNG?raw=true)

## Second Sprint

### Custom DateTime Validation

One of the Management Portal's core features is the ability to schedule users to work specific shifts on a job. Upon beginning my second sprint with Management Portal, my first task was to emend this functionality to prevent users from creating new shifts for past dates. To that end, I wrote a custom attribute to add more complex validation logic to the schedule model.

```C#
using System;
using System.ComponentModel.DataAnnotations;

namespace ManagementPortal.Validation
{
    public class DateRangeAttribute : ValidationAttribute
    {
        public DateRangeAttribute(string errorMessage) : base(errorMessage) { }

        public override bool IsValid(object value)
        {
            if (value != null)
            {
                if (value as string == "")
                {
                    return true;
                }
                else
                {
                    DateTime? date = value as DateTime?;
                    if (date >= DateTime.Now)
                    {
                        return true;
                    }
                    return false;
                }
               

            }
            return true;
	    
        }
    }
}
```

And then to the Schedule class I added:

```C#
[DateRange("Cannot enter a date earlier than today.")]
public DateTime? EndDate { get; set; }
```
This method worked but for a strange validation error that declared invalid any day after the 12th of each month. I struggled to find the source of this error until I looked at the actual SQL tables and saw that date and month were being switched upon submission to the database, so, for example, 7/25/2019 became 25/7/2019. After determining that the problem lay in translating the incoming data to SQL, I needed to find a way to intercept and manipulate the data sent from the view before it reached the database. Because I was using EditorFor HTML helpers to send view data to the controller, I didn't have direct access to the pre-packaged code handling the controller logic. I therefore created a custom ModelBinding for the DateTime and DateTime? data types, which could then be applied across the entire Management Portal project.

```C#
using System;
using System.Web.Mvc;
using System.Globalization;

namespace ManagementPortal.Validation
{
    public class DatetimeModelBinder : DefaultModelBinder
    {
        private string[] _dateFormats;

        public DatetimeModelBinder(string[] dateFormats)
        {
            _dateFormats = dateFormats;
        }

        public override object BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)
        {
            DateTime dateValue;
            var value = bindingContext.ValueProvider.GetValue(bindingContext.ModelName);
            if (value.AttemptedValue != "")
            {
                if (DateTime.TryParseExact(value.AttemptedValue, _dateFormats, CultureInfo.InvariantCulture, DateTimeStyles.None, out dateValue))
                {
                    return dateValue;
                }
                else
                {
                    throw new Exception(value.AttemptedValue + " is not a valid date");
                }
            }
            else return null;
        }
    }
}
```

And then in Global.asax in Application Start:

```C#
 // DateTime ModelBinder
            var formatStrings = new string[] { "MM/dd/yyyy hh:mm tt", "MM/dd/yyyy", "MM/dd/yyyy hh:mm", "" };
            var datetimebinder = new DatetimeModelBinder(formatStrings);
            ModelBinders.Binders.Add(typeof(DateTime), datetimebinder);
            ModelBinders.Binders.Add(typeof(DateTime?), datetimebinder);
```

This logic set a list of acceptable formats for DateTime input, and ensured that these formats were not changed upon submission to the database.

### New User Registration and Account Management

Potential new Management Portal users begin by requesting an account from an administrator. The administrator then generates a confirmation code for a new username, and sends it back to the requester.  Only then can a new user register.  Most of this logic already existed when I started on this assignment, but registration was encountering validation errors.  I discovered that these errors stemmed from required User model attributes that were left out of the associated view model, and once I added these, registration began working.  I proceeded to link the views required by the registration process to appear in the necessary order, and to limit which of these views and methods non-admins could access.  Next, I added logic to allow login with both email address and username.

```C#
 string username;
            if (model.UserName.Contains("@"))
            {
                var user = await UserManager.FindByEmailAsync(model.UserName);
                if (user == null)
                {
                    return View(model);
                }
                else
                {
                    username = user.UserName;
                }
            }
            else
            {
                username = model.UserName;
            }
```

I also wrote code giving users the ability to edit their own account information where before they could only change their passwords.

```C#
        //GET: /Manage/EditAccountInfo
        public ActionResult EditAccountInfo()
        {
    
            return View();
        }

        
        //POST: /Manage/EditAccountInfo
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult EditAccountInfo(EditAccountInfoViewModel model)
        {
            var manager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(new ApplicationDbContext()));
            var ASPcurrentUser = manager.FindById(User.Identity.GetUserId());
            string ASPcurrentUserId = ASPcurrentUser.Id;

            ApplicationUser currentUser = db.Users.Find(ASPcurrentUserId);

            currentUser.UserName = model.UserName;
            currentUser.DisplayName = model.DisplayName;
            currentUser.Email = model.Email;
            currentUser.FirstName = model.FirstName;
            currentUser.LastName = model.LastName;
            db.SaveChanges();

            return View(model);
        }
```


### CRUD Drop-down Menus

A number of our database models were interconnected by foreign keys, which needed to be accounted for in creation and editing. An instance of "Job", for example, contains an instance of "ApplicationUser" for a manager property, and an instance of "JobSite" for a location property.  I was tasked with adding drop-down lists for these navigation properties to the create and edit views for the Job class. The following function encapsulates creation of the actual drop-downs:

```C#

        private void PopulateJobDropDowns(object model, int JobId = 1, string UserId = null)
        {
            var jobSites = new SelectList(db.JobSites.ToList(), "JobSiteID", "SiteName", JobId);
            ViewData["JobSites"] = jobSites;

           SelectList users;
            if (UserId == null)
            {
                users = new SelectList(db.Users.ToList(), "Id", "FullName");
            }
            else
            {
                users = new SelectList(db.Users.ToList(), "Id", "FullName", UserId);
            }
            ViewData["Users"] = users;
        }
```
The conditional logic determines if the model already has a value for the relevant property, and if so, preloads the drop-down list with that value. Then, on the view:

```razor
<div class="form-group">
	@Html.LabelFor(model => model.Location, htmlAttributes: new { @class = "control-label col-md-2" })
	<div class="col-md-10">
		@Html.DropDownList("LocationSelector", (IEnumerable<SelectListItem>)ViewData["JobSites"], new { @class = "form-control" })
		@Html.ValidationMessageFor(model => model.Location, "", new { @class = "text-danger" })
	</div>
</div>

<div class="form-group">
	@Html.LabelFor(model => model.Manager, htmlAttributes: new { @class = "control-label col-md-2" })
	<div class="col-md-10">
		@Html.DropDownList("ManagerSelector", (IEnumerable<SelectListItem>)ViewData["Users"], new { @class = "form-control" })
		@Html.ValidationMessageFor(model => model.Manager, "", new { @class = "text-danger" })
	</div>
</div>
		
```
And on the controller:

```C#
        // POST: Jobs/Create
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "JobIb,JobTitle,JobType,Active,Location,Manager, WeeklyShifts")] Job job)
        {

            PopulateJobDropDowns(job);
            var LocationId = Request.Form["LocationSelector"].ToString();
            var ManagerId = Request.Form["ManagerSelector"].ToString();

            var weeklyShifts = new ShiftTime { Job = job };
            job.WeeklyShifts = weeklyShifts;
            
            if (ModelState.IsValid)
            {
                job.Location = db.JobSites.Find(Int32.Parse(LocationId));
                job.Manager = db.Users.Find(ManagerId);
                db.ShiftTime.Add(job.WeeklyShifts);
                db.Jobs.Add(job);
                db.SaveChanges();
                return RedirectToAction("Index");
            }
            return View(job);
        }
```
I then used the same logic to implement similar navigation property drop-downs for the Schedule class.

