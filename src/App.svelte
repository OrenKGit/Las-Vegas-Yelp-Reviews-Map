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
          selectedCuisines = []; // Clear the array

          // Check each checkbox and add selected cuisines to the array
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
        '<strong>Make it Mount Pleasant</strong><p><a href="http://www.mtpleasantdc.com/makeitmtpleasant" target="_blank" title="Opens in a new window">Make it Mount Pleasant</a> is a handmade and vintage market and afternoon of live entertainment and kids activities. 12:00-6:00 p.m.</p>',
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
        '<strong>Mad Men Season Five Finale Watch Party</strong><p>Head to Lounge 201 (201 Massachusetts Avenue NE) Sunday for a <a href="http://madmens5finale.eventbrite.com/" target="_blank" title="Opens in a new window">Mad Men Season Five Finale Watch Party</a>, complete with 60s costume contest, Mad Men trivia, and retro food and drink. 8:00-11:00 p.m. $10 general admission, $20 admission and two hour open bar.</p>',
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
        '<strong>Big Backyard Beach Bash and Wine Fest</strong><p>EatBar (2761 Washington Boulevard Arlington VA) is throwing a <a href="http://tallulaeatbar.ticketleap.com/2012beachblanket/" target="_blank" title="Opens in a new window">Big Backyard Beach Bash and Wine Fest</a> on Saturday, serving up conch fritters, fish tacos and crab sliders, and Red Apron hot dogs. 12:00-3:00 p.m. $25.grill hot dogs.</p>',
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
        '<strong>Big Backyard Beach Bash and Wine Fest</strong><p>EatBar (2761 Washington Boulevard Arlington VA) is throwing a <a href="http://tallulaeatbar.ticketleap.com/2012beachblanket/" target="_blank" title="Opens in a new window">Big Backyard Beach Bash and Wine Fest</a> on Saturday, serving up conch fritters, fish tacos and crab sliders, and Red Apron hot dogs. 12:00-3:00 p.m. $25.grill hot dogs.</p>',
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
      <label for="Mexican" class="checkboxes">Mexican<input type="checkbox" id="Mexican" class="cuisine-checkbox" value="Mexican" /></label>
      <label for="American" class="checkboxes">American<input type="checkbox" id="American" class="cuisine-checkbox" value="American" /></label>
      <label for="Italian" class="checkboxes">Italian<input type="checkbox" id="Italian" class="cuisine-checkbox" value="Italian" /></label>
      <label for="Chinese" class="checkboxes">Chinese<input type="checkbox" id="Chinese" class="cuisine-checkbox" value="Chinese" /></label>
      <label for="Japanese" class="checkboxes">Japanese<input type="checkbox" id="Japanese" class="cuisine-checkbox" value="Japanese" /></label>
      <label for="Korean" class ="checkboxes">Korean<input type="checkbox" id="Korean" class="cuisine-checkbox" value="Korean" /></label>
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
        Data has been filtered to only include the Las Vegas area

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
    }

    .overlay4 {
    font-size: 0.9em;
    background-color: rgba(100, 100, 100, 0.1);
    position: absolute;
    min-width: 250px;
    width: 15%;
    top: 170px;
    right: 10px; /* Update to move to bottom left */
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