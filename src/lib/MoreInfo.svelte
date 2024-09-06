<script>
    import { onMount, createEventDispatcher } from "svelte";
    import Line from "svelte-chartjs/src/Line.svelte"

    export let service = {
        id: "bc3e774a",
        name: "Demo Service",
        stalls: "4"
    };

    const dispatch = createEventDispatcher();
    function close()
    {
        dispatch('close', {});
    }

    let tempPlotData = {}
    let humiPlotData = {}
    let pirPlotData  = {}

    const randomRgbColor = () => {
        const r = Math.floor(Math.random() * 256);
        const g = Math.floor(Math.random() * 256);
        const b = Math.floor(Math.random() * 256);
        return `rgb(${r}, ${g}, ${b})`;
    };

    let prevTime = ""
    const getTime = (time) => {
        return time.substring(9, 11) + ":" + 
        time.substring(11, 13) + ":" + time.substring(13, 15);
    };

    function update()
    {
        const url = `http://127.0.0.1:8010/proxy/~/in-cse/in-name/data/${service.id}/la`;
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
                let time = getTime(json["m2m:cin"]["ct"])
                if(time == prevTime) return
                prevTime = time
                tempPlotData.labels = [...tempPlotData.labels, time]
                humiPlotData.labels = [...humiPlotData.labels, time]
                pirPlotData.labels =  [...pirPlotData.labels, time]

                var temp = tempPlotData.datasets;
                var humi = humiPlotData.datasets;
                var pir  = pirPlotData.datasets;

                let stalls = parseInt(service.stalls)
                let using = 0
                for (let i = 0; i < stalls; i++)
                {
                    temp[i].data = [...temp[i].data, value[i]["temperature"]]
                    humi[i].data = [...humi[i].data, value[i]["humidity"]]
                    if(value[i]["pir"] == 1) ++using
                    // pir[i].data = [...pir[i].data, value[i]["pir"]]
                }
                pir[0].data = [...pir[0].data, using]
                tempPlotData.datasets = temp;
                // console.log(tempPlotData);
                humiPlotData.datasets = humi;
                pirPlotData.datasets = pir;
                // console.log(value)
            });
        });



// setTimeout(update, 3000);
    }

    onMount(() => {
        if(service.id == "acfa26c7")
        {
            service.stalls = "2"
        }

        const url = `http://127.0.0.1:8010/proxy/~/in-cse/in-name/data/${service.id}/?rcn=4`;
        const header = {
            'X-M2M-Origin': 'admin:admin',
            'Content-type': 'application/json',
        }
        fetch(url, {
            method: "GET",
            headers: header
        }).then(res => {
            res.json().then((j) => {
                // debugger;
                let values = j["m2m:cnt"]["m2m:cin"];
                let labels = [], pir_labels = []
                // console.log(service)
                let stalls = parseInt(service.stalls)
                let temp = Array(stalls).fill([]);
                let humi = Array(stalls).fill([]);
                let pir = [];
                let k = 0
                values.forEach(value => {
                    k++
                    // if ((k % 10) != 0) return

                    if ((k % 10) == 0)
                    {
                        labels = [...labels, getTime(value["ct"])]
                    }
                    pir_labels = [...pir_labels, getTime(value["ct"])]

                    prevTime = getTime(value["ct"])
                    let obj = JSON.parse(value["con"]);

                    let using = 0
                    for (let i = 0; i < stalls; i++)
                    {
                        if ((k % 10) == 0)
                        {
                            temp[i] = [...temp[i], obj[i]["temperature"]]
                            humi[i] = [...humi[i], obj[i]["humidity"]]
                        }
                        if(obj[i]["pir"] == 1) ++using
                        // pir[i] = [...pir[i], obj[i]["pir"]]
                    }
                    pir = [...pir, using]
                });

                tempPlotData.labels = labels
                tempPlotData.datasets = []

                humiPlotData.labels = labels
                humiPlotData.datasets = []

                pirPlotData.labels = pir_labels
                pirPlotData.datasets = []
                for (let i = 0; i < temp.length; i++)
                {
                    let color = randomRgbColor();
                    let obj = 
                    {
                        label: `Stall ${i}`,
                        fill: true,
                        lineTension: 0.1,
                        // backgroundColor: color,
                        borderColor: color,
                        pointRadius: 0.1,
                        pointHitRadius: 0.5,
                        data: temp[i]
                    }
                    tempPlotData.datasets.push(obj)

                    obj = 
                    {
                        label: `Stall ${i}`,
                        fill: true,
                        lineTension: 0.1,
                        // backgroundColor: color,
                        borderColor: color,
                        pointRadius: 0.1,
                        pointHitRadius: 0.5,
                        data: humi[i]
                    }
                    humiPlotData.datasets.push(obj)
                }
                // for (let i = 0; i < pir.length; i++)
                {
                    let color = "rgb(200, 200, 200)";
                    let obj = 
                    {
                        label: `PIR`,
                        fill: true,
                        lineTension: 0.1,
                        // backgroundColor: color,
                        borderColor: color,
                        pointRadius: 0.1,
                        pointHitRadius: 0.5,
                        data: pir
                    }
                    pirPlotData.datasets.push(obj)
                }
                tempPlotData = tempPlotData
                humiPlotData = humiPlotData
                pirPlotData = pirPlotData
            });
        });
        setInterval(update, 3000);
    });
</script>

<div class="modal active">
    <a href="#close" class="modal-overlay" aria-label="Close"></a>
    <div class="modal-container">
        <div class="modal-header">
            <button id="close" class="btn btn-link float-right" on:click={close}>
                <i class="icon icon-cross"></i>
            </button>
            <h1>{service.name}</h1>
        </div>
        <div class="modal-body">
            <h3>Temperature</h3>
            <Line data={tempPlotData} options={{ responsive: true }}/>
            <h3>Humidity</h3>
            <Line data={humiPlotData} options={{ responsive: true }}/>
            <h3>PIR</h3>
            <Line data={pirPlotData} options={{ responsive: true }}/>
        </div>
        <div class="modal-footer">
        </div>
    </div>
</div>
