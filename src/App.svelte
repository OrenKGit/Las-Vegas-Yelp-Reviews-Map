<script>

  import { onMount, afterUpdate } from 'svelte'
  let tempData = [];
  import mapboxgl from 'mapbox-gl';
  import 'mapbox-gl/dist/mapbox-gl.css';
  
  
  let map;
  let data;
  let slider_stars = 3; // initial value for stars slider
  let slider_label = "";
  
  onMount(async () => {
  
      mapboxgl.accessToken =
      'pk.eyJ1IjoibXF1ZSIsImEiOiJjbHNtMm51bGcwaWhqMmlyeXZocjNhdXRiIn0.FeNJ2Al_w7at2zuWD-P_qA';
  
      // create map
      /*map = new mapboxgl.Map({
          container: 'map', // container id
          style: 'mapbox://styles/examples/cjgioozof002u2sr5k7t14dim' // map style URL from Mapbox Studio
      });*/
      
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
            map.setFilter('restaurants', ['<=', ['get', 'stars'], slider_stars]);
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

          document.getElementById('stars').addEventListener('input', handleSliderChange);
        }); 
      });

      
  </script>
  
  <main>
    <div class="overlay">
      <label for="stars">Filter by Stars:</label>
      <input
        type="range"
        id="stars"
        min="1"
        max="5"
        step="0.5"
        bind:value={slider_stars}
      />
      <span>{slider_stars}</span>
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
      background-color: rgba(255, 255, 255, 0.4);
      position: absolute;
      min-width:250px;
      width: 15%;
      top: 10px;
      right: 10px;
      padding: 10px;
      z-index: 3;
    }

    label {
      font-size: 1.5em;
      font-family: sans-serif;
      font-weight: bold;
    }
  </style>