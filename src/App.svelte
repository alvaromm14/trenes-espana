<script>
  import * as topojson from "topojson-client";
  import { geoConicConformal, geoPath } from "d3-geo";
  import municipalities from "$data/municipalities.json";
  import estaciones from "$data/estaciones.js";
  import rutas from "$data/rutas.js";
  import { scaleLinear } from "d3-scale";
  import { max } from "d3-array";
  import Tooltip from "$components/Tooltip.svelte";

  // --- MAPA ---
  let provincias = topojson.feature(
    municipalities,
    municipalities.objects.provinces,
  ).features;

  let width = 752;
  let height = 600;
  let svgEl;

  $: projection = geoConicConformal()
    .center([-3.7, 40.4])
    .parallels([36, 44])
    .scale(width * 5.1)
    .translate([width / 2.3, height / 2.2]);

  $: path = geoPath(projection);

  // --- ESTADO ---
  let puertoType = "larga";
  let hovered = null;
  let selected = null;

  $: active = selected || hovered;

  function setPuertoType(type) {
    puertoType = type;
    selected = null;
    hovered = null;
  }

  // --- DATOS ---
  let estacionesByName = {};
  for (let e of estaciones) estacionesByName[e.estacion] = e;

  let maxRuta = max(rutas, (r) => r[puertoType]);
  let routeScale = scaleLinear().domain([0, maxRuta]).range([1, 12]);

  $: sizeScale = scaleLinear().domain([0, 32789245]).range([2, 50]);
</script>

<!-- BOTONES -->
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

<!-- CONTENEDOR -->
<div
  class="chart-container"
  on:click={() => {
    selected = null;
  }}
>
  <svg
    bind:this={svgEl}
    viewBox={`0 0 ${width} ${height}`}
    preserveAspectRatio="xMidYMid meet"
    class="map-svg"
    on:click|stopPropagation
  >
    <!-- PROVINCIAS -->
    {#each provincias as provincia}
      <path
        d={path(provincia)}
        fill="white"
        on:click={() => (selected = null)}
      />
    {/each}

    <!-- RUTAS -->
    {#each rutas as r}
      {#if r[puertoType] > 0 && estacionesByName[r.origen] && estacionesByName[r.destino]}
        {@const oEst = estacionesByName[r.origen]}
        {@const dEst = estacionesByName[r.destino]}
        {@const o = projection([oEst.longitude, oEst.latitude])}
        {@const d = projection([dEst.longitude, dEst.latitude])}

        <line
          x1={o[0]}
          y1={o[1]}
          x2={d[0]}
          y2={d[1]}
          stroke="#ffb600"
          stroke-width={routeScale(r[puertoType])}
          stroke-opacity={active &&
          (r.origen === active.estacion || r.destino === active.estacion)
            ? 1
            : active
              ? 0.15
              : 0.65}
          pointer-events="none"
        />
      {/if}
    {/each}

    <!-- PUERTOS -->
    {#each estaciones as puerto}
      {#if puerto[puertoType] >= 1}
        <circle
          cx={projection([puerto.longitude, puerto.latitude])[0]}
          cy={projection([puerto.longitude, puerto.latitude])[1]}
          r={sizeScale(puerto[puertoType])}
          fill="#cc1414"
          fill-opacity={selected
            ? selected === puerto
              ? 1
              : 0.3
            : hovered
              ? hovered === puerto
                ? 1
                : 0.3
              : 1}
          on:mouseover={() => (hovered = puerto)}
          on:mouseout={() => (hovered = null)}
          on:click={() => (selected = puerto)}
        />
      {/if}
    {/each}
  </svg>

  <!-- TOOLTIP SOLO EN HOVER -->
  {#if hovered}
    <Tooltip
      {projection}
      selection={hovered}
      {width}
      {height}
      {puertoType}
      {svgEl}
    />
  {/if}
</div>

<style>
  .map-svg {
    width: 100%;
    height: auto;
  }

  .chart-container {
    width: 100%;
    max-width: 752px;
    aspect-ratio: 752 / 600;
    background-color: #f6efef;
  }

  circle {
    transition:
      r 250ms ease,
      fill 250ms ease,
      fill-opacity 250ms ease,
      stroke-opacity 250ms ease;
    cursor: pointer;
  }

  .button-container {
    width: 100%;
    max-width: 752px;
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
    background-color: #f27979;
    transition: background-color 0.2s ease;
  }

  .button-container button.selected {
    background-color: #cc1414;
    font-weight: bold;
    box-shadow: 0 0 0 1px black;
  }

  .button-container button:hover {
    opacity: 0.85;
  }
</style>
