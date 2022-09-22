<template>
    <head>
      <link
        rel="stylesheet"
        href="https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-draw/v1.3.0/mapbox-gl-draw.css"
        type="text/css"
      />
    </head>
    <div>
      <div>
        <!-- <div class="mapboxgl-ctrl-geocoder mapboxgl-ctrl">
            <input type="text" class="mapboxgl-ctrl-geocoder--input" placeholder="Search" aria-label="Search">
            <div class="suggestions-wrapper">
                <ul class="suggestions"></ul>
            </div>
            <div class="mapboxgl-ctrl-geocoder--pin-right">
                <button aria-label="Clear" class="mapboxgl-ctrl-geocoder--button"></button>
            </div>
        </div> -->
        <select id="layer-change">
          <option selected value="mapbox://styles/mapbox/streets-v11">
            Dark
          </option>
          <option value="mapbox://styles/mapbox/outdoors-v11">outdoors</option>
          <option value="mapbox://styles/mapbox/light-v10">light</option>
          <option value="mapbox://styles/mapbox/dark-v10">dark</option>
          <option value="mapbox://styles/mapbox/satellite-v9">satellite</option>
          <option value="mapbox://styles/mapbox/satellite-streets-v11">
            satellite-streets
          </option>
          <option value="mapbox://styles/mapbox/navigation-day-v1">
            navigation-day
          </option>
          <option value="mapbox://styles/mapbox/navigation-night-v1">
            navigation-night
          </option>
        </select>
      </div>
      <main class="w-screen h-screen">
        <v-map
          class="w-full h-full"
          :options="data.options"
          @loaded="onMapLoaded"
        >
          <div class="pre">
            <pre id="info"></pre>
          </div>
        </v-map>
      </main>
      <div id="map"></div>
    </div>
  </template>
  <script setup lang="ts">
  import mapboxgl from "mapbox-gl";
  import VMap from "v-mapbox";
  import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
  import MapboxDraw from "@mapbox/mapbox-gl-draw";
  import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";
  // function upload() {
  //     console.log("hii");
  // }
  const data = reactive({
    options: {
      accessToken:
        "pk.eyJ1IjoibWF5dXJ3YWtpa2FyIiwiYSI6ImNsNmdjdGxwbjBiNGMzY282bWh0dng2c2kifQ.y-m4-zQKOeOOnDG5I1u6ng",
      style: "mapbox://styles/mapbox/outdoors-v11",
      center: [73.85490417480469,18.522585482938677],
      zoom: 10,
      maxZoom: 22,
      crossSourceCollisions: false,
      failIfMajorPerformanceCaveat: false,
      attributionControl: false,
      preserveDrawingBuffer: true,
      hash: false,
      minPitch: 0,
      maxPitch: 60,
      container: "map",
    } as mapboxgl.MapboxOptions,
    map: {} as mapboxgl.Map,
  });
  let allStud = reactive({
    mapData: [],
  });
  allStud.mapData = await $fetch("http://localhost:3001/mapbox1/");
  console.log("map data", allStud.mapData);
  async function onMapLoaded(map: mapboxgl.Map) {
    //   Marker Starts
    //   data.map=map;
    allStud.mapData.map((ele) => {
      new mapboxgl.Marker({
        draggable: true,
        color: "#" + (Math.random().toString(16) + "000000").substring(2, 8),
      })
        .setLngLat([ele.lat, ele.long])
        .addTo(map);
    });
    // Marker Ends
    data.map = map;
    mapboxgl.accessToken =
      "pk.eyJ1IjoibWFkaHVyaS1waGF0YWtlIiwiYSI6ImNsNmdmNzN6ZTBkdzEzb282MmN3dmhraTUifQ.TV2rvKkRw3pkH-Iolsk5vw";
    const geocoder = new MapboxGeocoder({
      accessToken: mapboxgl.accessToken,
      mapboxgl: mapboxgl,
    });
    data.map.addControl(geocoder);
    // console.log('on map laoded: ', map);
    //add polygon using backend
    let pointsData1 = [];
    allStud.mapData.map((ele) => {
      let arrFormat = [ele.lat, ele.long];
      pointsData1.push(arrFormat);
    });
    console.log(pointsData1);
    map.addSource("pune", {
      type: "geojson",
      data: {
        type: "Feature",
        geometry: {
          type: "Polygon",
          coordinates: [
            pointsData1,
          ],
        },
      },
    });
    map.addLayer({
      id: "pune",
      type: "fill",
      source: "pune", // reference the data source
      layout: {},
      paint: {
        "fill-color": "red", // blue color fill
        "fill-opacity": 0.9,
      },
    });
    //end
    data.map.addSource("vaijapur", {
      type: "geojson",
      data: {
        type: "Feature",
        properties: {},
        geometry: {
          type: "LineString",
          coordinates: [
            // [
            [73.68942260742188, 18.530398219358684],
            [73.65509033203125, 18.340187242207897],
            [73.99154663085938, 18.359739156553683],
            [73.99429321289062, 18.641040231399984],
            [73.68942260742188, 18.530398219358684],
            // ],
          ],
        },
      },
    });
    const layer: mapboxgl.AnyLayer = {
      id: "pune",
      type: "line",
      source: "vaijapur",
      layout: {
        "line-join": "round",
        "line-cap": "round",
      },
      paint: {
        "line-color": "green", //blue color fill
        "line-width": 1,
        "line-opacity": 1,
      },
    };
    // Add a new layer to add linestring
    data.map.addLayer(layer);
    const setStyle: any = document.getElementById("layer-change");
    setStyle.addEventListener("change", (event) => {
      console.log(event);
      map.setStyle(event.target.value);
    });
    // Add a new layer to visualize the polygon.
    data.map.addLayer({
      id: "pune",
      type: "fill",
      source: "pune", // reference the data source
      layout: {},
      paint: {
        "fill-color": "black", // blue color fill
        "fill-opacity": 0.5,
      },
    });
    //Draw tool
    var Draw = new MapboxDraw();
    map.addControl(Draw, "top-right");
    //Draw tool end
    //show lat long when mouse move
    map.on("mousemove", (e) => {
      document.getElementById("info").innerHTML =
        JSON.stringify(e.point) + "<br />" + JSON.stringify(e.lngLat.wrap());
    });
  }
  </script>
  <style>
  #layer-change {
    position: fixed;
    top: 10px;
    left: 10px;
    z-index: 1;
  }
  #file {
    width: 185px;
    position: fixed;
    top: 40px;
    right: 5px;
    z-index: 1;
  }
  #search {
    position: fixed;
    top: 10px;
    right: 10px;
    z-index: 1;
  }
  html,
  body {
    margin: 0;
  }
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2C3E50;
  }
  .w-screen {
    width: 100vw;
  }
  .h-screen {
    height: 100vh;
  }
  .h-full {
    height: 100%;
  }
  .w-full {
    width: 100%;
  }
  .pre {
    z-index: 1;
    top: 0px;
    left: 600px;
    position: relative;
    width: 30%;
    height: 9%;
    background-color: white;
  }
  </style>
  
  
  
  
  
  
  
  