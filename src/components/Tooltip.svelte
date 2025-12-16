<script>
    import { fly, fade } from "svelte/transition";

    export let selection;
    export let projection;
    export let svgEl;
    export let puertoType;

    let tooltipWidth = 0;
    let tooltipHeight = 0;

    const xOffset = 10;
    const yOffset = 10;

    // Coordenadas SVG
    $: [svgX, svgY] = projection([selection.longitude, selection.latitude]);

    // Bounding box real del SVG
    $: svgRect = svgEl?.getBoundingClientRect();

    // Escalas SVG → pantalla
    $: scaleX = svgRect ? svgRect.width / svgEl.viewBox.baseVal.width : 1;

    $: scaleY = svgRect ? svgRect.height / svgEl.viewBox.baseVal.height : 1;

    // Coordenadas reales del punto
    $: screenX = svgRect ? svgRect.left + svgX * scaleX : 0;

    $: screenY = svgRect ? svgRect.top + svgY * scaleY : 0;

    // ⚠️ GUARDIA: no calcular hasta tener tamaño
    $: canPosition = tooltipWidth > 0 && tooltipHeight > 0 && svgRect;

    // Horizontal: derecha o izquierda (TU lógica)
    $: xPosition = canPosition
        ? screenX + tooltipWidth + xOffset < svgRect.right
            ? screenX + xOffset
            : screenX - tooltipWidth - xOffset
        : 0;

    // Vertical: abajo o arriba (TU lógica)
    $: yPosition = canPosition
        ? screenY + tooltipHeight + yOffset < svgRect.bottom
            ? screenY + yOffset
            : screenY - tooltipHeight - yOffset
        : 0;
</script>

<div
    class="tooltip"
    in:fly={{ y: 8, duration: 120 }}
    out:fade
    style="
    position: fixed;
    left: {xPosition}px;
    top: {yPosition}px;
    opacity: {canPosition ? 1 : 0};
  "
    bind:clientWidth={tooltipWidth}
    bind:clientHeight={tooltipHeight}
>
    <h1>{selection.estacion}</h1>

    <div class="info">
        <span
            class="cambio"
            style="
        background: #cc1414;
        color: white;
        font-weight: 800;
      "
        >
            {selection[puertoType] > 100
                ? (
                      Math.round(selection[puertoType] / 100) * 100
                  ).toLocaleString("es-ES")
                : selection[puertoType].toLocaleString("es-ES")}
            pasajeros
        </span>
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
            top 60ms ease,
            left 60ms ease,
            opacity 80ms ease;
        z-index: 10;
    }

    h1 {
        font-size: 1rem;
        font-weight: 500;
        margin-bottom: 3px;
    }

    .info {
        display: flex;
        column-gap: 8px;
        align-items: center;
    }

    .cambio {
        font-size: 11px;
        padding: 3px;
        border-radius: 3px;
        font-weight: 600;
    }
</style>
