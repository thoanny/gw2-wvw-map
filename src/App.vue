<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";

export default {
  data() {
    return {
      map: null
    };
  },
  mounted() {
    this.map = L.map("map", {
      crs: L.CRS.Simple,
      minZoom: 2,
      maxZoom: 5,
    }).setView([-128, 128], 3);

    L.tileLayer("https://v2.lebusmagique.fr/img/map/wvw-tiles/{z}/{x}/{y}.jpg", {
      minZoom: 0,
      maxZoom: 5,
      continuousWorld: true,
      maxBoundsViscosity: 0.8,
      noWrap: true
    }).addTo(this.map);

    this.map.setMaxBounds(new L.LatLngBounds([-67.9375, 45.78125], [-188, 210.15625]));

    const keepIcon = L.icon({
      iconUrl: '/icons/keep.png',
      iconSize: [32, 32],
      iconAnchor: [16, 16],
    });

    const towerIcon = L.icon({
      iconUrl: '/icons/tower.png',
      iconSize: [32, 32],
      iconAnchor: [16, 16],
    });

    L.marker([-92.3125, 132.71875], { icon: keepIcon }).addTo(this.map);
    L.marker([-82.75, 118], { icon: towerIcon }).addTo(this.map);

    var latlngs = [
      [-84.96875, 115.15625],
      [-82.75, 115.3125],
      [-80.875, 117.125],
      [-79.09375, 118.71875],
      [-80.1875, 121.21875],
      [-81.375, 120.75],
      [-82.78125, 121.875],
      [-84.34375, 121.40625],
      [-85.1875, 120.375],
      [-86.46875, 120.625],
      [-86.40625, 115.84375]
    ];
    var polygon = L.polygon(latlngs, { color: 'red' }).addTo(this.map);

    this.map.on('click', this.onMapClick);

  },
  methods: {
    onMapClick: (e => {
      console.log(e);
      console.log("Point @ [" + e.latlng.lat + ", " + e.latlng.lng + "]");
    })
  }
};
</script>

<template>
  <div id="map"></div>
</template>

<style>
#map {
  height: 100vh;
  width: 100vw;
  cursor: crosshair;
}

.leaflet-container {
  background: #000;
  outline: 0;
}
</style>