<script>

  import { onMount, afterUpdate } from 'svelte'
  let tempData = [];
  import mapboxgl from 'mapbox-gl';
  import 'mapbox-gl/dist/mapbox-gl.css';
  
  
  let map;
  let data;
  let slider_stars = 3; // initial value for stars slider
  let slider_label = "";
  let slider_start = 1;
  let slider_end = 5;
  let selectedCuisines = [];
  
  onMount(async () => {
  
    mapboxgl.accessToken =
    'pk.eyJ1IjoibXF1ZSIsImEiOiJjbHNtMm51bGcwaWhqMmlyeXZocjNhdXRiIn0.FeNJ2Al_w7at2zuWD-P_qA';
    
    const map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/mapbox/dark-v11", 
      center: [-115.1391, 36.1716], 
      zoom: 11, // starting zoom level
      minZoom: 10,
      maxZoom: 18,
    });

    fetch('data/output.geojson')
      .then(response => response.json())
      .then(initialData => {
        data = initialData;

        const colorScale = d3.scaleLinear()
          .domain([1, 5])
          .range(['#ff0000', '#66ff33']);

        // Create a size scale based on review count
        const sizeScale = d3.scaleLinear()
          .domain([0, d3.max(data.features, d => d.properties.review_count)])
          .range([5, 30]);

        function updateMapLayers() {
          map.setFilter('restaurants', [
            'all',
            ['>=', ['get', 'stars'], slider_start],
            ['<=', ['get', 'stars'], slider_end],
            selectedCuisines.length > 0
                ? [
                    'any',
                    ...selectedCuisines.map((cuisine) => [
                      'in',
                      cuisine.toLowerCase(),
                      ['downcase', ['get', 'categories']],
                    ]),
                  ]
                : true,
            ]);
        }

        map.addLayer({
          id: 'restaurants',
          type: 'circle',
          source: {
            type: 'geojson',
            data: data,
          },
          paint: {
            'circle-radius': [
              'interpolate',
              ['linear'],
              ['get', 'review_count'],
              10, 2,
              d3.max(data.features, d => d.properties.review_count)/1.25, 15
            ],
            'circle-color': [
              'interpolate',
              ['linear'],
              ['get', 'stars'],
              1, '#EC204F',
              5, '#2ECF03'
            ],
            //'circle-opacity': 0.3,
            'circle-opacity': [
                "interpolate", ["linear"], ["zoom"],
                // zoom is 5 (or less) -> circle radius will be 1px
                11, 0.3,
                // zoom is 10 (or greater) -> circle radius will be 5px
                15, 1
            ]
          }
        });
        
        function handleSliderChange() {
        updateMapLayers();
        }
        
        function handleCheckboxChange() {
          selectedCuisines = [];
          
          document.querySelectorAll('.cuisine-checkbox:checked').forEach((checkbox) => {
            selectedCuisines.push(checkbox.value);
          });

          updateMapLayers();
        }

        document.getElementById('start').addEventListener('input', handleSliderChange);
        document.getElementById('end').addEventListener('input', handleSliderChange);
        document.querySelectorAll('.cuisine-checkbox').forEach((checkbox) => {
          checkbox.addEventListener('change', handleCheckboxChange);
        });
      }); 
    
    // Create a popup, but don't add it to the map yet.
    const popup = new mapboxgl.Popup({
      closeButton: false
    });
    
    map.on('load', () => {
        
      map.on('mousemove', 'restaurants', (e) => {
      // Change the cursor style as a UI indicator.
      map.getCanvas().style.cursor = 'pointer';
        
      // Populate the popup and set its coordinates based on the feature.
      const feature = e.features[0];
      popup
        .setLngLat(feature.geometry.coordinates)
        .setHTML(
          `<h3>${feature.properties.name}</h1> 
          <b>Star Rating:</b> ${feature.properties.stars} <br>
          <b>Review Count:</b>  ${feature.properties.review_count} <br>
          <b>Address:</b>  ${feature.properties.address}`,
        )
        .addTo(map);
      });
        
      map.on('mouseleave', 'restaurants', () => {
        map.getCanvas().style.cursor = '';
        popup.remove();
      });
        
    });
      
    // MARKERS WITH STUFF
    map.on('load', () => {
      map.addSource('places', {
        'type': 'geojson',
        'data': {
        'type': 'FeatureCollection',
        'features': [
        {
        // 
        'type': 'Feature',
        'properties': {
        'description':
        '<strong>The Las Vegas Strip</strong><p>The strip, which is the most famous part of Las Vegas, is obviously home to the most popular businesses, but there is a mix of well-rated and poorly-rated businesses here</p>',
        'icon': 'theatre'
        },
        'geometry': {
        'type': 'Point',
        'coordinates': [-115.1728, 36.1147]
        }
        },
        {
        'type': 'Feature',
        'properties': {
        'description':
        '<strong>Downtown</strong><p>To the north of the strip, the downtown area is home to several highly rated but relatively obscure businesses</p>',
        'icon': 'bar'
        },
        'geometry': {
        'type': 'Point',
        'coordinates': [-115.1391, 36.1716]
        }
        },
        {
        'type': 'Feature',
        'properties': {
        'description':
        '<strong>Las Vegas Airport</strong><p>The airport has a large amount of very poorly rated restaurants, with very few reviews.</p>',
        'icon': 'airport'
        },
        'geometry': {
        'type': 'Point',
        'coordinates': [-115.1482, 36.0831]
        }
        },
        {
        'type': 'Feature',
        'properties': {
        'description':
        '<strong>China Town</strong><p>Not far from the strip, Chinatown is home to many hidden gems, with few reviews but very high ratings</p>',
        'icon': 'restaurant'
        },
        'geometry': {
        'type': 'Point',
        'coordinates': [-115.1960, 36.1257]
        }
        }
      ]
      }
      });
      // Add a layer showing the places.
      map.addLayer({
      'id': 'places',
      'type': 'symbol',
      'source': 'places',
      'layout': {
      'icon-image': ['get', 'icon'],
      'icon-allow-overlap': true,
      'icon-size': 2
      }
      });
      
      // When a click event occurs on a feature in the places layer, open a popup at the
      // location of the feature, with description HTML from its properties.
      map.on('click', 'places', (e) => {
      // Copy coordinates array.
      const coordinates = e.features[0].geometry.coordinates.slice();
      const description = e.features[0].properties.description;
      
      // Ensure that if the map is zoomed out such that multiple
      // copies of the feature are visible, the popup appears
      // over the copy being pointed to.
      while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
      coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
      }
      
      new mapboxgl.Popup()
      .setLngLat(coordinates)
      .setHTML(description)
      .addTo(map);
      });
      
      // Change the cursor to a pointer when the mouse is over the places layer.
      map.on('mouseenter', 'places', () => {
      map.getCanvas().style.cursor = 'pointer';
      });
      
      // Change it back to a pointer when it leaves.
      map.on('mouseleave', 'places', () => {
      map.getCanvas().style.cursor = '';
      });

      const popup = new mapboxgl.Popup({ closeOnClick: false })
      .setLngLat([-115.1960, 36.1310])
      .setHTML('<p>Click Me!</p>')
      .addTo(map);
    });
    });

      
  </script>
  
  <main>
    <h1 style="position: absolute; top: 20px; left: 50%; transform: translateX(-50%); z-index: 1000; color: white;">Las Vegas <span style="color: #ff4500;">Hot Spots</span></h1>  
    <h3 style="position: absolute; top: 60px; left: 50%; transform: translateX(-50%); z-index: 1000; color: white;"> <span style="color: #ff4500;"></span> Yelp Review Map </h3>  
    <div class="overlay">
      <label for="start">Filter by Stars:</label>
      <br><br>
      <label2 for="start">Above: {slider_start}</label2>
      <input
        type="range"
        id="start"
        min="1"
        max="5"
        step="0.5"
        bind:value={slider_start}
      />
      <label2 for="end">Below: {slider_end}</label2>
      <input
        type="range"
        id="end"
        min="1"
        max="5"
        step="0.5"
        bind:value={slider_end}
      />
      <span>Between {slider_start} and {slider_end} Stars</span>
    </div>
    <div class="overlay4">
      <label class = "cuisine-label">Filter by Cuisines:</label>
      <br><br>
      <span class="checkbox-container">
        <label for="Mexican" class="checkboxes">Mexican<input type="checkbox" id="Mexican" class="cuisine-checkbox" value="Mexican" />
          <span class="checkbox-label">Mexican</span></label>
        <label for="American" class="checkboxes">American<input type="checkbox" id="American" class="cuisine-checkbox" value="American" />
          <span class="checkbox-label">American</span></label>
        <label for="Italian" class="checkboxes">Italian<input type="checkbox" id="Italian" class="cuisine-checkbox" value="Italian" />
          <span class="checkbox-label">Italian</span></label>
        <label for="Chinese" class="checkboxes">Chinese<input type="checkbox" id="Chinese" class="cuisine-checkbox" value="Chinese" />
          <span class="checkbox-label">Chinese</span></label>
        <label for="Japanese" class="checkboxes">Japanese<input type="checkbox" id="Japanese" class="cuisine-checkbox" value="Japanese" />
          <span class="checkbox-label">Japanese</span></label>
        <label for="Korean" class ="checkboxes">Korean<input type="checkbox" id="Korean" class="cuisine-checkbox" value="Korean" />
          <span class="checkbox-label">Korean</span></label>
      </span>
    </div>
    <div class="overlay2">
      <label>Legend</label>
      <br><br>
      <span>
        <b>Circle:</b> Business Location <br>
        <b>Circle Size:</b> Number of Reviews <br>
        <b>Circle Color:</b> Star Rating <br>

      </span>
    </div>
    <div class="overlay3">
      <label>Description</label>
      <br><br>
      <span>
     
        Las Vegas is not a very walkable city. Find the hottest parts of town with our map, so you'll never have to walk far to get to your next dopamine hit.
        <br>
        <br>
        This visualization shows the distribution of businesses in Las Vegas, with their popularity and rating.
        <br>
        Circle size shows the number of reviews a business has and circle color shows how good the reviews are.
        <br>
        Larger circles have more reviews and greener circles have better reviews.
        <br>
        <br>
        Data Source: <a href="https://www.yelp.com/dataset">Yelp Open Dataset</a>
        <br>
        <br>
        Data has been filtered to only include the Las Vegas area
        
      </span>
      <br><br>

      <label>Write-Up</label>

      <br><br>

      <span>

        <b>Rationale for Design Decisions:</b>
        <br><br>
        We decided to take the approach of making a map to help tourists find popular businesses and areas. The choices we made serve this purpose. We chose to make a map of Las Vegas as opposed to other cities since it is a major renowned resort city that is known for its dining, entertainment, and nightlife and would have many interesting businesses we can examine on a map. 

<br><br>
For our visualization, we decided to go with a map, since it allows us to display how businesses are spread around Las Vegas. We then used circular points to mark each business, which have size and color attributes that we could scale to represent review count and rating respectively. For example, the radius of our circles represents the review count of the business, with larger ratings indicating a higher review count. We chose this design choice because we thought a higher review count would also be indicative of the popularity of the business, so the business would also be more busy as well. We also used dot color to represent the star rating of the businesses, with higher ratings represented by green dots (5 stars) and lower ratings represented by red dots (1 star). These specific colors were chosen since green usually represents good and red represents bad, so it would be intuitive for the viewers to understand. We chose a dark gray background as this allowed the bright colors to stand out and grab viewers' attention on popular spots on the map.
<br><br>
Our initial idea was to have an interactive map of Las Vegas, with a time-oriented slider. There would be points on the map, marking businesses, scaled by review count and colored according to average ratings. By dragging the slider to a different point in the day, the points would appear and disappear according to if that business was open at that time. However, we decided to scrap the initial idea of implementing a time-based slider because the way business hours were formatted in our dataset made it too time-consuming and complex to implement as a slider. Instead, we decided to implement a slider based on restaurant ratings, which filtered the map to show restaurants that were rated different stars ranging from 1 to 5. Although we wanted to make a double-ended slider where viewers can change the front and end positions, we were unable to get it to filter correctly. Rather, we settled on making a second slider so that viewers can also change the positions on the two sliders to see businesses that have ratings within that range. Although this was not as clean looking, it still worked effectively to serve our purpose. 
<br><br>
Furthermore, we also had the idea of adding a search bar where viewers would be able to search businesses, and the map would fly to the location of the business and display more information about it. We ended up going with another element instead due to difficulties in getting the search bar to work properly with our map. We imagine that most viewers would only be interested in restaurants specializing in foods they like, so we added checkboxes as cuisine filters to enable users to filter businesses by cuisines. This allows viewers to explore specific culinary preferences and find a good restaurant based on their ratings and reviews. When hovering over each point, viewers can read the names of the business, the star rating, the review count, as well as the address of the business. This makes it easy for tourists to identify and locate businesses that they find through our map. We also added icons to point out famous spots in Las Vegas, such as the Strip and Chinatown, that viewers can click on and read the description of. This allows the visualization to provide more information on how businesses are distributed around destinations that have more tourists and how they are rated. Due to not wanting our map to look too cluttered, we focused on these interactions to engage the viewers with exploring restaurants they are interested in. Future ideas that can be implemented to improve interactivity include making filters for the business type (e.g. fast food, cafe, bars) and adding a search bar so that viewers can search for restaurants or locations they have in mind to see their reviews and ratings quickly. 
<br><br>
    <b>Development Process:</b>
        <br><br>
We began by deciding on a dataset to use, going with the Yelp dataset because we believed that it provided very interesting data. We were especially interested in how it could map out businesses since eating out is a universal experience that can tell you a lot about recreational habits.
<br><br>
We attempted to filter our dataset to include only restaurants since food is a major scene in Las Vegas and tourists would be able to use our map to find the best places to eat in Las Vegas according to their culinary preferences. However, this was not a perfect filter as some businesses may not be restaurants but still include restaurants as one of their tags due to their association with food. Thus, our dataset also included some businesses as well due to the way our initial data was labeled, but it mostly focuses on restaurants. Furthermore, a few businesses specified a region of Las Vegas instead, such as North or South Las Vegas, and some select ones also had typos in their spelling of Las Vegas. Since we wanted to focus on just the central part and prevent our dataset from being too large, we filtered our data to just include businesses that had exactly “Las Vegas” as their city, which still gave us a large enough dataset to plot on the map. 
<br><br>
From there, we discussed how we wanted to approach the visualization, but we quickly ran into some significant issues. We struggled primarily with setting up our data and having the businesses appear on our maps, but we eventually resolved it by debugging our code and making sure that the datasets were in the correct format. Ethan imported the necessary data to the repository and provided ideas for the visualization, Maya added the points on the map and implemented interactive elements such as the slider and checkboxes, and Oren formatted the visualization, added engaging annotations, and improved the overall style of the visualization to make it cohesive for readers to view. After getting the points to appear on the map, we added visual encodings that were chosen specifically to be more intuitive for readers to understand. 
<br><br>
Then, we implemented our interaction elements, which took up the most time. Although we had several ideas on how to make the map interactive, we struggled the most with actually implementing and debugging our code, running into several issues where the points would not correctly appear on our map. This led to us scrapping our initial ideas and starting over with alternatives instead that still provided an engaging experience for viewers and were able to be implemented by us. After getting the interactivity to work, we made stylistic choices in formatting and designing our visualization to make it look elegant and thoughtful. We considered where was the best place to put the description, legend, and filter on the map, changed font colors and sizes, and improved color choices so that it was both aesthetically pleasing and attention-grabbing.
<br><br>
Everyone worked on the debugging, but Oren focused on the implementation of encodings and tooltips, Maya worked on the sliders and filters, and Ethan wrote up the write-up and tooltip descriptions. Overall, we spent about twenty people-hours on developing this visualization.

      </span>
    </div>
    
    <div>
      <h1>HI</h1>
    </div>
  </main>

  <style>
    :global(body) {
      margin: 0;
      padding: 0;
    }

    :global(#map) {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    }

    input {
      display: inline-block;
      width: 100%;
      position: relative;
      margin: 0;
      cursor: pointer;
    }

    .overlay {
      font-size: 0.9em;
      background-color: rgba(100, 100, 100, 0.1);
      position: absolute;
      min-width:250px;
      width: 15%;
      top: 10px;
      right: 10px;
      padding: 10px;
      z-index: 3;
      font-family: sans-serif;
      font-weight: lighter;
      color: rgba(200, 200, 200, 1);
    }

    .overlay2 {
      font-size: 0.9em;
      background-color: rgba(100, 100, 100, 0.1);
      position: absolute;
      min-width:250px;
      width: 15%;
      top: 10px;
      left: 10px;
      padding: 10px;
      z-index: 3;
      font-family: sans-serif;
      font-weight: lighter;
      color: rgba(200, 200, 200, 1);
    }

    .overlay3 {
      font-size: 0.9em;
      background-color: #6464641a;
      position: absolute;
      min-width:250px;
      width: 15%;
      top: 380px;
      right: 10px;
      padding: 10px;
      z-index: 3;
      font-family: sans-serif;
      font-weight: lighter;
      color: rgba(200, 200, 200, 1);
      overflow: auto;
      max-height: 400px;
    }

    .overlay4 {
    font-size: 0.6em;
    background-color: rgba(100, 100, 100, 0.1);
    position: absolute;
    min-width: 250px;
    width: 15%;
    top: 165px;
    right: 10px;
    padding: 10px;
    z-index: 3;
    font-family: sans-serif;
    font-weight: lighter;
    color: rgba(200, 200, 200, 1);
  }

  .cuisine-label {
    font-size: 2.2em
  }

    label {
      font-size: 1.5em;
      font-family: sans-serif;
      font-weight: lighter;
    }

    .label2 {
      font-size: 1em;
      font-family: sans-serif;
      font-weight: lighter;
    }
    a:link {
      color: red;
      background-color: transparent;
      text-decoration: none;
    }

    a:visited {
      color: red;
      background-color: transparent;
      text-decoration: none;
    }
    .checkboxes {
      display: flex;
      align-items: center;
      margin-bottom:5px;
    }
    .checkbox-container {
      display: grid;
      grid-template-columns: auto;
      gap: 5px;
    }
    .checkbox-label {
      display: flex;
      align-items: center;
      visibility:hidden;
    }

  </style>
