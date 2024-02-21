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
      <label>Cuisines</label>
      <br>
      <input type="checkbox" id="Mexican" class="cuisine-checkbox" value="Mexican" />
      <label for="Mexican">Mexican</label>
      <input type="checkbox" id="American" class="cuisine-checkbox" value="American" />
      <label for="American">American</label>
      <input type="checkbox" id="Italian" class="cuisine-checkbox" value="Italian" />
      <label for="Italian">Italian</label>
      <input type="checkbox" id="Chinese" class="cuisine-checkbox" value="Chinese" />
      <label for="Chinese">Chinese</label>
      <input type="checkbox" id="Japanese" class="cuisine-checkbox" value="Japanese" />
      <label for="Japanese">Japanese</label>
      <input type="checkbox" id="Korean" class="cuisine-checkbox" value="Korean" />
      <label for="Korean">Korean</label>
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
      top: 170px;
      right: 10px;
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
      top: 285px;
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
    bottom: 10px;
    left: 10px; /* Update to move to bottom left */
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

  </style>