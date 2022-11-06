---
layout: post
---

## Background
Purchasing a vehicle has become a daunting, time-consuming task since the height of the COVID-19 pandemic. The semiconductor shortage caused by interruptions in complex supply chains has inhibited the rate of vehicle production worldwide. At the time of writing, US Light Vehicle Sales remain down by over 13% and demand is expected to exceed supply into 2025 according to Haig Partners. Dealerships across the nation added additional markup beyond the Manufacturer Suggested Retail Price (MSRP) to make up for the reduced sales volume at the expense of the consumer. Excessive market adjustment fees combined with mandatory dealer-installed options have plagued the new vehicle market in the US since the beginning of the supply issues.     

Despite it being an awful time to purchase a vehicle, it has become inevitable for many Americans - myself included. Early this year, I embarked on a journey to purchase a new vehicle out of necessity. Since it is a large, important purchase that I plan to keep for a long time, I was looking for something fun, practical, reasonably priced, and something that I wouldn’t outgrow anytime soon. The resulting decision led me to the Hyundai Elantra N.   

![](/assets/20221106-image1.jpg)

## The Fun Begins

Upon visiting the Hyundai USA website, I went to the “[Inventory](https://www.hyundaiusa.com/us/en/inventory-search)” section to see what my options were. However, the website only gives a maximum search radius of 250 miles. Unfortunately, in today’s market, consumers may need to travel in excess of that in order to find a vehicle without unnecessary markups. In order to efficiently discover inventory outside of that search radius, I needed to figure out how the site was getting the information it displays to the user.  

![](/assets/20221106-image2.png)

This is when I first began to examine the web application in greater detail. Given that the search radius was a user-defined variable, I wondered if it is possible to alter its value to exceed the maximum allowed by Hyundai. Using the browser’s built-in developer tools to examine network traffic, I filtered the traffic using one of the likely parameters of 250 and noticed an HTTP GET request to “vehicleList.json” which includes our expected parameters. The response includes a JSON object containing dealerships, and those dealerships contain vehicles. This is how the web application fetches all of the available inventory within the search radius.

![](/assets/20221106-image5.png)

The “vehicleList.json” API appears to take 4 parameters: the user’s postal code, the year/model name of the vehicle, and the search radius. If we want to build a tool that can help us track Elantra N inventory nationwide, we must see if we can modify the search radius value successfully. If they are validating the user input on the server side before processing the request, the API call would be denied and we would be forced to find another solution.

![](/assets/20221106-image3.png)

When we try to directly visit the API, we are met with an HTTP 403 Error indicating access denied. Comparing the request made from the Inventory Search page versus the request made from directly visiting the API, the only major difference I noted between the two is the lack of the “referer” header in the direct visit. This indicates that Hyundai’s API likely uses  this header as a means to detect and prevent unintelligent scraping. I went ahead and implemented this basic request programmatically using the Python Requests library and it worked. The same data that I saw through visiting the website was now saved as a variable that I could control in my Python code. Oh, and I confirmed that adding extra zeros to the end of the radius works. I now had the ability to query the inventory for the entire United States with one HTTP request.   

![](/assets/20221106-image4.png)

## Data Parsing, Storage, and Analysis
Armed with the ability to programmatically fetch information from the server, it was time to make it usable. This meant first analyzing its structure to determine which information is actually important to me so I could determine how to extract it. I used an online JSON [parser](http://json.parser.online.fr/) to help make sense of the data - it’s not quite [JSON:API](https://jsonapi.org/) compliant but it is pretty close. There is a ‘status’ key whose value indicates whether or not the request was successful. There is also a JSON object which contains an array named ‘dealerInfo’ which contains objects for each dealership that has at least one Elantra N allocated to it. Each dealer object contains an array called ‘vehicles’ and each vehicle is represented as a JSON object including data such as the VIN, color, inventory status, and more.         
I decided to use SQLite to provide long-term storage for the data obtained via the API since it is lightweight and simple to integrate into a Python project. The database contains two tables - one for storing dealership information using the dealer code as the primary key, and one for storing each individual vehicle using the VIN as the primary key. 
Storing the data allows us to analyze historic trends over the course of the vehicle’s production that we would normally not be able to access. For example, by looking at the last five digits of each VIN, I know that there have been about 11k units produced at the time of writing. Because I have been storing this data in the SQLite database, I know that at least 2k of those were destined for the US. That said, this number is a lower bound since this method is only to track the subset of US-bound inventory which has been listed on the website. 

I was also able to determine that about 74% of sampled US-bound Elantra N vehicles use automatic (DCT) transmissions, while the other 26% are equipped with a manual. Initially, this started off closer to 66-33% split but the DCT option gained a larger share of the market. Also, the distribution of vehicles by color is as follows: Phantom Black (27%), Ceramic White (22%), Intense Blue (18%), Cyber Gray (18%), and Performance Blue (14%). Anecdotally speaking, the colors with the lowest production numbers seem to be the most sought after by far so it is interesting to see that they are the rarest. The distribution of vehicles per state somewhat follows the order of states by population, but the vehicle per capita ratio does not align as well with California, Texas, and Florida in the lead with 190, 182, and 132 units respectively.  

## Beating the Odds
Purchasing a new car in these market conditions and in a rare configuration (Performance Blue paint, manual transmission) would be no easy task. However, armed with up-to-date information of the market and some handy alerting which I implemented, I was able to defy the odds and find the exact configuration I wanted just a few months after US deliveries had begun. And I’ve been loving it ever since. 

![](/assets/20221106-image6.jpg)
