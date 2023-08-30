<script>
  import { onMount, afterUpdate } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import vancouverPublicArt from '../../data/vancouver-public-art.geo.json';
  import vancouverPublicTransit from '../../data/vancouver-transit.geo.json';
  import vancouverBoundary from '../../data/city-of-vancouver-boundary.geo.json';
  import notVancouverPolygon from '../../data/not-vancouver-polygon.geo.json';

  let map;
  let popupContent = false;

  function hidePopup() {
    popupContent = false;
  }

  let coordinates;
  let title;
  let description;
  let type;
  let status;
  let siteaddress;
  let primarymaterial;
  const photoURL = "https://opendata.vancouver.ca/explore/dataset/public-art/files/";
  let photoID;
  let year;
  
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
    map.addControl(new maplibregl.NavigationControl(), 'top-right');

    

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
					'line-width': 2,
					'line-dasharray': [3,1]
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
						'#DC4633',
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

    map.on('mouseenter', 'vancouverPublicArt', () => {
    map.getCanvas().style.cursor = 'pointer';
});

map.on('mouseleave', 'vancouverPublicArt', () => {
    map.getCanvas().style.cursor = '';
});

	map.on('click', 'vancouverPublicArt', (e) => {

    
      map.flyTo({
        center: e.features[0].geometry.coordinates,
        zoom: 17
      });
    
    $: coordinates = e.features[0].geometry.coordinates.slice();
    $: title = e.features[0].properties.title_of_work;
    $: description = e.features[0].properties.descriptionofwork;
    $: type = e.features[0].properties.type;
    $: status = e.features[0].properties.status;
    $: siteaddress = e.features[0].properties.siteaddress;
    $: primarymaterial = e.features[0].properties.primarymaterial;
    // $: photoURL = "https://opendata.vancouver.ca/explore/dataset/public-art/files/";
    $: photoID = JSON.parse(e.features[0].properties.photourl).id;
    $: year = e.features[0].properties.yearofinstallation;

    // Organize popup information
    // const htmlContent =
        // "<h1 id='pt'>" + title + "</h1>" +
        // "<p> <img src='" + photoURL + photoID + "/download/' width=300 height=240> </p>" +
        // "<p> <b>Description: </b>" + description + "</p>" +
        // "<p> <b>Type: </b>" + type + "</p>" +
        // "<p> <b>Current Status: </b>" + status + "</p>" +
        // "<p> <b>Primary Material: </b>" + primarymaterial + "</p>" +
        // "<p> <b>Address: </b>" + siteaddress + "</p>" +
        // "<p> <b>Year of Installation: </b>" + year + "</p>";

		// Populate the popup
		popup.setLngLat(coordinates);
    popupContent = true;
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

$: if (popupContent) {
  console.log(photoURL + "/" + photoID + "/download/");
}

</script>

<main>
  <div id='map'></div>

  <div class='legend'>
    <h1>Vancouver Public Art Map</h1>
    <div class='legend-item'>
      <span class='legend-color' style='background-color: #6D247A;'></span>
      <span class='legend-text'>In place &nbsp;</span>
      <span class='legend-color' style='background-color: #DC4633;'></span>
      <span class='legend-text'>No Longer In Place / Deaccessioned</span>
    </div>
    <p id="info">
      Map created by <a href="https://www.linkedin.com/in/irene-kcc/">Irene Chang</a> and <a href="https://jamaps.github.io/about.html">Jeff Allen</a> at the <a href="https://schoolofcities.utoronto.ca/">School of Cities</a> with data from the  <a href="https://opendata.vancouver.ca/explore/dataset/public-art/information/">City of Vancouver</a>. More on <a href="https://github.com/schoolofcities/vancouver-public-art">GitHub</a>.
    </p>
  </div>

  <div class='popup'>
    {#if popupContent}
      <div id="hide" on:click={hidePopup}>Click Here To Hide Content</div>
      <h2>{title}</h2>
      <p><img src={photoURL + "/" + photoID + "/download/"} width=300 height=240></p>
      <p> <span id="subtitle">Type: </span>{type}</p>
      <p>  <span id="subtitle">Description:  </span>{description}</p>
      <p>  <span id="subtitle">Current Status:  </span>{status}</p>
      <p>  <span id="subtitle">Primary Material:  </span>{primarymaterial}</p>
      <p>  <span id="subtitle">Address:  </span>{siteaddress}</p>
      <p>  <span id="subtitle">Year of Installation:  </span>{year}</p>
    {/if}
  </div>

  <!-- <div class='map-overlay-dropdown'>
    <form>
              <label>The artwork status is:</label>
              <select id="thelist">
                  <option value="1">Show all</option>
                  <option value="2">In place</option>
                  <option value="3">No longer in place</option>
                  <option value="4">Deaccessioned</option>
              </select>
    </form>
   </div> -->
</main>

<style>
  :global(body) {
    overflow-y: hidden;
  }
  @font-face {
    font-family: TradeGothicBold;
    src: url('../../assets/Trade Gothic LT Bold.ttf');
  }
  @font-face {
    font-family: RobotoRegular;
    src: url('../../assets/Roboto-Regular.ttf');
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
  top: 145px;
  left: 10px;
  width: 290px; /* Set a fixed width for the popup */
  max-height: calc(100% - 200px); /* Calculate the max height based on viewport height */
  /* overflow-y: scroll;  */
  background-color: rgb(254, 251, 249, 0.9);
  padding: 0px;
  padding-top: 15px;
  padding-left: 10px;
  padding-right: 20px;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
  overflow-x: hidden;
}

#hide {
  position: absolute;
  height: 15px;
  width: 100%;
  padding: 0px;
  margin: 0px;
  top: 0px;
  left: 0px;
  font-family: TradeGothicBold;
  font-size: 12px;
  color: #9da9bd;
  background-color: none;
  border-bottom: solid 1px rgb(227, 227, 227);
  z-index: 9999;
  text-align: center;
}
#hide:hover {
  color: #DC4633;
  cursor: pointer;
}

.legend {
    position: absolute;
    top: 10px;
    left: 10px;
    width: 300px;
    height: 105px;
    font-size: 17px;
    font-family: TradeGothicBold;
    background-color: rgb(254, 251, 249, 0.9);
    color: #1E3765;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
  }

  h1 {
    font-size: 20px;
    font-family: TradeGothicBold;
    padding: 0px;
    padding-left: 4px;
    padding-top: 2px;
    border-bottom: solid 1px #e7e7e7;
    padding-bottom: 3px;
    margin: 0px;
    margin-bottom: 7px;
    color: #00A189;
    /* background-color: #F1C500; */
    background-color: #ffffff;
    background: linear-gradient(135deg, #F1C50055 25%, transparent 25%) -4px 0/ 8px 8px, linear-gradient(225deg, #f1c50032 25%, transparent 25%) -4px 0/ 8px 8px, linear-gradient(315deg, #F1C50055 25%, transparent 25%) 0px 0/ 8px 8px, linear-gradient(45deg, #f1c50044 25%, #ffffff 25%) 0px 0/ 8px 8px;
    /* -webkit-text-stroke: 1px #6FC7EA; */
  }

  h2 {
    font-size: 26px;
    font-family: TradeGothicBold;
    padding: 2px;
    margin: 0px;
    margin-top: 8px;
    /* margin-bottom: -4px; */
    color: #00A189;
    background: linear-gradient(135deg, #F1C50055 25%, transparent 25%) -4px 0/ 8px 8px, linear-gradient(225deg, #f1c50032 25%, transparent 25%) -4px 0/ 8px 8px, linear-gradient(315deg, #F1C50055 25%, transparent 25%) 0px 0/ 8px 8px, linear-gradient(45deg, #f1c50044 25%, #ffffff 25%) 0px 0/ 8px 8px;
    /* -webkit-text-stroke: 1px #6FC7EA; */
  }

  #subtitle {
    font-family: TradeGothicBold;
    color: #1E3765;
    font-size: 16px;
  }

  p {
    font-family: RobotoRegular;
    font-size: 13px;
    opacity: 1;
    color: #1E3765;
  }

  a {
    text-decoration: underline;
    color: #1E3765;
  }
  a:hover {
    color: #DC4633;
  }

  #info {
    font-size: 11.2px;
    padding: 0px;
    margin: 0px;
    border-top: solid 1px #e7e7e7;
    margin-top: 7px;
    padding-top:7px;
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

  .legend-text {
    font-family: TradeGothicBold;
    color: #1E3765;
    font-size: 14px;
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