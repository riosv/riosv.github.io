---
layout: post
title: Exploring Toronto with Google Maps and Combinatorial Optimization
---

As the 5th largest city in North America, Toronto is a vibrant and colourful metropolis. Visiting Toronto is exciting but challenging. The city offers thousands of restaurants, hundreds of museums and galleries, and many other attractions. We will be using the google maps API and combinatorial optimization to find the optimal itinerary to visit Toronto's most famous attractions.

<a title="By Rob Sinclair (Toronto by night  Uploaded by Skeezix1000) [CC BY-SA 2.0 (http://creativecommons.org/licenses/by-sa/2.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File%3ANight_skyline_of_Toronto_May_2009.jpg"><img width="800" alt="Night skyline of Toronto May 2009" src="https://upload.wikimedia.org/wikipedia/commons/e/ee/Night_skyline_of_Toronto_May_2009.jpg"/></a>

A good starting point is this [list](http://www.planetware.com/tourist-attractions-/toronto-cdn-on-ont.htm) of the 15 top-rated attractions:

* CN Tower
* Royal Ontario Museum
* Rogers Centre
* Art Gallery of Ontario
* Casa Loma
* Toronto Zoo
* St. Lawrence Market
* Entertainment District
* Ripley's Aquarium of Canada
* City Hall
* Eaton Center
* Distillery District
* High Park
* Ontario Science Centre
* Toronto Islands

Lets say you have five days to visit these attractions. Now the task is to find the itinerary for each day with minimum travel time so you can spend more time enjoying them. This is a difficult task because first you need to find the travel time to each destination and then you need to find the best route among all the combinations. Part of the code in this post is taken from [Randal Olson's](http://www.randalolson.com/2015/03/08/computing-the-optimal-road-trip-across-the-u-s/) article about finding the optimal trip accross the U.S.

## Finding the best five-day itinerary in Toronto

Adding to the list of 15 destinations we also need to consider another point of interest: the place where you will be staying while in the city. For now lets consider this a popular hotel downtown. The hotel will function as a base where you will start and end your journey every day. 

The next step is to find the travel times between all of our 16 destinations. Probably you have used Google Maps before to find directions and travel times between two addresses. We can do the same for all the possible pairs of destinations, however doing this by hand would be a tremendous task. Thankfully with the help of the [Google Maps API](https://developers.google.com/maps/documentation/distance-matrix/intro) we can compute these distances with a short Python script. To make things simpler, we assume that the travel time from point A to point B is the same as travelling from point B to point A. In other words we are dealing with an [undirected graph][undirected-graph]. With this consideration, we only need to calculate 16<sup>2</sup>/2 - 16 = 112 travel times.

We want to know which attractions to visit each day and in what order with the objective of minimizing the total travel time. Several considerations need to be made. Besides travel times we should consider the time we will be spending at each attraction. Some attractions will require more time than other so our algorithm should take that into account when computing our daily itineraries. Speaking of that, we want to make sure we limit the available time each day or we would end up with crazy itineraries that require more hours than the day has. Most attractions open in the late morning and close in the afternoon so in practical terms we have a few hours available each day. I set up this limit to 8 hours per day.

This problem looks very similar to the vehicle routing problem ([VRP](https://en.wikipedia.org/wiki/Vehicle_routing_problem)), a classic in combinatorial optimization. In the vehicle routing problem a set of clients has to be visited by a fleet of vehicles traveling from a common depot. The objective is to find the optimal route for each vehicle. In our case we can assume that we have five vehicles representing our five vacation days. Our depot is our hotel and our clients are our 15 attractions. We also have constraints for our route lengths which makes it a distance-constrained capacitated vehicle problem (DCVRP).

The vehicle routing problem is a generalization of the travelling salesman problem, which is an NP-hard problem. This just means that while there are exact algorithms that can find optimal solutions, their computation time increases polinomially with the size of the problem. Even with modern computers, finding the optimal solution of a routing problem with more than 15 instances will take more than a life time. If we can't find the optimal solution maybe we can find a good-enough solution. We turn to heuristic and approximation algorithms that yield good solutions to large problems in reasonable time.

There are several libraries that implement exact and approximation algorithms for optimization. I use [or-tools](https://developers.google.com/optimization/), Google's suite for combinatorial optimization. Or-tools is open-source and actively maintained. Conveniently it includes a routing library.

Setting up the program is easy with the routing library in or-tools. After setting some parameters and defining our base point `routing = pywrapcp.RoutingModel(len(points), days)` creates the routing model with the number of destinations and days available. `routing.SetCost(dist_callback)` defines the distance costs between points using a callback function to a distance matrix with the travel and leisure times computed previously. `routing.AddDimension(dist_fixed_callback,28800,28800,True,"time")` adds constraints of eight hours per day.

The program is solved with [local search][local-search], a metaheuristic method that iterates through candidate solutions within a neighborhood. A neighborhood is a set of related solutions. For example solutions that differ in only one node in the route could be considered in the same neighborhood. The algorithm iterates until a solution deemed optimal is found or a time bound is reached.

### The optimal five-day itinerary in Toronto

The best itinerary obtained by our script is the following:
 
**Day one** (6.4 hrs) 

1. Royal Ontario Museum
2. CN Tower
3. Art Gallery of Ontario

**Day two** (7.2 hrs)

1. Toronto Zoo
2. Casa Loma
3. St. Lawrence Market

**Day three** (6.6 hrs)

1. Ripley's Aquarium
2. Rogers Centre
3. High Park

**Day four** (4.7 hrs)

1. Toronto Island Park
2. Entertainment District

**Day five** (6.8 hrs)

1. Ontario Science Centre
2. Eaton Centre
3. Distillery District
4. City Hall

Take a look at the map with the routes. You can toggle each day's route in the menu:

<iframe src="https://www.google.com/maps/d/embed?mid=zoY68Movpx3Y.kFtlZRppOPCk" width="640" height="480"></iframe>

### Considerations

We obtained a reasonable solution with at most 7.2 hrs of daily activities. Enough to explore the city during the day and relax in the evening. With some luck you could probably find a better solution. That's ok. Remember our objective was to find a good enough solution quickly. Having said that, there are some other considerations:

* The travel times depend on the time and day you run the script. This is because `gmaps.distance_matrix()` returns the durations at the time it was called. So if you run the script on your own you might get a different distance matrix. 
* Our model assumes all our attractions will be open during our available time. This might not be true for attractions that close during some days or have short opening hours so a final sanity check is a good idea. We can always change the order of our daily itineraries to move around this.
* Our model doesn't explicitly consider times for other activities such as finding parking and eating. These should be part of the time allocated to each attraction.

[undirected-graph]: https://en.wikipedia.org/wiki/Graph_(mathematics)#Undirected_graph
[local-search]: https://en.wikipedia.org/wiki/Local_search_(optimization)

