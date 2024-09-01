<script>
  import { onMount, onDestroy } from "svelte";
  import mapboxgl from "mapbox-gl";
  import "mapbox-gl/dist/mapbox-gl.css";

  import states from "../data/states.json";
  import stateLabels from "../data/state-labels.json";

  export let mapHeight;
  export let bounds = undefined;
  export let center = undefined;
  export let zoom = 10;
  export let id;

  let map;

  mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_TOKEN;

  // Initialize the map when the component mounts
  onMount(() => {
    // Initialize the map

    let mapConfig = {
      container: `map-${id}`, // container ID
      style: "mapbox://styles/agwaterdesk/cm0fs6ipr01uh01qq820jbz0o",
      padding: { top: 50, bottom: 50, left: 50, right: 50 },
      interactive: false,
    };

    if (bounds) {
      mapConfig = { ...mapConfig, bounds };
    }

    if (center) {
      mapConfig = { ...mapConfig, center, zoom };
    }

    map = new mapboxgl.Map(mapConfig);

    if (bounds) {
      // Adjust map to the bounds with padding
      map.fitBounds(bounds, { padding: 30 });
    }

    // Add markers and labels to the map
    stateLabels.forEach((marker) => {
      // Create marker element
      const markerEl = document.createElement("div");
      markerEl.className = "marker";

      // Add marker element to map
      new mapboxgl.Marker(markerEl)
        .setLngLat([marker.longitude, marker.latitude])
        .addTo(map);

      // Create label element
      const labelEl = document.createElement("div");
      labelEl.className = "state-label";
      labelEl.innerText = marker.label;

      // Add label element to map at the same position as the marker
      new mapboxgl.Marker(labelEl)
        .setLngLat([marker.longitude, marker.latitude])
        .setOffset([15, 0]) // Adjust the offset to position the label next to the marker
        .addTo(map);
    });

    // Add the GeoJSON polygon layer to the map
    map.on("load", () => {
      const layers = map.getStyle().layers;
      // Find the index of the first symbol layer in the map style.
      let firstSymbolId;
      for (const layer of layers) {
        if (layer.type === "symbol") {
          firstSymbolId = layer.id;
          break;
        }
      }

      map.addSource("states", {
        type: "geojson",
        data: states,
      });

      map.addLayer(
        {
          id: "states-outline-layer",
          type: "line",
          source: "states",
          layout: {},
          paint: {
            "line-color": "#FF6542",
            "line-width": 1.5,
          },
        },
        firstSymbolId
      );
    });
  });

  // Cleanup map resources when the component is destroyed
  onDestroy(() => {
    if (map) {
      map.remove();
    }
  });
</script>

<div id="map-{id}" class="map" style:--map-height="{mapHeight}px"></div>

<style lang="scss">
  :global {
    .map {
      width: 100%;
      height: var(--map-height);
    }

    .state-label {
      color: #616161;
      font-size: 16px;
      letter-spacing: 4px;
    }

    #map-zoom {
      .mapboxgl-ctrl-bottom-left,
      .mapboxgl-ctrl-bottom-right {
        opacity: 0;
      }
    }
  }
</style>
