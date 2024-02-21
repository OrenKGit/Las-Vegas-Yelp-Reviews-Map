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
  
        });    
      });
      
    function filterRestaurants(stars) {
      const filteredData = {
        type: 'FeatureCollection',
        features: data.features.filter((restaurant) => restaurant.properties.stars >= stars),
      };
      map.getSource('restaurants').setData(filteredData);
      slider_label = `Stars: ${stars}`;
    }
  
  </script>
  
  <main>
    <div class="overlay">
      <label for="stars">{slider_label}</label>
      <input id="slider" type="range" min="1" max="5" step="0.5" on:input={() => filterRestaurants(slider_stars)} bind:value={slider_stars} />
    </div>
  </main>