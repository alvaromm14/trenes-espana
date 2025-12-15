<script>
  import * as topojson from "topojson-client";
  import { geoConicConformal, geoPath } from "d3-geo";
  import { draw } from "svelte/transition";
  import municipalities from "$data/municipalities.json";
  import estaciones from "$data/estaciones.js";
  import rutas from "$data/rutas.js";
  import { scaleLinear } from "d3-scale";
  import { max, min } from "d3-array";
  import Tooltip from "$components/Tooltip.svelte";

  let provincias = topojson.feature(
    municipalities,
    municipalities.objects.provinces,
  ).features;

  let width = 752;
  let height = 600;

  $: projection = geoConicConformal()
    .center([-3.7, 40.4])
    .parallels([36, 44])
    .scale(width * 5.1)
    .translate([width / 2.3, height / 2.2]);

  $: path = geoPath(projection);

  let puertoType = "larga";

  function setPuertoType(type) {
    puertoType = type;
  }

  let estacionesByName = {};
  for (let e of estaciones) {
    estacionesByName[e.estacion] = e;
  }

  let maxRuta = max(rutas, (r) => r[puertoType]);

  let routeScale = scaleLinear().domain([0, maxRuta]).range([1, 12]);

  const colorRanges = {
    larga: ["#f27979", "#cc1414"],
    media: ["#f27979", "#cc1414"],
  };

  const borderRanges = {
    larga: ["#b31212", "#800000"],
    media: ["#b31212", "#800000"],
  };

  $: sizeScale = scaleLinear().domain([0, 32789245]).range([2, 50]);

  $: colorScale = scaleLinear()
    .domain([0, max(estaciones, (d) => d[puertoType])])
    .range(colorRanges[puertoType]);

  $: colorBorderScale = scaleLinear()
    .domain([0, max(estaciones, (d) => d[puertoType])])
    .range(borderRanges[puertoType]);

  let selection;
</script>

<div class="button-container">
  <button
    on:click={() => setPuertoType("larga")}
    class:selected={puertoType === "larga"}
  >
    Larga distancia
  </button>

  <button
    on:click={() => setPuertoType("media")}
    class:selected={puertoType === "media"}
  >
    Media distancia
  </button>
</div>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
  class="chart-container"
  on:click={() => {
    selection = null;
  }}
>
  <svg {width} {height} on:click|stopPropagation>
    {#each provincias as provincia}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <path d={path(provincia)} fill="white" />
    {/each}
    {#each rutas as r}
      {#if r[puertoType] > 0 && estacionesByName[r.origen] && estacionesByName[r.destino]}
        {@const origen = estacionesByName[r.origen]}
        {@const destino = estacionesByName[r.destino]}

        {@const o = projection([origen.longitude, origen.latitude])}
        {@const d = projection([destino.longitude, destino.latitude])}

        <line
          x1={o[0]}
          y1={o[1]}
          x2={d[0]}
          y2={d[1]}
          stroke="#e6a400"
          stroke-width={routeScale(r[puertoType])}
          stroke-opacity={selection &&
          (r.origen === selection.estacion || r.destino === selection.estacion)
            ? 1
            : selection
              ? 0.15
              : 0.65}
          pointer-events="none"
        />
      {/if}
    {/each}
    {#each estaciones as puerto}
      {#if puerto[puertoType] >= 1}
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <!-- svelte-ignore a11y-mouse-events-have-key-events -->
        <circle
          cx={projection([puerto.longitude, puerto.latitude])[0]}
          cy={projection([puerto.longitude, puerto.latitude])[1]}
          r={sizeScale(puerto[puertoType])}
          fill={colorScale(puerto[puertoType])}
          fill-opacity={selection
            ? selection === puerto
              ? "100%"
              : "30%"
            : "70%"}
          stroke={colorBorderScale(puerto[puertoType])}
          stroke-width="0.8"
          stroke-opacity={selection
            ? selection === puerto
              ? "100%"
              : "50%"
            : "100%"}
          on:mouseover={() => {
            selection = puerto;
          }}
          on:focus={() => {
            selection = puerto;
          }}
          tabindex="0"
          on:mouseout={() => {
            selection = null;
          }}
        />
      {/if}
    {/each}
  </svg>

  {#if selection}
    <Tooltip
      {projection}
      {selection}
      {width}
      {height}
      {puertoType}
      {colorRanges}
    />
  {/if}
</div>

<style>
  .chart-container {
    width: 752px;
    height: 600px;
    display: flex;
    background-color: #f6efef;
  }
  circle {
    transition:
      r 250ms ease,
      fill 250ms ease,
      fill-opacity 250ms ease,
      stroke-opacity 250ms ease;
    outline: none;
  }

  .button-container {
    width: 752px;
    display: flex;
    justify-content: center;
    margin-bottom: 8px;
    background-color: #f6efef;
  }

  .button-container button {
    margin: 0 3px;
    padding: 4px 8px;
    font-size: 14px;
    cursor: pointer;
    color: white;
    border: none;
    border-radius: 8px;
    background-color: #f27979; /* NO seleccionado */
    transition: background-color 0.2s ease;
  }

  .button-container button.selected {
    background-color: #cc1414; /* SELECCIONADO */
    font-weight: bold;
    box-shadow: 0 0 0 1px black;
  }

  .button-container button:hover {
    opacity: 0.85;
  }
</style>
