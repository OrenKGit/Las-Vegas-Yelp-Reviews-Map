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
      'icon-size': 1.25
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
      .setLngLat([-115.1960, 36.1257])
      .setHTML('<p>Click Me!</p>')
      .addTo(map);
    });
    });

      
  </script>
  
  <main>
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
      <label>Filter by Cuisines:</label>
      <br><br>
      <span>
        <label for="Mexican" class="checkboxes">Mexican<input type="checkbox" id="Mexican" class="cuisine-checkbox" value="Mexican" /></label>
        <label for="American" class="checkboxes">American<input type="checkbox" id="American" class="cuisine-checkbox" value="American" /></label>
        <label for="Italian" class="checkboxes">Italian<input type="checkbox" id="Italian" class="cuisine-checkbox" value="Italian" /></label>
        <label for="Chinese" class="checkboxes">Chinese<input type="checkbox" id="Chinese" class="cuisine-checkbox" value="Chinese" /></label>
        <label for="Japanese" class="checkboxes">Japanese<input type="checkbox" id="Japanese" class="cuisine-checkbox" value="Japanese" /></label>
        <label for="Korean" class ="checkboxes">Korean<input type="checkbox" id="Korean" class="cuisine-checkbox" value="Korean" /></label>
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
We decided to take the approach of making a map to help tourists find popular businesses and areas. The choices we made serve this purpose.
<br><br>
For our visualization, we decided to go with a map, since it allows us to display how businesses are spread around Las Vegas. We then used circular points to mark each business, which have size and color attributes that we could scale to represent review count and rating respectively. For example, the radius of our circles represents the review count of business, with larger ratings indicating higher review count. We chose this design choice because we thought a higher review count would also be indicative of the popularity of the business, so the business would also be more busy as well. We also used dot color to represent the star rating of the businesses, with higher ratings represented by green dots (5 stars) and lower ratings represented by red dots (1 stars). 
<br><br>
We chose a gray background as this allowed the bright colors to stand out. We also implemented double sliders so that the user can filter the businesses according to their ratings and included checkboxes to serve as cuisine filters to enable users to filter businesses by cuisines, allowing them to explore specific culinary preferences and judge them by their ratings and reviews. 
<br><br>
When hovering over each point, viewers can read the names of the business, the star rating, the review count, as well as the address of the business. This makes it easy for tourists to identify and locate businesses that they find through our map.
<br><br>
Although we tried filtering by only businesses, our datasets also included businesses as well due to the way it was labelled. Further improvements can be made to this visualization by including solely food restaurants in the map so that it is easier for viewers to understand, as well as changing the style of the checkboxes. 
<br><br>
    <b>Development Process:</b>
        <br><br>
We began by deciding on a dataset to use, going with the Yelp dataset because we believed that it provided very interesting data. We were especially interested in how it could map out businesses, since eating out is a universal experience that can tell you a lot about recreational habits.
<br><br>
From there, we discussed how we wanted to approach the visualization. Our initial idea was to have an interactive map of Las Vegas, with a time-oriented slider similar to the lab. There would be points on the map, marking businesses, scaled by review count and colored according to average ratings. By dragging the slider to a different point in the day, the points would appear and disappear according to if that business was open at that time.
<br><br>
However, we quickly ran into some significant issues. We struggled primarily with setting up our data and having the businesses appear on our maps, but we eventually resolved it by making sure that our datasets were in the correct format. Ethan imported the necessary data to the repository, Maya added the points on the map and implement interactive elements such as the slider and checkboxes, and Oren formatted the visualization, added annotations, and improved the overall style of the visualization to make it cohesive for readers to view. After getting the points to appear on the map, we decided to change the bubble size of the points as well as the color. Since our visualization at this point just included circles and was not very informative, we decided to add a tooltip that would show information about the business, such as its name, address, star rating, and review count when the viewer hover their mouse over it. In this way, the visualization not only provides spatial information about not only the physical distribution of businesses in Las Vegas, but also information such as its ratings and review counts. 
<br><br>
We decided to scrap the initial idea of implementing a time-based slider to show what businesses are open due to the variability in hours throughout the week, as well as difficulty in getting it to work due to the way that business hours were formatted in our dataset. Instead, we used a rating based slider to show businesses that were rated different stars ranging from 1 to 5. We also included a second slider so that viewers can also change the positions of the beginning of the slider to see just businesses that fall within a certain range. We added a checkbox that corresponds to popular cuisines in Las Vegas so that viewers can examine the rating and review count of businesses of a particular cuisine in Las Vegas as well as gain information about how it is distributed. Lastly, we added icons of significant landmarks of Las Vegas to the map that viewers can click on and read the description of. This allows the visualization to provide more information how businesses are distributed around destinations that have more tourists and how they are rated. 
<br><br>
In general, we spent about twenty people hours on developing this visualization. Although we had several ideas on how to make the map interactive, we struggled the most with actually implementing it and debugging our code, running to several issues when the points would not correctly appear on our map. We also spent a lot of time in the beginning stage when setting up our map, mostly due to our lack of familiarity with using D3 and svelte. 

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
      top: 350px;
      right: 10px;
      padding: 10px;
      z-index: 3;
      font-family: sans-serif;
      font-weight: lighter;
      color: rgba(200, 200, 200, 1);
      overflow: auto;
      max-height: 10000px;
      max-height: 430px;
    }

    .overlay4 {
    font-size: 0.6em;
    background-color: rgba(100, 100, 100, 0.1);
    position: absolute;
    min-width: 250px;
    width: 15%;
    top: 170px;
    right: 10px;
    padding: 10px;
    z-index: 3;
    font-family: sans-serif;
    font-weight: lighter;
    color: rgba(200, 200, 200, 1);
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
      justify-content: center;
      align-items: center;
      vertical-align: middle;
      word-wrap: break-word;
    }
  </style>