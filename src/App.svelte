<script>

import { onMount } from 'svelte'
let tempData = [];

onMount(async () => {

    const res = await fetch('data/yelp_neworleans.json'); 

    const json = await res.json();
    
    console.log(json);

});
</script>

<main>
    
</main>