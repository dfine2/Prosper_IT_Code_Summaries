# C# Live Project
## Introduction
After learning the basics C# and ASP.NET with the Tech Academy, I had the opportunity to work with a team of other students to design an Management Portal for Erectors, Inc., a real construction company.  Using Entity Framework Code First, we built a fully operational MVC application to manage employees, work shifts, jobs, company news, and other important operations.  The two-week sprint began relatively late into the application's development, and I spent most of the first week adding functionality and finishing touches to a nearly deliverable-ready project. During the second week, the team and I began on a fresh version of the project, giving me hands-on experience with Code First and the early stages of MVC development.
## Week One
### Directions to the Worksite
One of the many functions of the Management Portal was to display a list of scheduled construction jobs for various customers.  While the app already provided addresses and maps for each of these jobs, it did not provide travel instructions. My task was to provide the user step by step navigation from their current location to the job site. First, I wrote a function to get the user's current location

The app already provided a Leaflet map with a marker for the jobsite, so Leaflet's Routing API offered the best solution for the task at hand. 
