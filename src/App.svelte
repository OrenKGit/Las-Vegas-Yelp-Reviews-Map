<script>

import { onMount } from 'svelte'
let tempData = [];
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
let map;
onMount(async () => {

    const res = await fetch('data/yelp_neworleans.json'); 

    const json_data = await res.json();
    
    tempData = json_data

    console.log(tempData);

    mapboxgl.accessToken =
    'pk.eyJ1IjoibXF1ZSIsImEiOiJjbHNtMm51bGcwaWhqMmlyeXZocjNhdXRiIn0.FeNJ2Al_w7at2zuWD-P_qA';

    // create map
    /*map = new mapboxgl.Map({
        container: 'map', // container id
        style: 'mapbox://styles/examples/cjgioozof002u2sr5k7t14dim' // map style URL from Mapbox Studio
    });*/
    const map = new mapboxgl.Map({
		container: "map",
		style: "mapbox://styles/mapbox/light-v11", 
		center: [-90.0715, 29.9511], 
		zoom: 11, // starting zoom level
		minZoom: 5,
		maxZoom: 25,
	});

    // adding geocoder and search bar (has to be address)
    map.on('load', () => {
        const geocoder = new MapboxGeocoder({
            accessToken: mapboxgl.accessToken,
            mapboxgl: mapboxgl, // Set the mapbox-gl instance
            zoom: 13, // zoom level for geocoding results
            placeholder: 'Search here', // placeholder displayed in the search bar
            bbox: [-94.043, 28.929, -88.817, 33.019] // set bounding box to lousiana 
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
            map.getSource('tilequery').setData(json);
        });

        map.addSource('tilequery', {
            // Add a new source to the map style: https://docs.mapbox.com/mapbox-gl-js/api/#map#addsource
            type: 'geojson',
            data: {
                type: 'FeatureCollection',
                features: []
            }
        });

        map.addLayer({
            id: 'tilequery-points',
            type: 'circle',
            source: 'tilequery', // Set the layer source
            paint: {
                'circle-stroke-color': 'white',
                'circle-stroke-width': {
                stops: [
                    [0, 0.1],
                    [18, 3]
                ],
                base: 5
                },
                'circle-radius': {
                stops: [
                    [12, 5],
                    [22, 180]
                ],
                base: 5
                },
                'circle-color': [
                // Specify the color each circle should be
                'match',
                ['get', 'stars'], // getting stars and coloring them
                5,
                '#008000',
                4,
                '#9ACD32',
                3,
                '#FF8C00',
                2,
                '#FF0000',
                1,
                '#FF0000'
                ]
            }
            });
        
        console.log(json);
        });
        });


console.log(tempData)

</script>

<main>
</main>