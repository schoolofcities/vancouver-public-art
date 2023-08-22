<script>
  import { onMount, afterUpdate } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import vancouverPublicArt from '../../data/vancouver-public-art.geo.json';
  import vancouverPublicTransit from '../../data/vancouver-transit.geo.json';
  import vancouverBoundary from '../../data/city-of-vancouver-boundary.geo.json';
  import notVancouverPolygon from '../../data/not-vancouver-polygon.geo.json';

  let map;
  let popupContent = '';
  
  onMount(() => {
    const maxBounds = [ 
      [-123.8, 49.0], //SW coords
      [-122.6, 49.5] //NE coords
    ];
	
    map = new maplibregl.Map({
		container: 'map',
      style: './vector-tiles-vintage-v4.json',//'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
      center: [-123.109, 49.257], // starting position
      zoom: 11.3, // starting zoom;
      minZoom: 8,
      maxZoom: 17,
      maxBounds: maxBounds, //maximum and minimum scroll bounds
      projection: 'globe',
      scrollZoom: true,
      attributionControl: false,
		});

    // Adding scale bar to the map
    let scale = new maplibregl.ScaleControl({
      maxWidth: 100,
      unit: 'metric',
    });
    map.addControl(scale, 'bottom-left');

    // Adding zoom and rotation controls to the map
    map.addControl(new maplibregl.NavigationControl(), 'bottom-left');

    

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

  map.addSource('notVancouverPolygon', {
    type: 'geojson',
      data: notVancouverPolygon
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
					'line-color': '#F1C500', 
					'line-width': 1.5
					// 'line-dasharray': [1,1]
				}
			}, 'highway_name_major');

	map.addLayer({
				'id': 'vancouverBoundary',
				'type': 'line',
				'source': 'vancouverBoundary',
				'layout': {},
				'paint': {
          'line-opacity': 0.64,
					'line-color': '#575870', 
					'line-width': 4,
        
					'line-dasharray': [6,0.5,1,0.5,1,0.5]}
			}, 'highway_name_major');

    map.addLayer({
      'id': 'notVancouverPolygon',
      'type': 'fill',
      'source': 'notVancouverPolygon',
      'layout': {},
      'paint': {
        'fill-color': 'white',
        'fill-opacity': 0.42
    }});

	map.addLayer({
				'id': 'vancouverPublicArt',
				'type': 'circle',
				'source': 'vancouverPublicArt',
				'layout': {},
				'paint': {
					'circle-radius': [
            'interpolate',
            ['linear'],
            ['zoom'],
            0, 1,
            12, 5,
            18, 12
        ],
          'circle-stroke-color': 'white', // Stroke color
          'circle-stroke-width': [
            'interpolate',
            ['linear'],
            ['zoom'],
            12, 1,
            20, 3
        ], 
					'circle-color': [
						'match',
						['get', 'status'],
						'In place',
						'#6D247A', //3F7671, 93677D, 8095D6
						'No longer in place',
						'#DC4633', //A1B053
						'Deaccessioned',
						'#007FA3',
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
    const photoID = JSON.parse(e.features[0].properties.photourl).id;
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
    popupContent = htmlContent;
    //document.getElementById('popup').innerHTML = htmlContent;
    });

    // Update map filter on dropdown change
    document.getElementById('thelist').addEventListener('change', (e) => {
      //if the selected dropdown element has index 1, filter the 'vancouverPublicArt' layer to show only public art with the status of in place
      if (document.getElementById("thelist").selectedIndex===1) {
          map.setFilter('vancouverPublicArt', ['==', ['get', 'status'], 'In place']);
      } 
      //if the selected dropdown element has index 2, filter the 'vancouverPublicArt' layer to show only public art with the status of no longer in place
      else if (document.getElementById("thelist").selectedIndex===2) {
          map.setFilter('vancouverPublicArt', ['==', ['get', 'status'], 'No longer in place']);
      } 
      //if the selected dropdown element has index 3, filter the 'vancouverPublicArt' layer to show only public art with the status of deaccessioned
      else if (document.getElementById("thelist").selectedIndex===3) {
          map.setFilter('vancouverPublicArt', ['==', ['get', 'status'], 'Deaccessioned']);
      } 
      //if the selected dropdown element has index 0 , remove the 'vancouverPublicArt' layer filter to show all public art
      else if (document.getElementById("thelist").selectedIndex===0) {
          map.setFilter('vancouverPublicArt', null);
      } 
    });


});
</script>

<main>
  <div id='map'></div>

  <div class='legend'>
    <h1>Vancouver Public Art</h1>
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

  <div class='popup' id='popup'>
  {#if popupContent}
    <p>{@html popupContent}</p>
  {/if}
  </div>

  <div class='map-overlay-dropdown'>
    <form>
              <label>The artwork status is:</label>
              <select id="thelist">
                  <option value="1">Show all</option>
                  <option value="2">In place</option>
                  <option value="3">No longer in place</option>
                  <option value="4">Deaccessioned</option>
              </select>
    </form>
   </div>
</main>

<style>
  @font-face {
    font-family: TradeGothicBold;
    src: url('../../assets/Trade Gothic LT Bold.ttf');
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
  top: 160px;
  left: 10px;
  width: 300px; /* Set a fixed width for the popup */
  max-height: 93vh; /* Calculate the max height based on viewport height */
  overflow-y: scroll; /* Enable vertical scrolling when content overflows */
  font: 15px TradeGothicBold;
  background-color: white;
  padding: 0px;
  padding-left: 10px;
  padding-right: 10px;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
}

.legend {
    position: absolute;
    top: 10px;
    left: 10px;
    width: 300px;
    font-size: 17px;
    font-family: TradeGothicBold;
    background-color: rgba(255, 255, 255, 0.75);
    color: #1E3765;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
  }

  h1 {
    font-size: 25px;
    padding: 0px;
    padding-bottom: 10px;
    margin: 0px;
    color: #1E3765;
    text-decoration: underline;
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
    position: absolute;
    font: 17px/20px 'Trade Gothic LT Bold';
    background: rgba(249, 249, 249, 1);
    height: 45px;
    bottom: 195px;
    width: 157px;
    margin: 10px 0 0 10px;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
    overflow: visible;
}

</style>