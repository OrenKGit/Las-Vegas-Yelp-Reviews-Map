**Rationale for Design Decisions**:

**Development Process**:
We began by deciding on a dataset to use, going with the Yelp dataset because we believed that it provided very interesting data. We were especially interested in how it could map out restaurants, since eating out is a universal experience that can tell you a lot about recreational habits.

From there, we discussed how we wanted to approach the visualization. Our initial idea was to have an interactive map of Las Vegas, with a time-oriented slider similar to the lab. There would be points on the map, marking restaurants, scaled by review count and colored according to average ratings. By dragging the slider to a different point in the day, the points would appear and disappear according to if that restaurant was open at that time.

However, we quickly ran into some significant issues. We struggled primarily with setting up our data and having the restaurants appear on our maps. This took up around a dozen people-hours, but we eventually resolved it. Ethan imported the necessary data to the repository, Maya formatted and filtered the dataset, and Oren converted it to the correct file type and got the restaurants to appear on the map.

We decided to scrap the initial idea of implementing a time-based slider to show what restaurants are open due to the variability in hours throughout the week, as well as difficulty in getting it to work. Instead
