<script>
  import { onMount } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import Navbar from '../../lib/Navbar.svelte';
  import vancouverPublicArt from '../../data/vancouver-public-art.geo.json';
  import vancouverPublicTransit from '../../data/vancouver-transit.geo.json';

  let map;
  let pageWidth;

  onMount(() => {
    map = new maplibregl.Map({
      container: 'map',
      style: 'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
      center: [-123.109, 49.257], // starting position
      zoom: 11.5, // starting zoom
      projection: 'globe',
      scrollZoom: true,
      attributionControl: false,
    });

    // Adding zoom and rotation controls to the map
    map.addControl(new maplibregl.NavigationControl(), 'top-left');

    // Adding scale bar to the map
    let scale = new maplibregl.ScaleControl({
      maxWidth: 100,
      unit: 'metric',
    });
    map.addControl(scale, 'bottom-left');

    // Adding additional layers from geojson
    map.on('load', function () {
      const layers = map.getStyle().layers;

      // Find the index of the first symbol layer in the map style.
      let firstSymbolId;
      for (const layer of layers) {
        if (layer.type === 'symbol') {
          firstSymbolId = layer.id;
          break;
        }
      }

	map.addSource('vancouverPublicTransit', {
        type: 'geojson',
        data: vancouverPublicTransit
	});
	
    map.addSource('vancouverPublicArt', {
    	type: 'geojson',
        data: vancouverPublicArt
    });

	map.addLayer({
				'id': 'vancouverPublicTransit',
				'type': 'line',
				'source': 'vancouverPublicTransit',
				'layout': {},
				'paint': {
					'line-color': '#dfc8c8', 
					'line-width': 3,
					//'line-dasharray': [5,2]
				}
			}, 'rail');

	map.addLayer({
				'id': 'vancouverPublicArt',
				'type': 'circle',
				'source': 'vancouverPublicArt',
				'layout': {},
				'paint': {
					'circle-color': '#3F7671', 
					'circle-radius': [
						"interpolate",
						["linear"],
						["zoom"],
						8,
						2,
						16,
						10
					]
				},
				before: 'vancouverPublicTransit',
			}, 'rail');
    });

	// Create pop-up
	const popup = new maplibregl.Popup({
		closeButton: false,
        closeOnClick: false
        });

	map.on('mouseenter', 'vancouverPublicArt', (e) => {
        map.getCanvas().style.cursor = 'pointer';
	
		const coordinates = e.features[0].geometry.coordinates.slice();
		const type = e.features[0].properties.type;
		const photo = e.features[0].properties.photourl;


		// Populate the popup
		popup.setLngLat(coordinates).setHTML(photo).addTo(map);
        });

		map.on('mouseleave', 'vancouverPublicArt', () => {
            map.getCanvas().style.cursor = '';
            popup.remove();
        });
  });
</script>

<main>
  <div class='bar'>
    <Navbar />
  </div>

  <div id='map'></div>
</main>

<style>
  @font-face {
    font-family: TradeGothicBold;
    src: url('./assets/Trade Gothic LT Bold.ttf');
  }

  :root {
    font-family: 'Roboto', sans-serif;
  }

  .bar {
    text-align: center;
  }

  #map {
    height: calc(100vh - 60px); /* calculate height of the screen minus the heading */
    width: 100%;
    top: 0;
    left: 0;
    position: relative;
  }

.maplibregl-popup {
    max-width: 400px;
    font: 12px/20px 
}
</style>