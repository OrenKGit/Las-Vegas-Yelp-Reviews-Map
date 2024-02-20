<script>

import { onMount } from 'svelte'
let tempData = [];
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
let map;
onMount(async () => {

    const res = await fetch('data/output.geojson'); 

    const json_data = await res.json();
    
    tempData = json_data

    console.log(tempData);

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
      .then(data => {
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
      })

        const geocoder = new MapboxGeocoder({
            accessToken: mapboxgl.accessToken,
            mapboxgl: mapboxgl, // Set the mapbox-gl instance
            zoom: 13, // zoom level for geocoding results
            placeholder: 'Search here', // placeholder displayed in the search bar
            bbox: [-120.005, 35.001, -114.039, 42.002] // set bounding box to lousiana 
        });
        map.addControl(geocoder, 'top-left'); // add the search box to the top left
        
        // map flies to location when user enters location
        const marker = new mapboxgl.Marker({ color: '#008000' }); // Create a green marker

        geocoder.on('result', async (event) => {
        // When the geocoder returns a result
        const point = event.result.center; // Capture the result coordinates

        marker.setLngLat(point).addTo(map); // Add the marker to the map at the result coordinates

        // access token and id need to be from same account
        const tileset = 'mque.asmw1dy4' // id for mapbox tileset (data converted to mapbox format)
        const radius = 1609; // one mile
        const limit = 50; // max amount of results to return
        
        marker.setLngLat(point).addTo(map); // Add the marker to the map at the result coordinates
        const query = await fetch(
            `https://api.mapbox.com/v4/${tileset}/tilequery/${point[0]},${point[1]}.json?radius=${radius}&limit=${limit}&access_token=${mapboxgl.accessToken}`,
            { method: 'GET' }
        );
        const json = await query.json();
        console.log(json);
        });
        });

console.log(tempData)

</script>


<main>
</main>