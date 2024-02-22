**Rationale for Design Decisions**:

We decided to take the approach of making a map to help tourists find popular businesses and areas. The choices we made serve this purpose.

For our visualization, we decided to go with a map, since it allows us to display how businesses are spread around Las Vegas. We then used circular points to mark each business, which have size and color attributes that we could scale to represent review count and rating respectively. For example, the radius of our circles represents the review count of business, with larger ratings indicating higher review count. We chose this design choice because we thought a higher review count would also be indicative of the popularity of the business, so the business would also be more busy as well. We also used dot color to represent the star rating of the businesses, with higher ratings represented by green dots (5 stars) and lower ratings represented by red dots (1 stars). 

We chose a gray background as this allowed the bright colors to stand out. We also implemented double sliders so that the user can filter the businesses according to their ratings and included checkboxes to serve as cuisine filters to enable users to filter businesses by cuisines, allowing them to explore specific culinary preferences and judge them by their ratings and reviews. 

When hovering over each point, viewers can read the names of the business, the star rating, the review count, as well as the address of the business. This makes it easy for tourists to identify and locate businesses that they find through our map.

Although we tried filtering by only restaurants, our datasets also included businesses as well due to the way it was labelled. Further improvements can be made to this visualization by including solely restaurants in the map so that it is easier for viewers to understand, as well as changing the style of the checkboxes. 

**Development Process**:
We began by deciding on a dataset to use, going with the Yelp dataset because we believed that it provided very interesting data. We were especially interested in how it could map out businesses, since eating out is a universal experience that can tell you a lot about recreational habits.

From there, we discussed how we wanted to approach the visualization. Our initial idea was to have an interactive map of Las Vegas, with a time-oriented slider similar to the lab. There would be points on the map, marking businesses, scaled by review count and colored according to average ratings. By dragging the slider to a different point in the day, the points would appear and disappear according to if that business was open at that time.

However, we quickly ran into some significant issues. We struggled primarily with setting up our data and having the businesses appear on our maps, but we eventually resolved it by making sure that our datasets were in the correct format. Ethan imported the necessary data to the repository, Maya added the points on the map and implement interactive elements such as the slider and checkboxes, and Oren formatted the visualization, added annotations, and improved the overall style of the visualization to make it cohesive for readers to view. Since our visualization at this point just included circles and was not very informative, we decided to add a tooltip that would show information about the business, such as its name, address, star rating, and review count when the viewer hovers their mouse over it. In this way, the visualization not only provides spatial information about the physical distribution of businesses in Las Vegas, but also information such as its ratings and review counts. 

We decided to scrap the initial idea of implementing a time-based slider to show what businesses are open due to the variability in hours throughout the week, as well as difficulty in getting it to work due to the way that business hours were formatted in our dataset. Instead, we used a rating-based slider to show businesses that were rated different stars ranging from 1 to 5. We also included a second slider so that viewers can  change the positions of the beginning of the slider to see only the businesses that fall within a certain range. We added a checkbox that corresponds to popular cuisines in Las Vegas so that viewers can examine the rating and review count of businesses of a particular cuisine in Las Vegas as well as gain information about how it is distributed. Lastly, we added icons of significant landmarks of Las Vegas to the map that viewers can click on and read the description of. This allows the visualization to provide more information how businesses are distributed around destinations that have more tourists and how they are rated. 

Everyone worked on the debugging, but Oren focused on the implementation of encodings and tooltips, Maya worked on the sliders and filters, and Ethan wrote up the write up and tooltip descriptions. In general, we spent about twenty people-hours on developing this visualization. Although we had several ideas on how to make the map interactive, we struggled the most with actually implementing it and debugging our code, running into several issues where the points would not correctly appear on our map. We also spent a lot of time in the beginning stage when setting up our map, mostly due to our lack of familiarity with using D3 and svelte. 


