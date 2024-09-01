<script>
  import Window from "./components/Window.svelte";
  import Map from "./components/Map.svelte";
  import Overlay from "./components/Overlay.svelte";
  import { MousePointer } from "lucide-svelte";
  import mapboxgl from "mapbox-gl";

  import towns from "./data/towns.json";

  // Handle responsive iframes for embeds
  import pym from "pym.js";

  new pym.Child({
    polling: 500,
  });




  let overlayInfo;
  let marker = null;

  function reset() {
    overlayInfo = undefined;
    marker.remove();
  }

  // Function to calculate bounds based on markers
  function getBounds(markers) {
    const bounds = new mapboxgl.LngLatBounds();
    markers.forEach((marker) =>
      bounds.extend([marker.longitude, marker.latitude])
    );
    return bounds;
  }

  let townBounds = getBounds(towns);
</script>

<Window />
<!-- Outer div must have class 'chart-container' don't change -->
<div class="chart-container">
  <h1 class="headline">
    A river town isolated by winding, changing Mississippi River
  </h1>
  <p class="dek">
    <span class="town orange" /> Corona is a community isolated from its state
    due to the winding path of the Mississippi that forms the
    <span class="border" />
    border between Arkansas, Tennessee and Mississippi.
    <span class="town white" />Indicate other towns along the Mississippi River.
  </p>

  <p class="sr-only">A map showing river towns along the border of Arkansas, Tennessee and Mississippi.</p>

  <div id="g-viz">
    <Map
      mapHeight="300"
      center={[-90.07454488249725, 35.421939203160846]}
      zoom="10.1"
      id="zoom"
    />

    <Map mapHeight="600" bounds={townBounds} id="overview" />
  </div>


    <div class="credit">
      Graphic by Jared Whalen /
      <a target="_blank" href="https://agwaterdesk.org/">Ag & Water Desk</a>
    </div>

</div>

<style lang="scss">
  .chart-container {
    max-width: 800px;
    width: 100%;
    padding: 1rem;
  }

  #g-viz {
    position: relative;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  :global {
    .town {
      $size: 12px;
      margin-left: calc($size + 4px);
      position: relative;

      &.orange {
        &::after {
          background: #f1b82d;
        }
      }

      &.white {
        &::after {
          background: #fff;
        }
      }

      &::after {
        width: $size;
        height: $size;
        content: "";
        border: 1px;
        position: absolute;
        left: calc($size * -1 - 2px);
        top: 53%;
        transform: translateY(-50%);
        border: 2px solid;
        border-radius: 100%;
      }
    }

    .border {
      $size: 12px;
      margin-left: calc($size + 4px);
      position: relative;

      &::after {
        width: $size;
        height: 1.5px;
        content: "";
        border: 1px;
        position: absolute;
        left: calc($size * -1 - 2px);
        top: 55%;
        transform: translateY(-50%);
        background: #ff6542;
      }
    }
  }
</style>
