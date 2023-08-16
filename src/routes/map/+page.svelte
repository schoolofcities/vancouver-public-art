<script>
  import { onMount } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import vancouverPublicArt from '../../data/vancouver-public-art.geo.json';
  import vancouverPublicTransit from '../../data/vancouver-transit.geo.json';
  import vancouverBoundary from '../../data/city-of-vancouver-boundary.geo.json';

  let map;
  let publicartStatus = '';

  onMount(() => {
	const maxBounds = [ 
		[-123.3, 49.15], //SW coords
		[-122.6, 49.35] //NE coords
		];
	
    map = new maplibregl.Map({
		container: 'map',
    	style: './vector-tiles-vintage-v3.json',//'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
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
						['get', 'status'],
						'In place',
						'#CC322B', //3F7671, 93677D, 8095D6
						'No longer in place',
						'#223b53', //A1B053
						'Deaccessioned',
						'#EEA950',
						//others
						'#ccc'
					]
				},
				before: 'vancouverPublicTransit',
			});
    });


	// Create pop-up 
	const popup = new maplibregl.Popup({ 
		closeButton: true,
    closeOnClick: true,
    maxWidth: 'none',
    });

	map.on('click', 'vancouverPublicArt', (e) => {
    map.flyTo({
                center: e.features[0].geometry.coordinates,
                zoom: 17
            });
    const coordinates = e.features[0].geometry.coordinates.slice();
    const title = e.features[0].properties.title_of_work;
    const description = e.features[0].properties.descriptionofwork;
    const type = e.features[0].properties.type;
    const status = e.features[0].properties.status;
    const siteaddress = e.features[0].properties.siteaddress;
    const primarymaterial = e.features[0].properties.primarymaterial;
    const photoURL = "https://opendata.vancouver.ca/explore/dataset/public-art/files/";
    const photoID = e.features[0].properties.photourl.id;
    const year = e.features[0].properties.yearofinstallation;

    // Organize popup information
    const htmlContent =
        "<p> <b> <h3>" + title + "</h3> </b> </p>" +
        "<p> <img src='" + photoURL + photoID + "/download/' width=300 height=240> </p>" +
        "<p> <b>Description: </b>" + description + "</p>" +
        "<p> <b>Type: </b>" + type + "</p>" +
        "<p> <b>Current Status: </b>" + status + "</p>" +
        "<p> <b>Primary Material: </b>" + primarymaterial + "</p>" +
        "<p> <b>Address: </b>" + siteaddress + "</p>" +
        "<p> <b>Year of Installation: </b>" + year + "</p>";

		// Populate the popup
		popup.setLngLat(coordinates).setHTML(htmlContent);
    document.getElementById('popup').innerHTML = htmlContent;
    });

  // Update map filter on dropdown change
  function updateMapFilter() {
    let filter;
    
    if (publicartStatus === 'All') {
        filter = null; // No filtering if 'All' is selected
    } else {
        filter = ['==', 'status', publicartStatus];
    }
    map.setFilter('vancouverPublicArt', filter);
    }
    //const filter =
      //publicartStatus === 'All' ? ['has', 'status'] : ['==', 'status', publicartStatus];
    //map.setFilter('vancouverPublicArt', filter);
    //}

    document.getElementById('artstatus').addEventListener('change', (e) => {
      publicartStatus = e.target.value;
      updateMapFilter();
      });  
});
</script>

<main>
  <div id='map'></div>

  <div class='legend'>
    <h4>Vancouver Public Art</h4>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #CC322B;'></span>
      In place
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #223b53;'></span>
      No longer in place
    </div>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #EEA950;'></span>
      Deaccessioned
    </div>
  </div>

  <div class='popup' id='popup'></div>

  <div class='map-overlay-dropdown'>
    <form>
      <fieldset>
          <select id="artstatus">
              <label>Select Public Art Status</label>
              <select id="astatus" name="astatus">
                  <option value="" disabled selected>artwork status is?</option>
                  <option value="In place">In place</option>
                  <option value="No longer in place">No longer in place</option>
                  <option value="Deaccessioned">Deaccessioned</option>
                  <option value="All">Show all</option>
              </select>
          </select>
      </fieldset> 
    </form>
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

  #map {
    height: 100vh;
    width: 100%;
    top: 0;
    left: 0;
    position: relative;
  }

.popup {
	position: absolute;
  top: 2vh;
  right: 15px;
  width: 300px; /* Set a fixed width for the popup */
  max-height: 100vh; /* Calculate the max height based on viewport height */
  overflow-y: scroll; /* Enable vertical scrolling when content overflows */
  font: 15px 'Trade Gothic LT Bold';
  background-color: white;
  padding: 10px;
  border-radius: 5px;
}

.legend {
    position: absolute;
    bottom: 45px;
    left: 10px;
    font: 17px 'Trade Gothic LT Bold';
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

  .map-overlay-dropdown {
    position: relative;
    font: 15px/20px 'Trade Gothic LT Bold';
    background: rgba(249, 249, 249, 0.8);
    top: 30px;
    left: 10px;
    width: 200px;
    margin: 10px 0 0 10px;
    padding: 10px;
    overflow: visible;
}

</style>