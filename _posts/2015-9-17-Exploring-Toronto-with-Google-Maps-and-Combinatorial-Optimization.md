---
layout: post
title: Exploring Toronto with Google Maps and Combinatorial Optimization
tags: google-maps, optimization, python
comments: true
---

As the 5th largest city in North America, Toronto is a vibrant and colourful metropolis with many attractions to offer. Visiting as many as possible is a challenging task to any visitor and requires the best use of their time.  In this post we will use the google maps API and combinatorial optimization to find the optimal itinerary to visit Toronto's most famous attractions.

![CN-tower](http://i.imgur.com/K53dCRu.jpg?1 "CN Tower source:https://www.flickr.com/photos/cityoftoronto/9841374213")

A good suggestion of the top attractions in Toronto is this [list](http://www.planetware.com/tourist-attractions-/toronto-cdn-on-ont.htm) from Planet Ware:

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

Lets say we have five days to visit these 15 attractions. If we want to optimize our time we need to find an itinerary with minimum travel time. This is a difficult task because first we need to find the distance to each destination and then we need to find the best route among all of the possible combinations. 

## Finding the best five-day itinerary in Toronto

First we need to add another point of interest: the hotel where we will be staying during our visit. The hotel will function as a start and end point for our daily schedules. 

In order to find the best itinerary we need the travel times between all of our 16 destinations. Google Maps is a great tool for that. We can get travel times for all of the possible pairs of destinations, however doing this by hand would be a tremendous task. Thankfully with the help of the [Google Maps API](https://developers.google.com/maps/documentation/distance-matrix/intro) we can compute these distances with a short Python script. To make things simpler, we assume that the travel time from point A to point B is the same as travelling from point B to point A. In other words we are dealing with an [undirected graph][undirected-graph]. With this consideration, we only need to calculate 16<sup>2</sup>/2 - 16 = 112 travel times. We also assume we are traveling by car - although we can change that.

We should also consider the leisure time at each attraction. We'll add these times to the total travel times for each attraction. We also want to limit the available time each day or we would end up with itineraries that require more hours than the day has. Most attractions open in the late morning and close in the afternoon so in practical terms we have a few hours available each day. I set up this limit to 8 hours per day.

This problem looks very similar to the vehicle routing problem ([VRP](https://en.wikipedia.org/wiki/Vehicle_routing_problem)), a classic in combinatorial optimization. In the vehicle routing problem a set of clients has to be visited by a fleet of vehicles traveling from a common depot. The objective is to find the optimal route for each vehicle. In our case we can assume that we have five vehicles representing our five vacation days. Our depot is our hotel and our clients are our 15 attractions. We also have constraints for our route lengths which makes it a distance-constrained capacitated vehicle problem (DCVRP).

The vehicle routing problem is a generalization of the [travelling salesman problem](https://en.wikipedia.org/wiki/Vehicle_routing_problem), which is an [NP-hard problem](https://en.wikipedia.org/wiki/NP-hardness). This means that while there are exact algorithms that can find optimal solutions, their computation time increases polinomially with the size of the problem. Even with modern computers, finding the optimal solution of a routing problem with more than 15 instances will take more than a life time. If we can't find the optimal solution maybe we can find a good-enough solution. We turn to heuristic and approximation algorithms that yield good solutions to large problems in reasonable time.

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

We obtained a reasonable solution with at most 7.2 hrs of daily activities. Enough time to explore the city during the day and relax in the evening. There is no guarantee this is the best solution achievable. With some luck you could probably find a better solution. That's ok. Remember our objective was to find a good enough solution quickly. Having said that, there are some other considerations:

* The travel times depend on the time and day you run the script. This is because `gmaps.distance_matrix()` returns the durations at the time it was called. So if you run the script on your own you might get a different distance matrix. 
* We assume all of our attractions will be open during our trip. This might not be true for attractions that close during some days or have short opening hours so a final sanity check is a good idea. We can always change the order of our daily itineraries to move around this.
* We don't consider times for other activities such as finding parking and eating. These should be part of the time allocated to each attraction.

[undirected-graph]: https://en.wikipedia.org/wiki/Graph_(mathematics)#Undirected_graph
[local-search]: https://en.wikipedia.org/wiki/Local_search_(optimization)

### Building your own

You can find the code used in this post in [this ipython notebook](https://github.com/riosv/Code-for-blog-posts/blob/master/Toronto-attractions.ipynb). Part of the code is taken from [Randal Olson's](http://www.randalolson.com/2015/03/08/computing-the-optimal-road-trip-across-the-u-s/) article about finding the optimal trip accross the U.S. You can build your own itinerary by defining your own points of interest and visiting times. You might as well change the mode of transportation. Finally you are not restricted to traveling within a city. You can use this model for any set of points for which you can get directions from Google Maps.

