<script>
import { onMount } from "svelte";
import { Map, Tile, View } from "ol";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import * as proj from "ol/proj";

let map;
onMount(async() => {
    navigator.geolocation.getCurrentPosition(pos => {
        const lat  = pos.coords.latitude;
        const long = pos.coords.longitude;
        console.log(pos.coords);

        map = new Map({
            target: 'map',
            layers: [
                new TileLayer({
                    source: new OSM()
                })
            ],
            view : new View({
                center: proj.fromLonLat([long, lat]),
                zoom: 18
            })
        });

    }, error => {
        console.log(error);
    });
});

export function focus(long, lat)
{
    map.setView(new View({
        center: proj.fromLonLat([long, lat]),
        zoom: 18
    }));
}
</script>

<div id="map" class="map"></div>

<style>
#map
{
    position: absolute;
    top: 7%;
    bottom: 0;
    /* height: 88%; */
    width: 100%;
    /* overflow: hidden; */
    /* z-index: -1; */
}
</style>
