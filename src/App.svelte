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

        document.getElementById('start').addEventListener('input', handleSliderChange);
        document.getElementById('end').addEventListener('input', handleSliderChange);
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
        
        Data Source: <a href="https://www.yelp.com/dataset">Yelp Open Dataset</a>
        <br>
        <br>
        This visualization shows the distribution of businesses in Las Vegas by their popularity and rating.
        <br>
        Circle size shows the number of reviews a business has and circle color shows how good the reviews are.
        <br>
        Larger circles have more reviews and greener circles have better reviews.
        <br>
        <br>
        Data has been filtered to only include the Las Vegas area
      </span>
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
      background-color: rgba(100, 100, 100, 0.1);
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