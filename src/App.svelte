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
    'pk.eyJ1Ijoicm9iaW5rb2hycyIsImEiOiJjanU5am95bm4xZnZ6NDNrOTRyYTYwdzJzIn0.iMFQgQIlhz36wB3819Xftw';

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
});

console.log(tempData)

</script>

<main>
</main>