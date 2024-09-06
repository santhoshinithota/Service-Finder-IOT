<script>
import Map from "./lib/Map.svelte";
import Navbar from "./lib/Navbar.svelte";
import Tags from "./lib/Tags.svelte";
import ServiceList from "./lib/ServiceList.svelte";
import MoreInfo from "./lib/MoreInfo.svelte";

let map, serviceList;

</script>

<main>
    <Map bind:this={map}/>
    <Navbar />
    <Tags serviceList={serviceList}/>
    <ServiceList bind:this={serviceList} map={map}/>
</main>

<style>
</style>
