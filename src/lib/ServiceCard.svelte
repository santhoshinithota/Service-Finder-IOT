<script>
    import { onMount } from "svelte";
    import MoreInfo from "./MoreInfo.svelte";
    // import * as mqtt from "mqtt/dist/mqtt";
    import mqtt from 'mqtt';

    export let map;
    export let service = {
        id: "demo",
        image: "https://picturepan2.github.io/spectre/img/osx-yosemite-2.jpg",
        name: "Demo Service",
        type: "Bathroom",
        location: ["17.4479092", "78.3493142"],
        stalls: "4"
    };

    let displayMoreInfo = false;
    let canBook = true;
    let canOpen = false;

    let mqttClient = null;

    function update()
    {
        canBook = false;
        if(service.id != "acfa26c7") return;
        canBook = true;

        const url = `http://127.0.0.1:8010/proxy/~/in-cse/in-name/pir/${service.id}/la`;
        const header = {
            'X-M2M-Origin': 'admin:admin',
            'Content-type': 'application/json',
        }
        fetch(url, {
            method: "GET",
            headers: header
        }).then((res) => {
            res.json().then((json) => {
                let value = JSON.parse(json["m2m:cin"]["con"])
                console.log(value)
                if(value["users"].length >= service.stalls)
                {
                    canBook = false;
                }
                canBook = canBook && !value["users"].includes("bhv");
                canOpen = value["users"].includes("bhv");
            });
        });
    }

    function book()
    {
        mqttClient.publish(`book/${service.id}`, "bhv");
        update();
    }

    function open()
    {
        mqttClient.publish(`open/${service.id}`);
        update();
    }

    onMount(() => {
        if(service.id == "acfa26c7")
        {
            mqttClient = mqtt.connect("ws://public.mqtthq.com:8083/mqtt");
        }

        update();
        setInterval(update, 3000);
    });
</script>

<div class="column col-10 card p-0 m-2">
    <div class="card-image" style="height: 75px;">
        <div class="image" style="background: url({service.image})"> </div>
    </div>
    <div class="card-header">
        <button class="btn btn-primary float-right" on:click={() => map.focus(parseFloat(service.location[1]), parseFloat(service.location[0]))}>
            <i class="icon icon-location"></i>
        </button>
        <div class="card-title h5">{service.name}</div>
        <div class="card-subtitle text-gray">{service.type}</div>
    </div>
    <div class="card-footer">
        <button class="btn btn-primary" on:click={() => displayMoreInfo = true}>
            Know More
        </button>
        {#if canBook}
            <button class="btn btn-secondary" on:click={book}>Book</button>
        {:else if canOpen}
            <button class="btn btn-secondary" on:click={open}>Open</button>
        {:else}
            <button class="btn btn-secondary disabled">Can't Book</button>
        {/if}

    </div>
</div>

{#if displayMoreInfo}
    <MoreInfo service={service} on:close={() => displayMoreInfo = false}/>
{/if}

<style>
.image
{
    background-size: cover;
    height: 100%;
    overflow-y: hidden;
}
</style>
