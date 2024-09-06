<script>
    import { fly } from 'svelte/transition';

    export let visible = false;
    export let map;
    import ServiceCard from "./ServiceCard.svelte";

    let data = [];
    export function Display(tag)
    {
        data = []
        visible = true;
        const url = `http://127.0.0.1:8010/proxy/~/in-cse/in-name/services/${tag}/?rcn=4`;
        const header = {
            'X-M2M-Origin': 'admin:admin',
            'Content-type': 'application/json',
        }
        fetch(url, {
            method: "GET",
            headers: header
        }).then(res => {
            res.json().then((j) => {
                let services = j["m2m:cnt"]["m2m:cin"];
                services.forEach(service => {
                    let obj = JSON.parse(service["con"])
                    data = [...data, obj]
                });
            });
        });
    }

    export function Hide()
    {
        data = []
        visible = false;
    }
</script>

{#if visible}
<div id="services" class="columns col-oneline" transition:fly = "{{ y:200, duration: 500 }}">
    <button id="close" class="btn btn-link" on:click={() => Hide()}>
        <i class="icon icon-cross"></i>
    </button>
    {#each data as service}
        <ServiceCard map={map} service={service} />
    {/each}
</div>
{/if}

<style>
#services
{
    background: white;
    position: absolute;
    bottom: 0%;
    width: 100%;
    padding-bottom: 5%;
    padding-left: 5%;
    padding-top: 5%;
    margin-left: 3px;

    border-top-left-radius: 50px;
    border-top-right-radius: 50px;

    box-shadow: -2px -2px 42px 4px rgba(0,0,0,0.35);
    -webkit-box-shadow: -2px -2px 42px 4px rgba(0,0,0,0.35);
    -moz-box-shadow: -2px -2px 42px 4px rgba(0,0,0,0.35);

    overflow-y: hidden;
}
#close
{
    position: fixed;
    right: 30px;
    /* top: -4px; */
    bottom: 250px;
}
</style>
