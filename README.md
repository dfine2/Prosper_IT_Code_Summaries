# Python Live Projects
## Introduction
While studying Python with the Tech Academy, I joined a team of other students on a two-week sprint to roll out two MVT web apps using Django.  The first of these, [DataScrape] (#datascrape), collects pertinent information such as weather, news, and restaurants from across the Web and delivers it to the user in a single compact application. The second app, [TravelScrape] (#travelscrape), operates on the same model but with a narrower scope towards information that would be useful to someone planning a trip, such as flights, hotels, and travel advisories. Not only did I get the opportunity to work on a number of rewarding user stories, but I also got first-hand experience working with project managers and colleagues and familiarizing myself with management software such as Azure DevOps.


## DataScrape

### Flexible Craigslist Searching
The shopping app in our DataScrape project uses BeautifulSoup to collect Craigslist postings in response to user searches.  The first iteration of the app only returned results near Portland, Oregon, and my task was to expand the functionality to allow for flexible location searching. This proved a challenge because Craigslist uses a different website for each of it's locations, each with it's own URL. The first step was to figure out which cities have Craigslist sites, and to then match them to their respective URLs. A little research led me to a Google site with a table of all Craigslist locations, and I scraped this table for city names and URLs using the following code:

CODE SNIPPET

I then adjusted the original search URL, which directed searches to Portland's Craigslist, to instead send requests to the site that matches the user's search.  

CODE SNIPPET


## TravelScrape

### Building the Dashboard App
I was tasked with laying the groundwork of the Dashboard App that serves a the central hub of the TravelScrape Program. The Dashboard provides the interface from which users can access all of their other apps.  This was mostly a front-end project, although I did need to set-up the proper URL configuration and connect the Dashboard links to a "dummy app" until the actual TravelScrape apps were ready for use.

CODE SNIPPET

### Building the Hotels App
I created an app that uses the Amadeus API to find hotel offers near a given city. The app gives users the hotel's name, star-rating, address, cost per night, and if available, a little information blurb.  My biggest challenge was that Amadeus takes input in the form of city aiport IATA codes, and because I knew users were more likely to enter a city name, I had to find a way to match city names to the relevant codes.  I did this by scraping Wikipedia's airport listings and dumping the information into a .csv file that ultimately contained information for 8904 unique airports.  

CODE SNIPPET

I could then iterate through this list to find the entry that matched the user's search. 

I dealt with cases in which the city name on Wikipedia contained accented characters, which I didn't want to interfere with the matching, by creating another column in the .csv with all accents removed. The app could thereby process requests without accent marks, and still return the properly accented city name in the response. I also realized that for cities with multiple airports, my code would only return the alphabetically first match, which might not be the code that Amadeus uses to reference that city. I handled this issue by creating an intermediary screen between the search and the returned results, so that the user can select from a list of cities that match their search.  This solution also addressed any duplicate city names.

CODE SNIPPET

