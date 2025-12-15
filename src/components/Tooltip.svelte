<script>
    export let selection;
    export let width;
    export let height;
    export let projection;
    export let puertoType;
    export let colorRanges;

    import { fly, fade } from "svelte/transition";

    let tooltipWidth;
    let tooltipHeight;

    const xOffset = 10;
    const yOffset = 10;

    // Posición base del punto
    $: [pointX, pointY] = projection([selection.longitude, selection.latitude]);

    // X: derecha del punto, o izquierda si se sale
    $: xPosition =
        pointX + tooltipWidth + xOffset < width
            ? pointX + xOffset + 10
            : pointX - tooltipWidth - xOffset;

    // Y: debajo del punto, o arriba si se sale
    $: yPosition =
        pointY + tooltipHeight + yOffset < height
            ? pointY + yOffset
            : pointY - tooltipHeight - yOffset;
</script>

<div
    class="tooltip"
    in:fly={{ y: 8, duration: 120 }}
    out:fade
    style="
        position: absolute;
        left: {xPosition}px;
        top: {yPosition}px;
    "
    bind:clientWidth={tooltipWidth}
    bind:clientHeight={tooltipHeight}
>
    <h1>{selection.estacion}</h1>
    <h2>
        {selection.estacion === selection.estacion ? "" : selection.estacion}
    </h2>
    <div class="info">
        <span
            class="cambio"
            style="background:{colorRanges[
                puertoType
            ][1]}; color:white; font-weight:800"
            >{selection[puertoType] > 100
                ? (
                      Math.round(selection[puertoType] / 100) * 100
                  ).toLocaleString("es-ES")
                : selection[puertoType].toLocaleString("es-ES")} pasajeros</span
        >
    </div>
</div>

<style>
    .tooltip {
        background-color: white;
        box-shadow: 2px 3px 8px rgba(0, 0, 0, 0.15);
        padding: 8px 6px;
        border-radius: 3px;
        pointer-events: none;
        transition:
            top 50ms ease,
            left 50ms ease;
    }

    .info {
        display: flex;
        justify-content: space-between;
        column-gap: 8px;
        align-items: center;
    }

    .cambio {
        font-size: 11px;
        padding: 3px;
        border-radius: 3px;
        font-weight: 600;
    }

    h1 {
        font-size: 1rem;
        font-weight: 500;
        margin-bottom: 3px;
    }

    h2 {
        font-size: 0.75rem;
        font-weight: 400;
        font-style: italic;
        margin-bottom: 5px;
    }

    .tooltip {
        transform: translateZ(0);
    }
</style>
