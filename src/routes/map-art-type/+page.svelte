<script>
  import { onMount } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import Navbar from '../../lib/Navbar.svelte';
  import vancouverPublicArt from '../../data/vancouver-public-art.geo.json';
  import vancouverPublicTransit from '../../data/vancouver-transit.geo.json';
  import vancouverBoundary from '../../data/city-of-vancouver-boundary.geo.json';

  let map;
  let pageWidth;

  onMount(() => {
    const maxBounds = [ 
		[-123.3, 49.15], //SW coords
		[-122.6, 49.35] //NE coords
		];
	
    map = new maplibregl.Map({
		container: 'map',
    	style: './vector-tiles-vintage-v1-1.json',//'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
    	center: [-123.109, 49.257], // starting position
    	zoom: 11.3, // starting zoom
    	maxBounds: maxBounds, //maximum and minimum scroll bounds
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

	map.addSource('vancouverBoundary', {
    	type: 'geojson',
        data: vancouverBoundary
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
					'line-width': 3
					//'line-dasharray': [5,2]
				}
			});

	map.addLayer({
				'id': 'vancouverBoundary',
				'type': 'line',
				'source': 'vancouverBoundary',
				'layout': {},
				'paint': {
					'line-color': '#575870', 
					'line-width': 3}
			});

	map.addLayer({
				'id': 'vancouverPublicArt',
				'type': 'circle',
				'source': 'vancouverPublicArt',
				'layout': {},
				'paint': {
					'circle-radius': 5,
					'circle-color': [
                        'match',
						['get', 'type'],
						'Sculpture',
						'#CC322B',
						'Memorial or monument',
						'#036A5F',
            'Mural',
            '#EEA950',
            'Site-integrated work',
            '#223B53',
            'Socially engaged art',
            '#8C6F56',
            'Media work',
            '#390493',
            'Two-dimensional artwork',
            '#813E41',
            'Mosaic',
            '#824F26',
            'Relief',
            '#717A70',
            'Totem pole',
            '#2086C2',
            'Gateway',
            '#FD85E0',
            'Fountain or water feature',
            '#E6E679',
            'Welcome figure',
            '#F4978E',
            'Figurative',
            '#7EC629',
						//Other
						'#ccc'
					]
				},
				before: 'vancouverPublicTransit',
			});
    });

	// Create pop-up 
	const popup = new maplibregl.Popup({ 
		closeButton: false,
        closeOnClick: false
        });

	map.on('mouseenter', 'vancouverPublicArt', (e) => {
        map.getCanvas().style.cursor = 'pointer';
	
		const coordinates = e.features[0].geometry.coordinates.slice();
		const title = e.features[0].properties.title_of_work; 
		const projectstatement = e.features[0].properties.artistprojectstatement;
		const type = e.features[0].properties.type;
		const status = e.features[0].properties.status;
		const siteaddress = e.features[0].properties.siteaddress;
		const primarymaterial = e.features[0].properties.primarymaterial;
		const photo = e.features[0].properties.photourl;
		const year = e.features[0].properties.yearofinstallation;

		// Organize popup infomration
		const htmlContent = "<p> <b> <h3>" + title + "</h3> </b> </p>" + 
							"<p>" + projectstatement + "</p>" + 
							"<p> <b>Type: </b>" + type + "</p>" + 
							"<p> <b>Current Status: </b>" + status + "</p>" + 
							"<p> <b>Primary Material: </b>" + primarymaterial + "</p>" +
							"<p> <b>Address: </b>" + siteaddress + "</p>" +
							"<p> <b>Year of Installation: </b>" + year + "</p>" +
							"<p> <b>Photo: </b> <br> <img src='" + photo + "'> </p>"

		// Populate the popup
		popup.setLngLat(coordinates).setHTML(htmlContent).addTo(map);
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

  <div class='legend'>
    <h4>Legend</h4>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #7EC629;'></span>
      Figurative
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #E6E679;'></span>
      Fountain or water feature
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #FD85E0;'></span>
      Gateway
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #036A5F;'></span>
      Memorial or monument
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #390493;'></span>
      Media work
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #824F26;'></span>
      Mosaic
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #EEA950;'></span>
      Mural
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #717A70;'></span>
      Relief
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #CC322B;'></span>
      Sculpture
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #223B53;'></span>
      Site-integrated work
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #8C6F56;'></span>
      Socially engaged art
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #2086C2;'></span>
      Totem pole
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #813E41;'></span>
      Two-dimensional artwork
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #F4978E;'></span>
      Welcome figure
    </div>
  </div>
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

.legend {
    position: absolute;
    bottom: 20px;
    right: 20px;
    background-color: white;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
  }

  .legend h4 {
    margin-bottom: 10px;
  }

  .legend-item {
    display: flex;
    align-items: center;
    margin-bottom: 5px;
  }

  .legend-color {
    width: 13px;
    height: 13px;
    margin-right: 5px;
    border-radius: 50%;
  }
</style>